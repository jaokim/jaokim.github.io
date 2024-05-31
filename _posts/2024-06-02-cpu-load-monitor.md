---
localpath: C:\Users\JSNORDST\dev\git\github.com\code.jaokim.github.io\cpu-load-monitor
layout: post
title: CPU load monitor
excerpt: 'Triggering events on high CPU load'
author: [JoakimNordstrom]
tags: ["HotSpot", "JFR", "Flight Recorder", "CPU load"]
image: /images/cpu-load-monitor-img/alex-motoc-P43VRz8fLWs-unsplash.jpg
hidden: true
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
We open a recording stream from the current JVM, enable the `jdk.CPULoad` event to be triggered every 5 seconds. For each CPU load event the "processTotal" attribute is checked, and when it exceeds 60% the default JFR recording is written to disk. 

``` java
    try (RecordingStream stream = new RecordingStream()) {
      stream.enable("jdk.CPULoad").withPeriod(Duration.ofSeconds(5));
      stream.onEvent("jdk.CPULoad", (event) -> {

        float cpuLoad = event.getFloat("processTotal");

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

When starting a recording using RecordingStream we can tell it to use one of the built-in configurations, i.e. `default.jfc`, or `profile.jfc` in `$JDK_HOME/lib/jfr/`:

```
final Configuration config = Configuration.getConfiguration("default");
RecordingStream stream = new RecordingStream(config);
```

Or we can load a custom .jfc file using the `create()` method:

```
final Configuration config = Configuration.create(Path.of("/home/app/custom.jfc"));
RecordingStream stream = new RecordingStream(config);
```

With either of these approaches we'd start a recording with the set of events configured in either configuration.

# CPU load monitoring in JDK 8 
As mentioned, the JFR recording stream came first in JDK 14. If you're still stuck on JDK 8, are you out of luck? No, for this particular use-case, where you want to monitor CPU usage, you can use the javax Management API. It's a wee bit more complex, and less intuitive -- but it works.


## Getting the CPU load
``` java
import java.lang.management.ManagementFactory;
import javax.management.MBeanServer;
import javax.management.ObjectName;

public class CPUMonitor {  
  public double getCpuLoad() throws Exception {
    final MBeanServer mrBean  = ManagementFactory.getPlatformMBeanServer();
    final ObjectName  os      = ObjectName.getInstance("java.lang:type=OperatingSystem");
    // Get wanted attribute; ProcessCpuLoad or SystemCpuLoad
    final Double cpuLevel = (double)mrBean.getAttribute(os, "ProcessCpuLoad");
    return cpuLevel;
  }
}
```

You get the Platform MBean Server, retrieve the operating system object, which has the `ProcessCpuLoad` attribute showing the CPU load.

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


# The whole packaged solution
The resulting [code is available on my github](https://github.com/jaokim/code.jaokim.github.io/tree/main/cpu-load-monitor). With JDK 8, and later JDKs having different APIs (technically later JDKs can of course use the JDK 8 API), I've packaged the entire solution in a Multi-JAR. Multi-JAR allows for different Java source files targetting different JDK releases to be included in the same JAR; if executed on JDK 8, or earlier, the standard "classes" dir in the JAR are used, but if it's running with a later JDK, classes are also loaded from that JDK-specific classes dir in the JAR, f.i. "classes-17".

