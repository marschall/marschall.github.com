---
layout: post
title:  HotSpot Ergonomics and Docker
published: false
---

When no maximum heap size is specified using `-Xmx` the Java HotSpot Virtual Machine uses (ergonomics)[https://docs.oracle.com/javase/9/gctuning/ergonomics.htm#JSGCT-GUID-DA88B6A6-AF89-4423-95A6-BBCBD9FAE781] to set the maximum heap size.  In partice this means one quarter of the physical memory. On Java 8 per default cgroups memory limits are ignored. We can observe this using

```
docker run -it --rm -m 8g openjdk:8 java -XX:+UseParallelGC -XX:+PrintFlagsFinal -version | grep MaxHeapSize
    uintx MaxHeapSize                              := 16873684992                         {product}
```

On a machine with 64 gigabyte of memory we get a maximum 16 gigabyte heap even though we set the memory limit to 8 gigabytes. Later versions of Java 8 can be made to respect (cgroup limits)[https://blogs.oracle.com/java-platform-group/java-se-support-for-docker-cpu-and-memory-limits]

```
docker run -it --rm -m 8g openjdk:8 java -XX:+UseParallelGC -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:+PrintFlagsFinal -version | grep MaxHeapSize
    uintx MaxHeapSize                              := 2147483648                          {product}
```

Now the cgroup limit of 8 gigabytes as regarded as the amount of available memory and per default we get a maximum heap of one quarter of this with results in a maximum 2 gigabyte heap. Java 10 does this by default

```
docker run -it --rm -m 8g openjdk:10 java -XX:+UseParallelGC -XX:+PrintFlagsFinal -version | grep MaxHeapSize
   size_t MaxHeapSize                              = 2147483648                               {product} {ergonomic}
```

Java 10 allows for easy control of what percentage of memory should be used for the heap using (MaxRAMPercentage)[https://bugs.openjdk.java.net/browse/JDK-8186248]

```
docker run -it --rm -m 8g openjdk:10 java -XX:+UseParallelGC -XX:MaxRAMPercentage=75 -XX:+PrintFlagsFinal -version | grep MaxHeapSize
   size_t MaxHeapSize                              = 6442450944                               {product} {ergonomic}
```

and we end up with a 6 gigabyte heap, 75% of 8 gigabytes.

Having that said I would still recommend sizing the heap using `-Xmx` based on the live set and sizing the docker container based on the memory requirements of the JVM rather than the other way around.


