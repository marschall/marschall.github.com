---
layout: post
title: Better Windows Networking with JDK 8u60 
---

JDK 8u60 contains two features that should improve the networking performance of Java applications on Windows under certain conditions.

 * [JDK-8060170](https://bugs.openjdk.java.net/browse/JDK-8060170) makes Java use a loopback interface similar to Unix for localhost TCP connections.
 * [JDK-8064407](https://bugs.openjdk.java.net/browse/JDK-8064407) makes Java use a system call similar to <code>sendfile()</code> when <code>FileChannel.transferTo()</code> is used.

These two patches have been contributed by [Microsoft Open Technologies, Inc.](https://msopentech.com/).

For now tese features are not enabled by default and have to be enabled by setting the following system properties to <code>true</code>.

<pre><code>-Djdk.net.useFastTcpLoopback=true \
-Djdk.net.enableFastFileTransfer=true</code></pre>

