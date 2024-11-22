---
layout: post
title: Using the JDK Flight Recorder
excerpt: 'A quick guide on the JDK Flight Recorder options.'
author: [JoakimNordstrom]
toc: true
tags: HotSpot JFR
image: /images/using-the-jdk-flight-recorder-img/markus-spiske-gnhxvdGmGG8-unsplash.jpg
---
<style>
div.jdkspecific {
    border-right-style: solid;
    border-bottom-style: none;
    border-top-style: none;
    border-width: 1em;
    margin-bottom: 1em;
    margin-top: 1em;
    padding-right: 1em;
}
div.jdkspecific.jdk8 {
    border-color: #9d68683d;
}
div.jdkspecific.jdk11 {
    border-color: #418a416b;
}
div.jdkspecific.jdk17 {
    border-color: #788a416b;
}
.jdkspecific:before {
    display: block;
    flex-direction: column-reverse;
    margin-top: 0px;
    align-items: baseline;
    float: right;
    margin-bottom: 0px;
}

.jdkspecific.jdk8:before {
    content: "JDK8";
    margin-right: -4.5em;
}
.jdkspecific.jdk11:before {
    content: "JDK11";
    margin-right: -5em;
}
.jdkspecific.jdk17:before {
    content: "JDK17";
    margin-right: -5em;
}
.jdk11.jdk17:before {
    content: "JDK11 JDK17";
    margin-right: -8em;
}
.jdk8.jdk11.jdk17:before {
    content: "JDK8 JDK11 JDK17";
    margin-right: -14em;
}

div.jdkspecific h5 {
    right: 0;
    margin: 0 0 0 0;
}

div.quote p {
    font-size: 1.5em;
    font-style: italic;
    text-align: center;
}
</style>
<script>
var toggles = [];
function setVisibilty(id, show) {
  
  const collection = document.getElementsByClassName(id);
  if(collection) {
    for (let i = 0; i < collection.length; i++) {
      if (show) {
        toggles[id] == "block";
        collection[i].style.display = "block";
      } else {
        collection[i].style.display = "none";
      }
    }
  }
}

function toggleJDKViews() {
  var cb = document.getElementById("jdk8_cb");
  if(cb.checked) {
    setVisibilty("jdk8", true);
  } else {
    setVisibilty("jdk8", false);
  }
  cb = document.getElementById("jdk11_cb");
  if(cb.checked) {
    setVisibilty("jdk11", true);
  } else {
    setVisibilty("jdk11", false);
  }
  cb = document.getElementById("jdk17_cb");
  if(cb.checked) {
    setVisibilty("jdk17", true);
  } else {
    setVisibilty("jdk17", false);
  }
}

</script>
<br>
  <p align="center">
    <img alt="Photo of a tape recorder" 
         src="/images/using-the-jdk-flight-recorder-img/markus-spiske-gnhxvdGmGG8-unsplash.jpg" 
         style="box-shadow: 0px 0px 20px 0px rgba(0,0,0,0.5);" width="75%"/>
  <br>
    <span>Photo by <a href="https://unsplash.com/@markusspiske?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Markus Spiske</a>.</span>
  </p>
<br>
  
The JDK Flight Recorder, JFR, is an integral part of the HotSpot JVM. I'm assuming you kind of know what JFR is, but in short it collects data about the JVM as well as the Java application running on it. 

This blog post is a quick guide on how to use JFR. To try and be less confusing on the slight differences between mostly JDK8 and newer versions, you can choose your flavor using the buttons below to hide/show relevant parts. To further confuse everything, if you're on OpenJDK8, JFR works more like it does on JDK11 (Open or not) and above, since it was backported from 11 to 8. If you're however on Oracle JDK8, it slightly differs from OpenJDK8 and JDK11 and up. 

  <label for="jdk8_cb">JDK8</label> <input id="jdk8_cb" type="checkbox" name="jdk8_cb" checked onchange="toggleJDKViews()">
  <label for="jdk11_cb">OpenJDK 8/JDK 11 and above</label> <input id="jdk11_cb" type="checkbox" name="jdk11_cb" checked onchange="toggleJDKViews()">
  <!--label for="jdk17_cb">JDK 17-</label> <input id="jdk17_cb" type="checkbox" name="jdk17_cb" checked onchange="toggleJDKViews()"-->


-----
There are basically two ways to start a JFR recording, either at startup by supplying command-line arguments, or during runtime by using the `jcmd` tool. You first however must make sure you have JFR enabled... or do you? Actually, since JDK 8u40 (i.e. since 2015) you don't have to explicitly enable it on startup.

# Using the JDK Flight Recorder
One relatively common misconception is that you have to restart the JVM if you haven't enabled JFR on startup. This is however not true, you can easily enable JFR in a running process using `jcmd`.

<div class="quote" markdown="1">
You don't have to restart your JVM to enable JFR
</div>

