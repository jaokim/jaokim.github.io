---
layout: post
title: Deciphering the stack trace
excerpt: 'When investigating the causes for a JVM crash, it is important to understand the crashlog, and the stack trace. Correctly interpreting the stack frames can give us valuable information on what has happened…'
author: [JoakimNordstrom]
tags: ["HotSpot", "Serviceability"]
image: /images/deciphering-the-stacktrace/pine-watt-3_Xwxya43hE-unsplash.jpg
---


In this post we will be looking at crash logs, the `hs_err` file, that is generated when the Java Virtual Machine crashes. Trying to find what is going wrong, and which component to blame, is important to understand how to interpret the crash log file. The focus will be on understanding the frames that make up the stack trace. 

<br>
<p align="center">
    <img alt="A hand holding an empty frame in front of a cliff by the sea" src="/images/deciphering-the-stacktrace/pine-watt-3_Xwxya43hE-unsplash.jpg" style="box-shadow: 0px 0px 20px 0px rgba(0,0,0,0.5);" width="75%"/><br>
    <span>Photo by <a href="https://unsplash.com/@pinewatt">Pine Watt</a></span>
</p>
<br>

## Divide and crash
For this post I've created an example consisting of a Java loop, dividing a constant by a decreasing denominator, eventually resulting in a division by zero. The code takes a few twists and turns to get a more interesting stack trace. It consists of two Java classes: `Dumpster` and `Divider`; `Dumpster` loops, and calls `Divider.do_div()` for each iteration. `Divider` makes the call to the native JNI library `libdump.so` that performs the division. 

