---
layout: post
title: Jfokus Programmer's Guide to JDK Flight Recorder
excerpt: 'I held a talk on JFR at the Jfokus conference in february 2023'
author: [JoakimNordstrom]
image: /images/jfokus-programmers-guide-to-jdk-flight-recorder-img/jaokim-at-jfokus-thumb.png
tags: Conference JFR
category: java
---

This is a talk about JFR that I gave at <a href="http://www.jfokus.se">Jfokus</a> in february 2023.


<iframe width="560" height="315" src="https://www.youtube.com/embed/2gTcZgiX7IE?si=9GlSO_ZaNsxSYxDA" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

## Synopsis
JDK Flight Recorder (JFR) is a low overhead profiling and troubleshooting framework built into the Java Virtual Machine. It comes with a powerful programming API that allows you to create application specific events. The API can also be used as a datasource for your own infrastructure for example when building dashboards and triggers. This talk will deep dive into the API and look at events, settings, content types, and other metadata. 

It will:

* Show how to start and stop recordings, read recording files, and consume events continuously
* Demonstrate the FlightRecorderMXBean and how it can be used to control and transfer recording data over JMX
* Provide practical tips and guidelines to help you craft your own events.


