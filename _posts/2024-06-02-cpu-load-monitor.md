---
localpath: C:\Users\JSNORDST\dev\git\github.com\code.jaokim.github.io\cpu-load-monitor
layout: post
title: CPU load monitor
excerpt: 'Triggering events on high CPU load'
author: [JoakimNordstrom]
tags: ["HotSpot", "JFR"]
image: /images/cpu-load-monitor-img/alex-motoc-P43VRz8fLWs-unsplash-thumb.png
category: java
draft: false
blog_date: 2024-06-02
---
<br>
  <p align="center">
    <img alt="" 
         src="/images/cpu-load-monitor-img/alex-motoc-P43VRz8fLWs-unsplash.jpg" 
         style="box-shadow: 0px 0px 20px 0px rgba(0,0,0,0.5);" width="75%"/>
  <br>
    <span>Photo by <a href="https://unsplash.com/@alexmotoc?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Alex Motoc</a>.</span>
  </p>
<br>
  
Quite a while ago I got a question [on ~~Twitter~~ X](https://twitter.com/csyangchsh/status/1567753686218309633?s=20&t=Or4VN7aRJI8zxJLD8NCqLA) asking whether its possible to trigger a flight recording when the CPU usage goes up. The tweeter mentioned it is possible to accomplish this using JDK Mission Control -- but that requires that you always have an instance of JDK Mission Control running with access to the JVM. Enter instead the JDK Flight Recorder's [`RecordingStream`](https://docs.oracle.com/en/java/javase/14/docs/api/jdk.jfr/jdk/jfr/consumer/RecordingStream.html), available since JDK 14. With this you can easily hook into the JFR stream of events.

# Monitoring the CPU Load
The functionality we want to implement is roughly this:

```
   while (true) {
      var cpuLoad = getCpuLoad();
      if (cpuLoad > 0.6) {
        dumpJFR();
     }
   }
```


## Getting the CPU Load
We open a recording stream from the current JVM, enable the `jdk.CPULoad` event to be triggered every 5 seconds. For each CPU load event the "systemTotal" attribute is checked, and when it exceeds 60% the default JFR recording is written to disk. 

``` java
    try (RecordingStream stream = new RecordingStream()) {
      stream.enable("jdk.CPULoad").withPeriod(Duration.ofSeconds(5));
      stream.onEvent("jdk.CPULoad", (event) -> {

        float cpuLoad = event.getFloat("systemTotal");

        if(cpuLoad > 0.6) {
          dumpJFR("recording.jfr");
        }
      });
      stream.start();
    }
```

## Writing the JFR Recording
The `RecordingStream` API has a `dump` method so we can simply dump the recording to disk.

```
        if(cpuLoad > 0.6) {
          stream.dump("recording.jfr");
        }
```

If we only ran this code as is, we would indeed get a `recording.jfr` file. However, with only the `jdk.CPULoad` event being enabled, we would only get `jdk.CPULoad` events in the recording. If the application was started with the JVM argument `-XX:StartFlightRecording=name=default`, a recording with all the default events would be dumped.

This highlights one important thing to notice when working with JFR and recordings -- there is really only one instance of the flight recorder in each JVM, despite how many recordings we might have running. This is also apparent when we issue the `dump()` method on the stream. The name is not save(), or something friendlier, because it really does dump the entirety of what the flight recorder has in its storage. This means that if we were to do a new dump() right after, it'd be virtually empty.

# CPU Load Monitoring in JDK 8 
As mentioned, the JFR recording stream was added to JDK 14. If you're still stuck on JDK 8, are you out of luck? No, for this particular use-case, where you want to monitor CPU usage, you can use the javax Management API. It's a wee bit more complex, and less intuitive -- but it works.


## Getting the CPU Load
``` java
import java.lang.management.ManagementFactory;
import javax.management.MBeanServer;
import javax.management.ObjectName;

public class CPUMonitor {  
  public double getCpuLoad() throws Exception {
    final MBeanServer mrBean  = ManagementFactory.getPlatformMBeanServer();
    final ObjectName  os      = ObjectName.getInstance("java.lang:type=OperatingSystem");
    // Get wanted attribute; SystemCpuLoad or ProcessCpuLoad
    final Double cpuLevel = (double)mrBean.getAttribute(os, "SystemCpuLoad");
    return cpuLevel;
  }
}
```

You get the Platform MBean Server, retrieve the operating system object, which has the `SystemCpuLoad` attribute showing the CPU load.

With the JFR stream API its easy to define how often the event should be created with the `enable(event).withPeriod(period)` call. In JDK 8 we can instead manually poll the CPU load using a `ScheduledExecutorService`.

``` java
    ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);
    executor.scheduleAtFixedRate(()->{
      double cpuLoad = getCpuLoad();

      if(cpuLoad > 0.6) {
        dumpJFR("recording.jfr");
      }

    }, 0, 5, TimeUnit.SECONDS);
```


## Writing the JFR recording
Using a JFR stream in later JDK versions its easy to dump a recording. In JDK 8 you can instead invoke the `jfrDump` diagnostics command. 

Although requiring quite a bit of ceremony, its not overly complex. You get the DiagnosticCommand object from the platform mbean server, and invoke the jfrDump command.

```
  private void dumpJFR(File jfrFile) throws Exception {
    final MBeanServer mrBean  = ManagementFactory.getPlatformMBeanServer();
    final String[] signature = {"[Ljava.lang.String;"};
    final ObjectName name = ObjectName.getInstance("com.sun.management:type=DiagnosticCommand");
    final Object[] params = new Object[1];
    params[0] = new String[]{"name=default", "filename="+jfrFile};
    mrBean.invoke(name, "jfrDump", params, signature);
  }
```


# Some notes regarding CPU load measuring





The [Javadoc for getSystemCpuLoad](https://docs.oracle.com/javase/8/docs/jre/api/management/extension/com/sun/management/OperatingSystemMXBean.html#getSystemCpuLoad--) states it returns an average "over the recent time period being observed". 


`double getSystemCpuLoad()`
: Returns the "recent cpu usage" for the whole system. This value is a double in the [0.0,1.0] interval. A value of 0.0 means that all CPUs were idle during the recent period of time observed, while a value of 1.0 means that all CPUs were actively running 100% of the time during the recent period being observed. All values betweens 0.0 and 1.0 are possible depending of the activities going on in the system. If the system recent cpu usage is not available, the method returns a negative value.

Returns:
: the "recent cpu usage" for the whole system; a negative value if not available.


This means that the value returned is based on _when_ you called this method _the last time_; getSystemCpuLoad is not an idempotent call. The last value is global for the entire JVM, so you can't query the CPU load from different threads or places at different times, and expect to get values that are deterministic.


|Time |   Consumer 1        |    Consumer 2        |
|     |   CPU Load interval |    CPU Load interval |
|-----|----                 |---                   |
|10:00|   5        
|10:01|   .
|10:02|   .
|10:03|   .
|10:04|   .
|10:05|   5
|10:06|                      | .   |
|10:07|                      |  2  |
|10:08|   .
|10:09|   .
|10:10|   3 

The example shows how one consumer tries to repeatedly read the CPU load during the last 5 seconds, when a second consumer comes in and reads the CPU load after 2 seconds, manifesting the observer effect, leaving only 3 seconds for consumer 1.

Worth noting, is that the JFR CPU load event has its own state. Since the JFR CPU load event is also triggered using the JFR API, it is easier to assert which resolution you get -- there are no other possibilities to query the JFR event, whereas the `getSystemCpuLoad` can in theory be called unnoticed by other code, or even from another JVM over JMX.



# The Packaged Solution
With the solution using one API for JDK 8, and another one for later JDKs, I've packaged the entire solution in a Multi-JAR. Multi-JAR allows for different Java source files targetting different JDK releases to be included in the same JAR; if executed on JDK 8, or earlier, the standard "classes" dir in the JAR are used, but if it's running with a later JDK, classes are also loaded from that JDK-specific classes dir in the JAR, f.i. "classes-17".

For more details on the entire solution, the resulting [code is available on my github](https://github.com/jaokim/code.jaokim.github.io/tree/main/cpu-load-monitor).
