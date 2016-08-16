---
layout: post
title: Setting Logstash JVM Options
---

Disclaimer: Elastic does not recommend [changing](https://www.elastic.co/guide/en/elasticsearch/guide/current/_don_8217_t_touch_these_settings.html) Logstash garbage collection settings.

With that out of the way there are situations in which you may want to change Logstash garbage collection settings. In our case we run three JVMs plus Logstash on a four core server. Logstash is the one with the most garbage collection overhead and uses four threads to do garbage collection in the default configuration on that machine. That means at critical times all four cores may be busy doing Logstash garbage collection. Ideally we would not run Logstash on this server and forward the logs through other means. However this was not an option in this case for various reasons.
We want to switch Logstash to the [serial collector](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/collectors.html) so that at most one core is used for Logstash garbage collection and the others are free for actual application processing. In other cases G1 may give you better results than the default CMS. Disclaimer: Elastic does not [recommend or support](https://www.elastic.co/blog/a-heap-of-trouble) using G1.

Simply setting `LS_JAVA_OPTS` will not work because that only appends to the default Logstash JVM Options and does not replace them. You can set `JAVA_OPTS` but that will not work unless you also est `HEAP_DUMP_PATH`. Below is the final configuration we use.

```sh
# replace logstash Java options since we only have 4 cores in production
# you will get a waring in the logs
# LS_JAVA_OPTS only appends
export JAVA_OPTS="-XX:+UseSerialGC -Djava.awt.headless=true -XX:+HeapDumpOnOutOfMemoryError"
# logstash will append Xmx
export LS_HEAP_SIZE="1g"
# this needs to be set because logstash will always append it
# if it's missing you will get an empty VM argument and it won't start
export HEAP_DUMP_PATH="-XX:HeapDumpPath=$LOGSTASH/heapdump.hprof"
```


