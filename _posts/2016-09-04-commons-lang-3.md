---
layout: post
title: 'Apache Commons Lang 3.5: Two Steps Forward in Performance'
---

When [Apache Commons Lang](https://commons.apache.org/proper/commons-lang/) [3.5](https://issues.apache.org/jira/browse/LANG/fixforversion/12331955/) will eventually be released it will contain among others two performance improvements.

First [LANG-1218](https://issues.apache.org/jira/browse/LANG-1218) will allow `EqualsBuilder#append(Object,Object)` to be inlined. This in turn will allow `EqualsBuilder` to be scalarized. You should see a reduction in allocation rate and an improvement in performance of up to a factor of three. It is important to note that this improvement only works as long as you're not calling `EqualsBuilder#append(Object,Object)` with an argument of static type `Object` and a dynamic type of array.

Second [LANG-1230](https://issues.apache.org/jira/browse/LANG-1230) allows multiple threads to call `#reflectionEquals` and `#reflectionHashCode` concurrently. You should see a reduction in lock contention and an improvement in throughput.


