---
layout: post
redirect_url: 
title: '"Failed to write core dump"'
author: [JoakimNordstrom]
tags: HotSpot
type: text
category: java
excerpt: 'Sometimes when the JVM crashes, it tells us it failed to write core dump. This is however not the cause for the crash...'
image: /images/failed-writing-core-dump/david-libeert-aGxAsacQ1LY-unsplash-thumb.jpg
---
<style>
	table td {
		white-space: nowrap;
	}
</style>

<p align="center">
    <img alt="A photo of a collection of dumpsters" src="/images/failed-writing-core-dump/david-libeert-aGxAsacQ1LY-unsplash.jpg" style="box-shadow: 0px 0px 20px 0px rgba(0,0,0,0.5);" width="75%"/><br>
	Photo by <a href="https://unsplash.com/@deefbelgium?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">David Libeert</a>
</p>
<br>


I have examined the HotSpot error files, `hs_err`, generated when the Java Virtual Machine (JVM) crashes (see [here](/2020/12/03/crash-outside-the-jvm/)). A common “error” is the failure “to write core dump”.

<br>
<p align="center">
    <img alt="Screenshot of crash with the text 'Failed to write core dump. Minidumps are not enabled by default on client versions of Windows'." src="/images/failed-writing-core-dump/minidumps-not-enabled.png" style="box-shadow: 0px 0px 20px 0px rgba(0,0,0,0.5);" width="75%"/><br>
	"Failed to write coredump."
</p>
<br>

The failure to write a core dump is however <ins>not the error causing the JVM to crash</ins>, it is merely the JVM informing you that <ins>no core dump was written</ins>. 