With `jcmd`, you can control various features in a running JVM. You can either use the Process ID (pid), or target any JVM running a particular main-class. In order to get the pid, you can run `jcmd` without arguments to get a list of all running Java applications. 

```
$ jcmd
27016 MicronautServer-1.0.jar
29784 jdk.jcmd/sun.tools.jcmd.JCmd
```

## Starting and Saving Recording with jcmd
In essence the three JFR commands `start`, `dump`, and `stop` are all you need.

```
$ jcmd <pid> JFR.start name=rec
$ jcmd <pid> JFR.dump name=rec
$ jcmd <pid> JFR.stop name=rec
```

<div class="jdkspecific jdk8" markdown="1">
In Oracle JDK8, JFR is a commercial feature, meaning that you either have to add `-XX:+UnlockCommercialFeatures` to the command line when starting the JVM, or you can unlock commercial features in runtime using `jcmd` and the `VM.unlock_commercial_features` command.

```
$ jcmd <pid> VM.unlock_commercial_features
```

With commercial options unlocked, you can go ahead and start your recording.
</div>

You start a recording with the command `JFR.start`.

```
$ jcmd <pid> JFR.start name=rec
```

The `name` argument is an optional identifier for the recording. If not defined the JVM will choose a numerical identifier, which you'll need to supply when using `JFR.dump` or `JFR.stop`.

<div class="jdkspecific jdk8" markdown="1"> 
For JDK 8 you need to specify the numerical identifier using `recording=x` .  
</div>
<div class="jdkspecific jdk11" markdown="1"> 
In JDK11 and above you specify the numerical id using `name=x`.
</div>
In order to get a running recording written to file, you use the `JFR.dump` command.


<div class="jdkspecific jdk8" markdown="1"> 
In JDK8 you must define either `recording` or `name` to the `JFR.dump` command, as well as a defined `filename`.
```
$ jcmd <pid> JFR.dump name=rec filename=recording.jfr
```

</div>

<div class="jdkspecific jdk11" markdown="1"> 
In JDK11 and above if you omit the `name` from `JFR.dump` all recordings will be dumped. If you omit the filename, an automated generated filename will be used
```
$ jcmd <pid> JFR.dump name=rec filename=recording.jfr
```
</div>

To stop a recording, you use the `JFR.stop` command. If you also add the `filename` argument to the command, the recording is also saved to disk.

```
$ jcmd <pid> JFR.stop name=rec
```


<br>
  <p align="center">
    <img 
    alt="Conceptual image showing the timeline of a JVM, with JFR registering events first when 'JFR.start' is issued. Actual recordings are saved and persisted to disk using 'JFR.dump', or 'JFR.stop'; the latter also stopping the recording." 
         src="/images/using-the-jdk-flight-recorder-img/recordings-1.png" 
         width="100%"/>
  <br>
    <span>Conceptual image of a JVM process where a recording was started using JFR.start, and then written to file (rec1.jfr) with JFR.dump, and finally stopped with a final write to file (rec2.jfr).</span>
  </p>
<br>

### Jcmd JFR options
The options for each of the JFR commands can be printed using the `jcmd` help command.

```
$ jcmd <pid> help JFR.start
```

Below are some of the more commonly used, and important options:


| Option      | Description |
| :---      | :--- |
|       `name` | Name that can be used to identify recording, e.g. \"My Recording\" |
|        `filename` | Resulting recording filename, e.g. \"C:\Users\user\My Recording.jfr\" |
|        `delay` | Delay recording start with (s)econds, (m)inutes), (h)ours), or (d)ays, e.g. 5h.|
|        `duration` | Duration of recording in (s)econds, (m)inutes, (h)ours, or (d)ays, e.g. 300s. |
|       `dumponexit` | Dump running recording when JVM shuts down |
|        `disk` | Recording should be persisted to disk |
|        `maxage` | Maximum time to keep recorded data (on disk) in (s)econds, (m)inutes, (h)ours, or (d)ays, e.g. 60m, or 0 for no limit|
|        `maxsize` | Maximum amount of bytes to keep (on disk) in (k)B, (M)B or (G)B, e.g. 500M, or 0 for no limit |

## Starting a Flight Recording at JVM Startup
A common use-case is to have a so-called default recording upon starting the JVM. The perks of this is you don't have to do anything manually, and you can have JFR dump a recording whenever the JVM decides to exit for whatever reason -- giving you ideas of what that "whatever" was.

To start a recording when the JVM starts you use the `-XX:StartFlightRecording` argument. The options are basically the same as when starting using `jcmd` shown above.

<div class="jdkspecific jdk8" markdown="1">
### JDK 8
For JDK 8 you need to unlock commercial features in order to use `StartFlightRecording`.


```
> java -XX:+UnlockCommercialFeatures -XX:StartFlightRecording=name=default,filename=dump.jfr,dumponexit=true,maxsize=250M -jar JettyServer-1.jar
Started recording 1.

Use JFR.dump name=default to copy recording data to file.
```

