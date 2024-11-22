---
layout: post
title: Øredev conference talk on JFR
excerpt: 'I presented my programmers guide to JFR at Øredev in November 2023'
author: [JoakimNordstrom]
tags: ["Conference", "JFR"]
category: java
image: /images/oredev-conference-talk-on-jfr-img/oredev-2023-11-04-thumb.png
blog_date: 2023-11-09
---

This is my talk about JFR that I gave at <a href="https://archive.oredev.org/2023/">Øredev</a> in November 2023.

<iframe width="560" height="315" src="https://www.youtube.com/embed/GQGbRWgRBkU?si=xliqAuz57TtWoycv" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

<a href="/images/oredev-conference-talk-on-jfr-img/programmers-guide-to-jfr-oredev.pdf">Slides</a>

## Synopsis

#### Key takeaways

- You will learn how to get flight recordings from your JVM, both for always-on monitoring, and more detailed profiling.
- You'll get know how to create your own JFR events, and learn how to analyze them using Java Mission Control
- Even if you're on the latest JDK, or on earlier you'll get tips on how to use JFR in unit-tests
- Using practical examples you'll see how JFR can be used to inspect the entire software stack, and correlate events from your application, through the JDK, all the way down to the JVM

JDK Flight Recorder (JFR) is a low overhead profiling and troubleshooting framework built into the Java Virtual Machine. It comes with a powerful programming API that allows you to create application specific events. The API can also be used as a datasource for your own infrastructure for example when building dashboards and triggers. This talk will deep dive into the API and look at events, settings, content types, and other metadata. 

It will:

* Show how to start and stop recordings, read recording files, and consume events continuously
* Demonstrate the FlightRecorderMXBean and how it can be used to control and transfer recording data over JMX
* Provide practical tips and guidelines to help you craft your own events.
