---
layout: post
title: Disabling the Gradle Daemon
---

Gradle 3 will ship with the [Gradle Daemon](https://docs.gradle.org/current/userguide/gradle_daemon.html) [enabled by default](https://gradle.org/blog/gradle-3-0-m1-unleash-the-daemon/). While this may be fine for people who regularly use Gradle if you just occasionally build something with Gradle that means you now have a background process that can use up to 1 GB of heap plus additional JVM memory. The initial heap used is your platform default. You can check that with
```
java -XX:+PrintFlagsFinal -Xmx1g -version 2>&1  | grep InitialHeapSize
```
on my machine that's 256 MB. An easy way to check if the Gradle Daemon is running on your machine is to use `jcmd`.


To disable the Gradle Daemon add the following line to `~/.gradle/gradle.properties`
```
org.gradle.daemon=false
```