The default maxsize is currently unlimited in JDK 8 when using `StartFlightRecording=name=default`. This means you could fill up your temporary directory, unless you give an explicit size or age boundary using `maxsize` or `maxage`. In the example above I've chosen to set `maxsize` explicitly to 250 MB, which is the default for later JDK versions.

If you're however used to starting your default recording with `FlightRecordingOption=defaultrecording=true`, the recording will get a default `maxage` of 15 minutes.

</div>

<div class="jdkspecific jdk11" markdown="1">
### JDK 11 and Above
In JDK 11 and later you just add the `StartFlightRecording` argument.
```
> java -XX:StartFlightRecording=filename=dump.jfr,dumponexit=true -jar MicronautServer-1.jar
```
</div>

With the `dumponexit` argument set to true, you'll get a recording when the JVM exits. 

If you want to dump the recording at any other time, you can use the `JFR.dump` command together with a filename.
```
> jcmd <PID> JFR.dump name=default filename=ongoing-recording.jfr
```


<br>
  <p align="center">
    <img 
    alt="The default recording registers events from the beginning of the JVM startup, and these events can be saved to file using 'JFR.dump' and the default recording's name. Another recording can be started in parallel using 'JFR.start' and corresponding 'JFR.dump', or 'JFR.stop'. At JVM exit, the default recording is saved to disk, containing only as much data as defined by 'maxage' or 'maxsize'" 
         src="/images/using-the-jdk-flight-recorder-img/recordings-2.png" 
         width="100%"/>
    <span>In this image a default recording is started when the JVM starts with the `StartFlightRecording` option. One can see that the default recording is active throughout the JVM's lifetime (from 00:01 to 00:27). <br/>
    Somewhere in the middle we start another recording named "`rec`" (the yellowish blocks) for a shorter period, which we first dump to `rec1.jfr`, and then stops with a write to `rec2.jfr`. <br/>
    After some time we also write the default recording to `def.jfr` using JFR.dump, and at the JVM exit, the default recording is written to `dump.jfr`</span>
  <br>
  </p>
<br>



### StartFlightRecording Options
Most options to `StartFlightRecording` can be recognized from the `JFR.start` command. 

| Option      | Description                               | 
| :---        |    :----                                  | 
| name        | Optional name to identify the recording   | 
| delay       | Start recording after a delay            | 
| duration    | Only record for a certain time            | 
| filename    | Filename where recording should be stored | 
| disk    | Specifies whether to write data to disk while recording.                    | 
| maxage    | List of settings files                    | 
| maxsize    | List of settings files                    | 


## Some Often Misunderstood Options
There are a few options that can cause confusion; the options `disk`, `maxage` and `maxsize`. Setting `disk` to true will make JFR store temporary data on disk instead of in-memory. Storing temporary data on disk will give you more data for the recording, as opposed to storing in-memory, since the default in-memory buffers are by default smaller. The two latter, `maxage` and `maxsize` are only used if `disk=true`. These settings control how much _temporary_ data JFR should store, they should *not* be used to "filter" the data. The `maxage` defines how long events are stored in the temporary directory; if you use `maxage=1h` you'd get data that is atleast 1 hour old. You could also get data older than 1 hour, if for instance the event fits within the chunks of 12 MB that JFR defaults to, or if the event is a long running event. Setting `maxsize` limits how much data is stored in the temporary directory, and with the default chunksize, it only makes sense to set it above 12 MB.

If you want a recording for a limited time period, you should use the `duration`, and possibly `delay` option.

  <p align="center">
    <img 
        alt="When setting delay=5m, and duration=15m, the recording will start after 5 minutes, and record for 15 minutes." 
         src="/images/using-the-jdk-flight-recorder-img/deldur.png" 
         width="100%"/>
  <br>
    <span>When setting `delay=5m`, and `duration=15m`, the recording will start after 5 minutes, and record for 15 minutes.</span>
  </p>

For instance, if you issue the below `JFR.start` jcmd at 08:00, the `delay=4h` will make your recording start at 12:00, and `duration` will give you 15 minutes worth of JFR data in the `diagnosis.jfr` file. The `delay` option can also be used just to delay recording only a few seconds in order to get the JVM to "warm up", as to not record JVM internal processes such as JIT compilation, etc -- depending on your needs.

```
jcmd 27016 JFR.start delay=4h,duration=15m,filename=diagnosis.jfr
```

## Reference
A good starting page for JFR related documentation is the [Oracle Java components](https://docs.oracle.com/en/java/java-components/) landing page, where you continue to the [JDK Mission Control (JMC) documentation](https://docs.oracle.com/en/java/java-components/jdk-mission-control/). In general you should always be using the latest JMC (8 as of writing) for analyzing JFR files, regardless of which JDK version you're using. So choose JMC 8 in the drop-down, and then consult the right hand links on the JMC documentation page, "Java/JDK Flight recorder" for your JDK.

* [docs.oracle.com/en/java/java-components/jdk-mission-control/](https://docs.oracle.com/en/java/java-components/jdk-mission-control/)

