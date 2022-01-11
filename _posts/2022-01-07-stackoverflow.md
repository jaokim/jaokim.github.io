---
layout: post
title: You don't gotta catch 'em all
excerpt: "Given a StackOverflowError, there's likely not enough stack to do anything about it."
author: [Joakim]
date:   2022-01-07 11:52:25 +0100
tags: hotspot stackoverflow exceptions
splash-url: /images/stackoverflow-img/josh-calabrese-fXigBxcZXWc-unsplash.jpg
---

There are a few throwables the JVM might throw at you, that you shouldn't try to catch. Basically, "a reasonable application" ([as the docs say](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Error.html)) shouldn't normally try to catch any throwable that is a java.lang.Error, because it indicates a serious problem with the JVM. In this post I'll take a closer look at the java.lang.StackOverflowError, and try to motivate why it's a bad idea to try to catch these.

<br>
  <p align="center">
    <img alt="A dog catching a ball" 
         src="/images/stackoverflow-img/josh-calabrese-fXigBxcZXWc-unsplash.jpg" 
         style="box-shadow: 0px 0px 20px 0px rgba(0,0,0,0.5);" width="90%"/><br/>
         <span>Photo by <a href="https://unsplash.com/@joshcala?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Josh Calabrese</a></span>
  <br>
  </p>
<br>

