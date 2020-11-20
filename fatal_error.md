There you are, relaxing and enjoying a game of Minecraft, and suddenly, "A fatal error has been detected by the Java Runtime Environment"!

A fatal error can be caused by any Java application, it's not specifically a Minecraft error. Trying to find a solution you want to look in the generated hs_err file.


#
# A fatal error has been detected by the Java Runtime Environment:
#
# EXCEPTION_ACCESS_VIOLATION (0xc0000005) at pc=0x0000000022051066, pid=2372, tid=0x00000000000017a0
#
# JRE version: Java(TM) SE Runtime Environment (8.0_172-b11) (build 1.8.0_172-b11)
# Java VM: Java HotSpot(TM) 64-Bit Server VM (25.172-b11 mixed mode windows-amd64 compressed oops)
# Problematic frame:
# C [OpenAL64.dll+0x11066]
#
# Failed to write core dump. Minidumps are not enabled by default on client versions of Windows
#
# An error report file with more information is saved as:
# C:\java\dir\mod\run\hs_err_pid2372.log
#
# If you would like to submit a bug report, please visit:
# http://bugreport.java.com/bugreport/crash.jsp
# The crash happened outside the Java Virtual Machine in native code.
# See problematic frame for where to report the bug.
#
AL lib: (EE) alc_cleanup: 1 device not closed
Java HotSpot(TM) 64-Bit Server VM warning: Using incremental CMS is deprecated and will likely be removed in a future release
Java Crash log:
#
# A fatal error has been detected by the Java Runtime Environment:
#
# EXCEPTION_ACCESS_VIOLATION (0xc0000005) at pc=0x0000000022051066, pid=2372, tid=0x00000000000017a0
#
# JRE version: Java(TM) SE Runtime Environment (8.0_172-b11) (build 1.8.0_172-b11)
# Java VM: Java HotSpot(TM) 64-Bit Server VM (25.172-b11 mixed mode windows-amd64 compressed oops)
# Problematic frame:
# C [OpenAL64.dll+0x11066]
#
# Failed to write core dump. Minidumps are not enabled by default on client versions of Windows
#
# If you would like to submit a bug report, please visit:
# http://bugreport.java.com/bugreport/crash.jsp
# The crash happened outside the Java Virtual Machine in native code.
# See problematic frame for where to report the bug.
#


You might find it's something like a SIGSEGV, SIGBUS, or EXCEPTION_ACCESS_VIOLATION. Before blaming the Java Runtime Environment, and filing a bugreport on java.com, you might want to make sure it's really the Java Runtime causing the crash. Actually, just below the bugreport link, if you see the text: "The crash happened outside the Java Virtual Machine in native code", it's a clear sign that the crash was caused, not by the Java Virtual Machine, but rather some other piece of code, most likely a library of some kind. The other string "See problematic frame for where to report the bug", suggests you should look further up in the hs_err file to pinpoint the problem. 
# Problematic frame:
# C [OpenAL64.dll+0x11066]

Searching for the problematic frame, we find that the crash happened in the OpenAL64.dll, a library that Minecraft uses to handle the sound in the game. Googling this, points us into the Minecraft forums, and possibly a solution [2], which in this case consisted of an updated OpenAL64.dll. 

Looking at another bug report [3] with a fatal error while running Minecraft, we find the error being in FamHook.dll. 
# Problematic frame:
# C [FamHook.dll+0x1704]

Further investigation on this matter, reveals it isn't directly tied to a Minecraft component, but rather a parental control software [4] presumably running in the background hooking into the network stack, disturbing Minecraft. I haven't investigated further, and the error could be either the parental control software doing something wrong, or Minecraft not handling something correctly. In my next blog post, I'll however take a look at a fatal crash, not caused by the Java Virtual Machine, but in the Java runtime.

Summary
When the Java Runtime Environment crashes, it can be due to external components, either something directly used by the running software, or indirectly related, for instance virus protection or other system software. In these cases it is unlikely that a bugreport to java.com will help. The first step to finding the real culprit, is to look in the problematic frame in the hs_err-file.

[2] https://www.minecraftforum.net/forums/archive/legacy-support/1737178-openal-access-violation
[3] https://bugs.openjdk.java.net/browse/JDK-8223809
[4] https://bugs.mojang.com/browse/MC-26702
