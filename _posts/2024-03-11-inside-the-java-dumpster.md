---
layout: post
title: Inside the Java Dumpster
excerpt: 'A Workshop on JVM Troubleshooting'
author: [JoakimNordstrom]
tags: ["Workshop", "HotSpot", "JFR"]
category: java
image: /images/inside-the-java-dumpster-img/kevin-butz-FvFuMYVbhzE-thumb.png
blog_date: 2024-03-11
---

<br>
  <p align="center">
    <img alt="" 
         src="/images/inside-the-java-dumpster-img/kevin-butz-FvFuMYVbhzE.png" 
         style="box-shadow: 0px 0px 20px 0px rgba(0,0,0,0.5);" width="75%"/>
  <br>
    <span>Photo by <a href="https://unsplash.com/@kevin_butz?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Kevin Butz</a></span>
  </p>
<br>

Inside the Java Dumpster is the collective name for a web application written to run on the JVM. The application tries to mimic a real-world-like business application, with one obvious exception -- it is <i>deliberately</i> filled with bugs, anti-patterns, and resource hogs in order to assist in learning how to troubleshoot Java applications. Besides having issues and bugs, the application also showcases a few good things, like how to use custom JFR events, and make use of various JDK, and JVM features. Like with any dumpster, you can find both good, and bad things.

## Workshop

With the Java dumpster as a base, I've developed a workshop to explore the different monitoring and troubleshooting tools available in the JDK. 

The application is setup in a few modules, all running in a few containers. Its a mix of Java versions, ranging from 8 to the latest. The aim of the workshop is to setup the JVM so you can easily monitor it, remotely start, stop and configure recordings, analyze recordings, and find all the hidden defects. 

<br>
  <p align="center">
    <img alt="" 
         src="/images/inside-the-java-dumpster-img/memorygrowth.png" 
         style="box-shadow: 0px 0px 20px 0px rgba(0,0,0,0.5);" />
  </p>
<br>

There are examples of
* Java memory leaks
* Deadlocks
* Performance issues
* Native memory leaks


### After completing the lab you'll be able to
* Remotely monitor and control a live Java application
* Configure Java Flight Recorder for different tasks
* Diagnose applications using JDK Flight Recorder and JDK Mission Control

Tools you'll learn to use:
- jfr
- jcmd
- jconsole
- Java Mission Control

# Current sessions
A workshop session is scheduled for May 28th 2024, in Stockholm, Sweden. The event is arranged by the Jfokus team, for the Jfokus Training Camp. Registration is open at: [jfokus.se/trainingday](https://www.jfokus.se/trainingday).


