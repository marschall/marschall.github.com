---
layout: post
title: How to Headless Flight Recorder 
---

This is more a documentation for myself on how to run [Flight Recorder](http://www.oracle.com/technetwork/java/javaseproducts/mission-control/java-mission-control-1998576.html) headless on start up. You will have to use Template Manager im Mission Control first to create a settings file.

<pre><code>-XX:+UnlockDiagnosticVMOptions \
-XX:+DebugNonSafepoints \
-XX:StartFlightRecording:duration=120s,filename=recording.jfr,settings=settings.jfc</code></pre>

If you want to use the built in profiling template then use `settings=profile`.

Sources
-------

 * [Troubleshoot with Java Mission Control](https://docs.oracle.com/en/java/javase/11/troubleshoot/diagnostic-tools.html#GUID-6786AA0B-B474-4E28-A9D6-75C31D37CCB7)
 * [Creating Flight Recordings](http://hirt.se/blog/?p=370)
 * [Improving the Fidelity of the JFR Method Profiler](http://hirt.se/blog/?p=609)


