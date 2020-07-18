---
layout: post
title: How to Headless Flight Recorder 
---

This is more a documentation for myself on how to run [Flight Recorder](http://www.oracle.com/technetwork/java/javaseproducts/mission-control/java-mission-control-1998576.html) headless on start up. You will have to use Template Manager im Mission Control first to create a settings file.

<pre><code>-XX:+UnlockDiagnosticVMOptions \
-XX:+DebugNonSafepoints \
-XX:StartFlightRecording=duration=120s,filename=recording.jfr,settings=settings.jfc</code></pre>

Sources
-------

 * [Java Mission Control Documentation](http://docs.oracle.com/javacomponents/jmc-5-4/jfr-runtime-guide/run.htm#CHDIDCHG)
 * [Creating Flight Recordings](http://hirt.se/blog/?p=370)
 * [Improving the Fidelity of the JFR Method Profiler](http://hirt.se/blog/?p=609)


