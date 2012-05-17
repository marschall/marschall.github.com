---
layout: post
title: SLF4J-Equinox
---

There are all these jokes about logging, logging frameworks and logging bridges in Enterprise Java. To my amazement I found out there is still a logging bridge that hasn't been written so I wrote [slf4j-equinox](https://github.com/marschall/slf4j-equinox). How has it come to this?

Let's start at the beginning. It's generally a good practice if a 3rd party framework doesn't use a specific logging implementation but rather an abstract logging API. This makes it possible to handle logging in a unified way throughout an application. For several years the logging API of choice was [commons-logging](http://commons.apache.org/logging/). If a 3rd party framework uses commons-logging the application can make it use a concrete logging implementation like [log4j](http://logging.apache.org/log4j/) or [java.util.logging](http://docs.oracle.com/javase/7/docs/api/java/util/logging/package-summary.html) without having to change the code of the 3rd party framework. However commons-logging logging fell out of favor because of its problematic [runtime discovery mechanism](http://articles.qos.ch/classloader.html). [SLF4J](http://www.slf4j.org/) was created to solve the same problem as commons-logging but with a different runtime discovery mechanism. Today SLF4J is the logging API of choice. Only very [few projects](http://static.springsource.org/spring/docs/3.1.x/spring-framework-reference/html/overview.html#d0e748) don't use it.

So all is well? Not quite. While there are many bridges from SLF4J to concrete logging implementations a crucial one is missing. There is no good solution for Eclipse RCP applications. You can run logging libraries like [logback](http://logback.qos.ch/) inside Eclipse RCP applications but that means you run an additional logging library besides the one that's included in the core framework ([Equinox](http://www.eclipse.org/equinox/)). So what's needed is a library that makes SLF4Js log messages to go [Equinox's extended log service](https://bugs.eclipse.org/bugs/show_bug.cgi?id=260672). This is what slf4j-equinox does. As a consequence all log messages sent to SLF4J end up in <code>.metadata/.log</code>.

Hopefully I'll find some time to blog about the implementation. In the mean time you're encouraged to fork.




