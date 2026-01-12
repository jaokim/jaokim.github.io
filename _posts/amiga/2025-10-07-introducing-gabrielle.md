---
layout: post
title: Introducing Gabrielle
excerpt: 'Gabrielle the ever present side-kick to Xena'
author: [JoakimNordstrom]
tags: ["Xena"]
category: amiga
draft: false
blog_date: 2025-10-07
---

So I’ve been poking at Xena. I’ve had this idea for the longest to do something with the chip. The biggest obstacle has however been actually compiling anything that runs on it. On various occasions I’ve set up the Xmos developer environment, but it’s always been a cumbersome moving of files and it has just taken the energy away from actually trying out some stuff.

Now, however, finally I’ve managed to get something working. Introducing Gabrielle! It’s Xena's companion, always there to help! It’s basically a script that sends off the files to compile to a compilation server. The compilation server in turn — aptly named Sai — collects the source files and compiles them into a xe binary, ready to be run with the XRunXE command.

So in order to compile and run the wiggle demo in the Xena demo drawer (usually SYS:Utilities/Xena), you just have to do this:

```
1.> gabrielle TARGET=wiggle.xe wiggle.xc
xcc wiggle.xc
Saved as wiggle.xe
1.> XRunXE wiggle.xe
```

I've assembled the script along with a console that allows you to do some experimenting. Basically just download the archive, unpack and test the GabriShell console.

<br>
  <p align="center">
    <img alt="" 
         src="/images/amiga/xena-img/gabrishell.png" 
         style="box-shadow: 0px 0px 20px 0px rgba(0,0,0,0.5);" />
  </p>
<br>


If you want to develop in your environment, Gabrielle sits happy in your path and can be invoked from a Makefile.

You can download the archive <a href="https://os4depot.net/?function=showfile&file=development/misc/gabrielle.lha">from os4depot</a>.






