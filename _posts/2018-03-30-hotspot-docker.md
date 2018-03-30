---
layout: post
title:  HotSpot Ergonomics and Docker
published: false
---

docker run -it --rm -m 128m marschall/oracle-server-jre:8 java -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:+PrintFlagsFinal -version | grep MaxHeapSize


