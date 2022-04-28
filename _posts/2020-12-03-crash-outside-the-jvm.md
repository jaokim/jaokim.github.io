---
layout: post
title: A crash happened outside the Java Virtual Machine
author: [JoakimNordstrom]
tags: ["HotSpot", "Serviceability"]
excerpt: 'There you are, relaxing and enjoying a game of Minecraft, and suddenly "A fatal error has been detected by the Java Runtime Environment"! To find cues, you want to look in the HotSpot error log.'

image: /images/crash-outside-the-jvm-img/cindy-tang-ob-hsLNxYPc-unsplash.jpg
---

There you are, relaxing and enjoying a game of Minecraft, and suddenly "`A fatal error has been detected by the Java Runtime Environment`"!

<br>
<p align="center">
	<img alt="book cover" src="/images/crash-outside-the-jvm-img/cindy-tang-ob-hsLNxYPc-unsplash.jpg" style="box-shadow: 0px 0px 20px 0px rgba(0,0,0,0.5);" width="75%"/><br>
	<span>Photo by <a href="https://unsplash.com/@tangcindy">Cindy Tang</a></span>
</p>
<br>

## Fatal errors in Java applications
A fatal error can be caused by any Java application. To find cues, you want to look in the generated `hs_err file`. Below is an example of a [crash that occured when running Minecraft][1].


<pre><code>#
# A fatal error has been detected by the Java Runtime Environment:
#
# EXCEPTION_ACCESS_VIOLATION (0xc0000005) at pc=0x0000000022051066, pid=2372, tid=0x00000000000017a0
#
# JRE version: Java(TM) SE Runtime Environment (8.0_172-b11) (build 1.8.0_172-b11)
# Java VM: Java HotSpot(TM) 64-Bit Server VM (25.172-b11 mixed mode windows-amd64 compressed oops)
</code><b style="color: red;"># Problematic frame:</b>
<b style="color: red;"># C [OpenAL64.dll+0x11066]</b><code>
#
# Failed to write core dump. Minidumps are not enabled by default on client versions of Windows
#
# If you would like to submit a bug report, please visit:
# http://bugreport.java.com/bugreport/crash.jsp
</code><b style="color: red;"># The crash happened outside the Java Virtual Machine in native code.</b>
<b style="color: red;"># See problematic frame for where to report the bug.</b>
<code>#</code>

</pre>

## Is it really the JVM that is crashing?
Before blaming the JVM, and filing a bug report on java.com, you want to make sure it's really the Java Runtime causing the crash. Actually, just below the bug report link, if you see the message: "`The crash happened outside the Java Virtual Machine in native code`", it's a clear sign that the crash was caused, not by the Java Virtual Machine, but rather some other piece of code, most likely a library of some kind. 

## The crash happened outside the Java Virtual Machine
Searching further in the `hs_err` file, you should see the string "`See problematic frame for where to report the bug`",  which suggests you should look further up in the file to pinpoint the problem. 

<pre><code>…
# Problematic frame:
# C [OpenAL64.dll+0x11066]
…</code></pre>

Looking at the problematic frame, we find that the crash happened in the `OpenAL64.dll`, a library that Minecraft uses to handle sounds.

## Stacktrace
We can further analyze the cause of the crash by looking at the stack trace.

<pre><code>…
Stack: [0x000000002bf80000,0x000000002c080000], sp=0x000000002c07e980, free space=1018k
Native frames: (J=compiled Java code, j=interpreted, Vv=VM code, C=native code)
C [OpenAL64.dll+0x11066]
C [OpenAL64.dll+0x1248f]
C 0x0000000005788c67

Java frames: (J=compiled Java code, j=interpreted, Vv=VM code)
j org.lwjgl.openal.ALC10.nalcCreateContext(JJ)J+0
j org.lwjgl.openal.ALC10.alcCreateContext(Lorg/lwjgl/openal/ALCdevice;Ljava/nio/IntBuffer;)Lorg/lwjgl/openal/ALCcontext;+8
j org.lwjgl.openal.AL.init(Ljava/lang/String;IIZZ)V+69
j org.lwjgl.openal.AL.create(Ljava/lang/String;IIZZ)V+246
j org.lwjgl.openal.AL.create(Ljava/lang/String;IIZ)V+5
j org.lwjgl.openal.AL.create()V+6
j paulscode.sound.libraries.LibraryLWJGLOpenAL.init()V+2
j paulscode.sound.SoundSystem.CommandNewLibrary(Ljava/lang/Class;)V+273
j paulscode.sound.SoundSystem.CommandQueue(Lpaulscode/sound/CommandObject;)Z+1206
j paulscode.sound.CommandThread.run()V+51
v ~StubRoutines::call_stub
…</code></pre>

We see there are a few native frames (prefixed with `C`) executing in OpenAL64 DLL. The call to the DLL is made from Java code (prefixed `j`) by LWJGL (Lightweight Java Game Library), which in turn is called from paulscode.sound (Paulscode's 3D Sound System). The `~StubRoutines::call_stub` is a call from the JVM that is used to call Java methods from C code. This stack trace also serves as compelling evidence that the crash is not due to problems in the JVM itself, given that the only JVM related stack trace entry is the StubRoutines call. Had we access to the `OpenAL64.dll`, or knew how it's supposed to be called, we could make our way through the stack, and the code involved, to possibly find if the error is in the DLL itself, the LWJGL code, or the sound system library.

# Summary
JVM crashes can be caused by external components, either something directly used by the running software or indirectly related, for instance, virus protection or other system software. In these cases, it is unlikely that a bug report on java.com will help. The first step is to find the real culprit by looking in the problematic frame in the `hs_err-file`, and then act accordingly, ex. report the issue to the appropriate owner.


[1]: https://bugs.openjdk.java.net/browse/JDK-8202755