An abbreviated version can be seen below. Full code with build and run instructions are available on my [GitHub](https://github.com/jaokim/code.jaokim.github.io/tree/deciphering-the-stacktrace/deciphering-the-stacktrace).

Java classes:
```java
class Dumpster {
  void main(...) {
    do_loops(no_of_loops);
  }
  
  void do_loops(int no_of_loops) {
    for(int i = no_of_loops; i >= 0; i--) {
        new Divider().do_div(i);
    }
  }
}
class Divider {
  void do_div(int denominator) {
    native_div_call(denominator);
  }
  
  native void native_div_call(int denominator);
}
```

Native libdump library:
```c
// This is the native method native_div_call
JNIEXPORT void JNICALL Java_jaokim_dumpster_Dumpster_native_1div_1call 
    (jint denominator) {
  int numerator = 9;
  // call C standard library div
  div(numerator, denominator);
}
```

Running this program for a few loops will trigger the crash.

```
duke@cafedead:/work# java Dumpster 10
    #
    # A fatal error has been detected by the Java Runtime Environment:
    #
    #  SIGFPE (0x8) at pc=0x00007fc0b124a5a3, pid=6208, tid=6209
    #
    # JRE version: OpenJDK Runtime Environment (15.0.1+9)
    # Java VM: OpenJDK 64-Bit Server VM (15.0.1+9-18, mixed mode, aot, sharing, tiered, …
->  # Problematic frame:
->  # C  [libc.so.6+0x3a5a3]  div+0x3
    #
    # If you would like to submit a bug report, please visit:
    #   https://bugreport.java.com/bugreport/crash.jsp
->  # The crash happened outside the Java Virtual Machine in native code.
->  # See problematic frame for where to report the bug.
```
Besides the problematic frame, there are some clear text instructions on what has happened: "The crash happened outside the Java Virtual Machine in native code" and "See problematic frame for where to report the bug".

> ```The crash happened outside the Java Virtual Machine in native code.```

More closely investigating the generated `hs_err` file, and specifically the stack trace, the first character for each frame tells us what kind of code we are at. There is a hint to their meaning directly in the `hs_err` file:

```
   J=compiled Java code, A=aot compiled Java code, j=interpreted, Vv=VM code, C=native code
```

We will go through the stack trace starting at the top, with the problematic frame, trying to make some more sense of the helpful hint.

```
    Stack: [0x00007f2b92572000,0x00007f2b92673000],  sp=0x00007f2b92671788,  free space=1021k
    Native frames: (J=compiled Java code, A=aot compiled Java code, j=interpreted, Vv=VM code, C=native code)
->  C  [libc.so.6+0x3a5a3]  div+0x3
    C  [libdump.so+0x1162]  Java_jaokim_dumpster_Divider_native_1div_1call+0x29
    j  jaokim.dumpster.Divider.native_div_call(I)V+0
    j  jaokim.dumpster.Divider.do_div(I)V+2
    j  jaokim.dumpster.Dumpster.do_loops(I)V+14
    j  jaokim.dumpster.Dumpster.doTestcase(Ljava/lang/Integer;)V+5
    j  jaokim.dumpster.Dumpster.main([Ljava/lang/String;)V+58
    v  ~StubRoutines::call_stub
    V  [libjvm.so+0x773a29]  JavaCalls::call_helper(...)+0x299
    V  [libjvm.so+0x804747]  jni_invoke_static(...) [clone .isra.0] [clone .constprop.1]+0x357
    V  [libjvm.so+0x80733f]  jni_CallStaticVoidMethod+0x12f
    C  [libjli.so+0x4627]  JavaMain+0xd17
    C  [libjli.so+0x7fc9]  ThreadJavaMain+0x9
```

#### C=native code
The top frame, the problematic one, is prefixed with a `C`. The `C` tells us that this is native code. We see it crashes in the C standard library `libc.so`, while executing the function `div`. The next frame is also native code, executing the awkwardly named `Java_jaokim_dumpster_Dumpster_native_1div_1call`-function in our own library `libdump.so`. 

#### j=interpreted
Before that, our Java classes `Divider` and `Dumpster` have been executing. They've been executing in interpreted mode, which the lowercase prefix `j` tells us. 

#### Vv=VM code
Our Java classes and methods are called after a few frames of internal native code; the `~StubRoutines::call_stub` prefixed with lowercase `v`, and the `libjvm` frames prefixed with uppercase `V`. The lowercase indicates VM generated code, and the uppercase is code from the internal `libjvm` library (this distinction is worthy a text of its own; it's sufficient to understand that `v` and `V` is internal VM native code). 

Execution of our program starts with the two bottom frames. As seen by the `C`, this is native code from the `libjli` Java launcher library. As the name suggests, this library launches the JVM, and is what the `java` command uses to start our Java program. Although being Java specific, it doesn't count as VM internal code, since it doesn't execute in the JVM/HotSpot memory space.

#### J=compiled Java code

Let's take a look at the uppercase `J` frame type, "compiled Java code". Using the same example, only increasing the number of loops to say a thousand iterations, we get a slightly different stack trace.

```
duke@cafedead:/work# java Dumpster 1000
#
# A fatal error has been detected by the Java Runtime Environment:
#
```

The crash log looks pretty much the same, apart from the Java frames for `native_div_call`, and `do_div_call`, now has an uppercase `J`. 

```
    C  [libc.so.6+0x3a5a3]  div+0x3
    C  [libdump.so+0x1162]  Java_jaokim_dumpster_Divider_native_1div_1call+0x29
->  J 56  jaokim.dumpster.Divider.native_div_call(I)V (0 bytes) @ 0x00007f67a87e46db [0x00007f67a87e4620+0x00bb]
->  J 55 c1 jaokim.dumpster.Divider.do_div(I)V (6 bytes) @ 0x00007f67a12b33a4 [0x00007f67a12b3320+0x0084]
    j  jaokim.dumpster.Dumpster.do_loops(I)V+14
    j  jaokim.dumpster.Dumpster.doTestcase(Ljava/lang/Integer;)V+5
    j  jaokim.dumpster.Dumpster.main([Ljava/lang/String;)V+58
    v  ~StubRoutines::call_stub
    V  [libjvm.so+0x773a29]  JavaCalls::call_helper(...)+0x299
    V  [libjvm.so+0x804747]  jni_invoke_static(...) [clone .isra.0] [clone .constprop.1]+0x357
    V  [libjvm.so+0x80733f]  jni_CallStaticVoidMethod+0x12f
    C  [libjli.so+0x4627]  JavaMain+0xd17
    C  [libjli.so+0x7fc9]  ThreadJavaMain+0x9
```

When we run our loop this many times, the JVM sees the method `do_div` being a hotspot, and instead of interpreting it each time, it compiles it just-in-time for execution in order to save valuable clock cycles. The numbers (56 and 55) are the ID of the task that compiled the method, the number of bytes is the size of the generated code, and the addresses point to the compiled code in memory.

#### A=aot compiled Java code

In order to get full coverage of the different frame types, we have to add an ahead-of-time compiled step, using the AOT feature of [JEP 295](https://openjdk.java.net/jeps/295). This example shows how we are using a pre-compiled (i.e. ahead-of-time) version of the `Divider` class; using the `jaotc` command the class is compiled into the AOT-library `divider.so`. We unlock experimental mode for Java and add the `AOTLibrary` argument. 

```
duke@cafedead:/work# java -XX:+UnlockExperimentalVMOptions -XX:AOTLibrary="divider.so" Dumpster 1000
```

Looking at that crash, the `Divider.do_div()`-frame is now prefixed with an uppercase `A` showing we're using the ahead-of-time compiled method.

```
    C  [libc.so.6+0x3a5a3]  div+0x3
    C  [libdump.so+0x1162]  Java_jaokim_dumpster_Divider_native_1div_1call+0x29
    J 55  jaokim.dumpster.Divider.native_div_call(I)V (0 bytes) @ 0x00007fc0987e465b [0x00007fc0987e45a0+0x00bb]
->  A 2  jaokim.dumpster.Divider.do_div(I)V (6 bytes) @ 0x00007fc0afb7d194 [0x00007fc0afb7d120+0x0074]
    j  jaokim.dumpster.Dumpster.do_loops(I)V+14
    j  jaokim.dumpster.Dumpster.doTestcase(Ljava/lang/Integer;)V+5
    j  jaokim.dumpster.Dumpster.main([Ljava/lang/String;)V+58
    v  ~StubRoutines::call_stub
    V  [libjvm.so+0x773a29]  JavaCalls::call_helper(...)+0x299
    V  [libjvm.so+0x804747]  jni_invoke_static(...) [clone .isra.0] [clone .constprop.1]+0x357
    V  [libjvm.so+0x80733f]  jni_CallStaticVoidMethod+0x12f
    C  [libjli.so+0x4627]  JavaMain+0xd17
    C  [libjli.so+0x7fc9]  ThreadJavaMain+0x9
```

## Who caused the crash?
In this particular case, we know what happened; we made a division by zero. Even though the problematic frame was in the C standard library, and `hs_err` file explicitly tells us to "See problematic frame for where to report the bug", it's rather unlikely that the well-used C standard library should crash this easily. So, instead of just blaming `libc.so`, one should look at the other frames, and see what is going on. The next suspect on our list is our own native library. And, yes, it could be fixed here, by checking if the `denominator` is zero before calling div.

```c
JNIEXPORT void JNICALL Java_jaokim_dumpster_Dumpster_native_1div_1call 
    (jint denominator) {
  int numerator = 9;
  // call C standard library div
  if(denominator == 0) {
    return;
  }
  div(numerator, denominator);
}
```

However, instead of actually calling the native code at all, it would be better to just fix this in the Java loop, making sure `i` never hits zero, by changing the "`i >= 0`" to just "`i > 0`". 

```java
class Dumpster {
  void do_loops(int no_of_loops) {
    for(int i = no_of_loops; i > 0; i--) {
        new Divider().do_div(i);
    }
  }
}
```


## Summary
The `hs_err` file has its snappy summary of the different stackframe types, and also gives helpful hints on where to report bugs and crashes.

```
J=compiled Java code, A=aot compiled Java code, j=interpreted, Vv=VM code, C=native code
```

However, we can enhance it a little into the table below, and hopefully, this post has given further background to its meaning.

| Char | Description                                   |
|------|-----------------------------------------------|
| `j`  | Java code interpreted by the JVM at runtime   |
| `J`  | Java code compiled just in time by the JVM    |
| `A`  | Java code compiled ahead-of-time, by `jaotc`  |
| `C`  | Native code from external library, not residing in the JVM |
| `V`  | Native code from library part of the JVM      |
| `v`  | Native code genereted by the JVM             |

## Further reading
[The Java troubleshooting guide](https://docs.oracle.com/en/java/javase/15/troubleshoot/troubleshoot-system-crashes.html) is great if you need further assistance in troubleshooting a crashing Java application. This guide gives advices on how to determine what crashed, but also tips and possible workarounds.

Full code with build and run instructions for the code is available on my [GitHub](https://github.com/jaokim/code.jaokim.github.io/tree/deciphering-the-stacktrace/deciphering-the-stacktrace)