<br>
**TL;DR** If you just want to get rid of the failure to write core dump, you can jump directly to the [arguments](#summary-arguments) to use.
<br><br>

**Nota bene** This will not solve the crash, you will just get a big file filling up your system, i.e. the core dump.
<br><br>

The core dump, or "Minidump" on Windows, is a snapshot of the crashed process. If you want an analogy, you can imagine the JVM being a car driving around, and suddenly crashing into whatever, like another car, a brick wall, or a badly behaving driver. In order to give the insurance company something to look at and try to find out what happened, the JVM can take a photo of the crash site.


Different operating systems have different ways of creating core dumps. Windows has the `MiniDumpWriteDump` function in the _dbghelp.dll_ DLL, it is [invoked by the JVM](https://github.com/openjdk/jdk/blob/b64a3fb946a2855c612458aca9d62ef008f8f9be/src/hotspot/os/windows/os_windows.cpp#L1190) to trigger a core dump. On the POSIX OS family (i.e. Unix, Linux & macOS), the [abort() function](https://github.com/openjdk/jdk/blob/b64a3fb946a2855c612458aca9d62ef008f8f9be/src/hotspot/os/posix/os_posix.cpp#L1906) call will make the OS generate a core dump.

The JVM argument to enable core dump writing, `-XX:+CreateCoredumpOnCrash`, is [enabled by default on JDK 9 and above](https://github.com/openjdk/jdk/blob/86bd44fe80c6222f81662b2167c402571ed68f43/src/hotspot/share/runtime/globals.hpp#L507). If you are still on JDK 8, the argument is instead named `-XX:+CreateMinidumpOnCrash`, and this only makes sense on Windows -- on other operating systems the argument is simply ignored. 
<br>

> `Failed to write core dump. Minidumps are not enabled by default on client versions of Windows`

The "client" in the "Minidumps are not enabled by default on client versions of Windows" message  refers to running on a <ins>Windows client</ins> as opposed to a <ins>Windows server</ins>; not to be confused with the old [HotSpot Client/Server VM distinction](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/index.html). On a Windows client you have to tell the JVM to write the minidump. This has nothing to do with Windows per se, so you don't need to change any settings in Windows (there are suggestions out there, that involves enabling settings for when _Windows_ crashes, but this doesn't apply here). The check to see if we're executing on a Windows client version, is explicitly [made in the JVM](https://github.com/openjdk/jdk/blob/b64a3fb946a2855c612458aca9d62ef008f8f9be/src/hotspot/os/windows/os_windows.cpp#L1132) in order to not have minidump files (which can be considerably large) fill up Windows client user's disk space. On a Windows server however the minidumps are enabled by default.

# How to enable core dumps/minidumps
## JDK 9 and above

In order to enable core dumps on JDK 9 and above for any OS, you simply add the `-XX:+CreateCoredumpOnCrash` argument to the Java launcher.

```
java -jar crasher.jar -XX:+CreateCoredumpOnCrash
```

If you instead want to _disable_ core dumps, you replace the plus sign with a minus sign: `-XX:-CreateCoredumpOnCrash`.

## JDK 8 on Windows
On a client version of Windows running JDK 8, you add the `-XX:+CreateMinidumpOnCrash` argument to create a minidump when the JVM crashes.

```
java -jar crasher.jar -XX:+CreateMinidumpOnCrash
```

If you instead want to _disable_ core dumps on a Windows _server_, you replace the plus sign with a minus: `-XX:-CreateMinidumpOnCrash`.

There is no JVM argument to disable core dumps in JDK 8 running on POSIX operating systems. Instead you must disable core dumps at the OS level.

## Unix, Linux, and macOS core dumps
On POSIX operating systems, core dumps can be disabled on the OS level. In such case, the JVM will report that "Core dumps have been disabled".

> `Failed to write core dump. Core dumps have been disabled. To enable core dumping, try "ulimit -c unlimited" before starting Java again`

To control the core dump file size, you use the `ulimit` command, and the `-c` argument. You set it to `0` to disable core dumps, and to `unlimited` to enable them. If you run `ulimit -c unlimited`, you will enable core dumps for all users and all programs. So, first enable core dumps at the OS level, then run the Java program. And remember that for JDK 9 and above, you need to add the `-XX:+CreateCoredumpOnCrash`.
```
ulimit -c unlimited
java -jar crasher.jar -XX:+CreateCoredumpOnCrash
```


## Summary of arguments {#summary-arguments}

When summarized in a table one can see that in JDK 9 and above, the handling of core dumps/minidumps is more consistent. The consistency was added to JDK 9 by [JDK-8074354, "Make CreateMinidumpOnCrash a new name and available on all platforms"](https://bugs.openjdk.java.net/browse/JDK-8074354). The two tables below summarizes the arguments and OS requirements to enable and disable core dump writing respectively. Pay attention to the plus and minus sign.

### Enabling core dumps
In this table bold entries mark the setting needed to override the default behavior (i.e. only on Windows client do you need to explicitly enable core dumps).

|----------------|-----------------------------------|----------------------------------|----------------|
| OS             | JDK 8 argument                    | JDK 9+ argumnet                  | OS control req. |
|----------------|-----------------------------------|----------------------------------|----------------|
| Windows client | **`-XX:+CreateMinidumpOnCrash`**  | **`-XX:+CreateCoredumpOnCrash`** | `N/A`      |
| Windows server | `-XX:+CreateMinidumpOnCrash`      | `-XX:+CreateCoredumpOnCrash`     | `N/A`      |
| POSIX          | `N/A` (always enabled)            | `-XX:+CreateCoredumpOnCrash`     | `ulimit -c unlimited` |
|----------------|-----------------------------------|----------------------------------|----------------| 

<br/>

### Disabling core dumps
The second table shows how to disable core dumps. Entries in bold show where the default needs to be overridden.

|----------------|-----------------------------------|----------------------------------|----------------|
| OS             | JDK 8                             | JDK 9-                           | OS control req. |
|----------------|-----------------------------------|----------------------------------|----------------|
| Windows client | `-XX:-CreateMinidumpOnCrash`      | `-XX:-CreateCoredumpOnCrash`     | `N/A` | 
| Windows server | **`-XX:-CreateMinidumpOnCrash`**  | **`-XX:-CreateCoredumpOnCrash`** | `N/A`      |
| POSIX          | `N/A` (always enabled)            | **`-XX:-CreateCoredumpOnCrash`** | `ulimit -c 0` |
|----------------|-----------------------------------|----------------------------------|----------------| 

<br/>



# What can be found in a core dump that can’t be found in the hs_err file? 

The core dump is an excellent addition to the `hs_err` text file. To examine it, you will need a native debugger.


We can already see quite a lot of things by just looking at the generated `hs_err` file.

```
# Problematic frame:
# C  [dump.dll+0x1006]
#
# Core dump will be written. Default location: D:\dumpster\hs_err_pid29804.mdmp
[...]

Native frames: (J=compiled Java code, j=interpreted, Vv=VM code, C=native code)
C  [dump.dll+0x1006]

Java frames: (J=compiled Java code, j=interpreted, Vv=VM code)
j  jaokim.dumpster.Divider.native_div_call(I)I+0
j  jaokim.dumpster.Divider.do_div(I)I+2
j  jaokim.dumpster.Dumpster.do_loops(I)V+16
j  jaokim.dumpster.Dumpster.doTestcase(Ljava/lang/Integer;)V+5
j  jaokim.dumpster.Dumpster.main([Ljava/lang/String;)V+58
v  ~StubRoutines::call_stub

siginfo: EXCEPTION_INT_DIVIDE_BY_ZERO (0xc0000094)

```

We can see that the problematic frame where the [crash happened is in native code](https://inside.java/2020/12/03/crash-outside-the-jvm/), in `dump.dll`, and that a core dump was written. The native method was called by the `jaokim.dumpster.Divider.native_div_call` Java method. We can also see what seems to be some sort of divide by zero exception (`EXCEPTION_INT_DIVIDE_BY_ZERO`). If we however want to get more details from the native code in `dump.dll`, the `hs_err` file won't help us more than this!

To get those missing details, we have to use a native debugger (ex. WinDbg on Windows) and the generated core dump. Below is a WinDbg screenshot pointing directly at the [crashing line](https://github.com/jaokim/inside-java-dumpster/blob/main/src/jaokim_dumpster_Divider.cpp#L10)!

<br>
<p align="center">
    <img alt="Screenshot of WinDbg showing the faulting source line." src="/images/failed-writing-core-dump/windbg-output.png" style="box-shadow: 0px 0px 20px 0px rgba(0,0,0,0.5);" width="75%"/><br>
</p>
<br>



To find the exact source line, the native debugger needs to have the debugging symbols. For Windows, these are in the `.pdb` file generated when the `dump.dll` is compiled. (You can put the `.pdb` file in the same directory as the coredump in order for WinDbg to find it.) On POSIX OS, the shared object (`dump.so`) needs to be compiled with debug symbols turned on.

# Summary
When the JVM crashes, it can write a core dump that contains details of the crashed process. If core dump writing is disabled, the JVM will tell you it failed to write a core dump -- this has however nothing to do with the actual reason the JVM crashed. The reason for the crash can, in most cases, be found in the `hs_err` file, and more intricate details can be found in the core dump. 

Usually, most developers don't need the core dump, but should instead focus on the `hs_err` file to first find which component caused the crash.
<br />
<br />
The example code used to crash, is [available on my GitHub](https://github.com/jaokim/code.jaokim.github.io/tree/deciphering-the-stacktrace/deciphering-the-stacktrace).