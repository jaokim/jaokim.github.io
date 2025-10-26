---
layout: post
title: Updating the XTools
excerpt: 'Updating the XTools for AmigaOS'
author: [JoakimNordstrom]
tags: ["Xena"]
category: amiga
draft: false
---

The XTools that are in `SYS:Utilities/Xena` were originally written by Segher Boessenkool and then adapted to AmigaOS 4 by Lyle Hazelwood. The sources are supplied in the drawer, but I've also added them to an SVN repo at sourceforge:
* [sourceforge.net/p/xena-xtools](https://sourceforge.net/p/xena-xtools).

<br>
  <p align="center">
    <img alt=""
         src="/images/amiga/xtools-repo-img/xtools.png"
         style="box-shadow: 0px 0px 20px 0px rgba(0,0,0,0.5);" />
  </p>
<br>

## Building

Building is straight-forward:

`make -f makefile-amiga`

In the sourceforge repo this also works on Amiga now -- there was a typo causing XPRegs to not be built.

### Chain too long

I've so far done a few minor fixes, for instance when there was an error it sometimes didn't close the `xena.resource`. The error usually seen is `fatal: chain too long`.

> `fatal: chain too long`

### Only builds with SDK 53.24

In normal circumstances there are no errors, so it's not major thing. However, the error always shows up if you build the XTools with a newer SDK. I've found that you need to have SDK 53.24 in order to build and also run the XTools. Because even if the sources compiles with a later SDK, when you run it, you'll get the `chain too long` error.

SDK 53.24 is still downloadable from Hyperions website.

### Doesn't run on X5000?

The same error has been spotted on the X5000, even when XTools are built with the correct SDK. I don't know the reason for this; I also don't own a X5000, so testing is difficult. I will investigate further and also file a bug report.

## Future improvements

I have som ideas for future improvements, and having the sources in a repository should hopefully help with keeping track of that. First thing I'd like to get is support for the X5000.

Suggestions and fixes are welcome! 




