---
layout: post
title: Intelligent JVM Monitoring, Combining JDK Flight Recorder with AI
excerpt: 'How to combine JFR and JMX with AI to create an intelligent JVM monitor. A talk given at Jfokus 2026'
author: [JoakimNordstrom]
image: /images/presentation-intelligent-jvm-monitoring-img/jfokus-2026-01-thumb.png
category: java
tags: ["Conference", "JFR"]
draft: false
blog_date: 2026-02-04
---
<br>
  <p align="center">
    <img alt="" 
         src="/images/presentation-intelligent-jvm-monitoring-img/jfokus-2026.jpg" 
         style="box-shadow: 0px 0px 20px 0px rgba(0,0,0,0.5);" width="75%"/>
  <br>
    <span>Photo by <a href="https://sthlmeventfoto.pixieset.com/jfokus2026/">Jens Reiterer</a>.</span>
  </p>
<br>

This is a talk I and my collegaue Yağmur Eren gave at <a href="http://www.jfokus.se">Jfokus</a> in february 2026. The talk was well-received and actually ended up on the top 12 list of talks!

You can find the talk with slides on the <a href="https://www.jfokus.se/talks.html?showid=3044">Jfokus page</a>, and the recording is also on the <a href="https://www.youtube.com/watch?v=z8mDLU8RCUU">Java YouTube channel</a>.

## Synopsis
How do you monitor your JVM applications effectively? One powerful option is JDK Flight Recorder (JFR). JFR makes troubleshooting and profiling easier by capturing detailed records of JVM events, and its streaming API lets you access this data in real-time. But what if we could take this a step further by streaming live JFR data from your JVM application directly into an AI system to enhance monitoring and troubleshooting or even prevent potential issues before they occur?

In this session, we’ll demonstrate how to use JFR to build self-improving applications with the help of AI and the latest JDK features. Using a real-life simulated example, you’ll learn how to:

* Capture and stream JFR data in real time,
* Integrate JVM event data to an AI-based system,
* Spot performance bottlenecks and unusual behavior automatically,
* Build better tools for debugging and monitoring.

By the end of this talk, you’ll have a clear roadmap for combining JFR and AI to enhance the troubleshooting experience and observability of your JVM applications.