A report that came to the JDK support team consisted of a reproducible case, where, allegedly, HotSpot failed to handle exceptions after a StackOverflowError [(JDK-8266955)](https://bugs.openjdk.java.net/browse/JDK-8266955).

The reproducer, supplied by reporter Yingquan Zhao, is short enough (slightly modified to add a System.in.read() in main):

```java
public class Bug {

    static Object m1(boolean var0) {
        throw new NullPointerException();
    }

    static boolean m2(boolean var0) {
        boolean var1 = false;

        try {
            var1 = m2(var0);
        } catch (StackOverflowError e) {
            return true;
        }
// System.out.println("This is the only diff");
        if (var1) {
            try {
                m1(var0);
            } catch (StackOverflowError e) { }
        }
        return false;
    }

    public static void main(String[] var0) throws Exception {
        try {
            m2(true);
        } catch(Throwable e) {
            e.printStackTrace();
        }
        System.in.read();
    }
}
```

Notice the commented `System.out.println("This is the only diff")`.

## Hypothetical program execution
Deciphering the source, `main` calls `m2`, which calls itself until the stack overflows. When the stack overflows, `m2` returns true, which makes the calling `m2` set `var1` to `true` which should call `m1` which will throw a NullPointerException. This exception will trickle up and be printed by the `main` try-catch. After this the program waits for input before it exits. 

<br>
  <p align="center">
    <img alt="Diagram showing the programs call flow." 
         src="/images/stackoverflow-img/source-path.png" 
         style="box-shadow: 0px 0px 0px 0px rgba(0,0,0,0.5);" width="100%"/><br/>
         <span>Figure 1. Visualization of the <i>hypothetical</i> program execution, without the <code>println</code>. One StackOverflowError (SOFE) is thrown. The observant reader might notice how fuzzy I made the stack limit here... (Hint: this picture is not the truth.)</span>
  <br>
  </p>
<br>

Thus, the expected output is to get a NullPointerException. Running it produces this output instead:

```
$ java Bug.java

```

I.e. nothing. The NullPointerException isn't thrown, and there is no indication of any error on the console. 

However, un-commenting the `System.out.println("This is the only diff")` will produce the following output:

```
$ java Bug.java
This is the only diff
Exception in thread "main" java.lang.NullPointerException
        at Bug.m1(Bug.java:4)
        at Bug.m2(Bug.java:18)
        at Bug.m2(Bug.java:11)
        at Bug.m2(Bug.java:11)
        at Bug.m2(Bug.java:11)
        ...
```

We first see "This is the only diff" from the `println`, followed by the expected NullPointerException, and an awfully long stacktrace.

So how come our NullPointerException wasn't shown _without_ the `println`? How does a `println` added before a method make it do what it should (throw an exception)? Why does throwing a NullPointerException require a `println`? 

## Generate a VM report
To get insight into a running VM, we can use the [`jcmd`](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jcmd.html) tool. With our program running we can execute `jcmd <pid> VM.info`. (In order to get the pid of the running VM, you can just run `jcmd` without any arguments.)

The output of `VM.info` is pretty much the same we get from the HotSpot error reports when the JVM crashes. There's quite a lot of information, we're going to focus on the reported _exception counts_ that comes in the beginning in the process section.

```
---------------  P R O C E S S  ---------------

OutOfMemory and StackOverflow Exception counts:
StackOverflowErrors=2

```

We see there are actually two StackOverflowErrors <superscript>(you might also see a few LinkageErrors in this section, but these are unrelated)</superscript>. 

## Actual program execution
The first StackOverflowError is thrown when the call to `m2` overflows the stack. This is caught, and `true` is returned, which sets `var1` in the calling `m2`. With `var1` true, a call to `m1` is done. The only thing `m1` tries to do is to construct and throw an exception. However, constructing the exception is essentially a method call, which will occur on the same stack level as the first failing `m2` call. Therefore a new StackOverflowError will be thrown. This is caught, silently ignored by the `m1`'s surrounding try-catch, and `false` is returned from `m2`. With `m2` returning `false`, `m1` is never called. This then unwinds back through the stack, returning `false` all the way, eventually exiting `m2`, and the `main` method (see figure 2).

<p align="center">
<img alt="Diagram showing the program's call flow." 
     src="/images/stackoverflow-img/actual-source-path.png" 
     style="box-shadow: 0px 0px 0px 0px rgba(0,0,0,0.5);" width="100%"/><br/>
     <span>Figure 2. A more accurate visualization of the program execution. Notice the stack limit is more accurately limited, clearly illustrating the behaviour; neither <code>m2</code>'s call to itself nor the <code>m1</code> call fits the stack, resulting in a StackOverflowError.</span>
</p>



## Adding some printlines
Now, lets take a closer look at what happens when we run _with_ the `println` in place. 

```
$ java Bug.java
This is the only diff
java.lang.NullPointerException
        at Bug.m1(Bug.java:3)
        at Bug.m2(Bug.java:16)
        at Bug.m2(Bug.java:9)
        at Bug.m2(Bug.java:9)
        at Bug.m2(Bug.java:9)
        at Bug.m2(Bug.java:9)
        ...
```

We see that `println` managed to output `"This is the only diff"` and the NullPointerException stacktrace. 

Running `jcmd <pid> VM.info` for this process reveals a whopping 114 StackOverflowErrors! 

```
---------------  P R O C E S S  ---------------

OutOfMemory and StackOverflow Exception counts:
StackOverflowErrors=114
```

What the... stack?


## Program execution with the println
The first StackOverflowError is thrown from `m2` when the stack is full. On returning `true` to the calling `m2`, there's a `println`. This `println` will naturally also require some stack to be called... that's, however, stack space we don't have. So a new error is thrown. Since the `println` isn't inside a catch clause, the exception is delegated to the calling `m2`, where it's caught, and `true` is returned. With that `true`, the previous caller tries to call its `println`. We get a few calls longer, but there's still not enough stack space, resulting in yet another stack overflow error. And like this, it continues down the call stack, when eventually, there's enough space to execute our `println`. When the `println` finally executes, the program can continue to look at `var1` being `true`, and then executing `m1` will throw the NullPointerException. And then we're done. 

<p align="center">
<img alt="Diagram showing the program's call flow." 
     src="/images/stackoverflow-img/actual-source-println-path.png" 
     style="box-shadow: 0px 0px 0px 0px rgba(0,0,0,0.5);" width="100%"/><br/>
     <span>Figure 3. Visualization of the program execution. As can be seen, the println requires a few stack frames, thus, generating quite a few StackOverflowErrors before enough stack is freed and the message is successfully printed.</span>
</p>

Whew, quite the trip!

It seems that when the `println` produced its stack overflows, there was eventually enough room for `m1` to complete. In other words, when the entirety of the `println` fit the stack and was done, that same stack amount could be used to both throw, and create the exception. What to note, though, is that `m1`'s NullPointerException wasn't thrown at the same stack level as the first StackOverflowError; it took 114 StackOverflowErrors before we got to executing `m1`.


## Given a StackOverflowError, there's likely not enough stack to do anything about it
This short code example is a perfect demo of why you shouldn't try to catch a StackOverflowError. It simply cannot be guaranteed that there is enough stack available for application code to do anything reasonable - not even logging it. So, if you ever find yourself catching a StackOverflowError, simply do a mic drop, and exit as fast as possible.

In a few coming posts, I'll further expand on how the stack is managed in HotSpot. Stay tuned.
