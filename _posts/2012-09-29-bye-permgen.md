---
layout: post
title: Bye PermGen
---
The [latest JKD 8 builds](http://download.java.net/jdk8/changes/jdk8-b58.html) no longer have PermGen. I remember back in I believe 2010 at either an [EclipseCon](http://www.eclipsecon.org/) or [Devoxx](http://www.devoxx.com/) where Oracle was talking about their future VM strategy. The future VM was to be [HotSpot](http://www.oracle.com/technetwork/java/javase/tech/index-jsp-136373.html) with parts from [JRockit] (http://www.oracle.com/technetwork/middleware/jrockit/overview/index.html) ported over. Jokingly this VM war referred to as HotRockit or JSpot. One of the parts to be ported was the removal of PermGen. It's nice to see tech companies deliver on their promises.

Does this Mean an End to java.lang.OutOfMemoryError: PermGen space?
-------------------------------------------------------------------
Yes. Now class metadata and interded strings are in [native memory](http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=6962931). In fact if you start the latest HotSpot with <code>-XX:MaxPermSize=256m</code> you will get the following error message:

    Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=256m; support was removed in 8.0

Does this Mean an End to ClassLoader Leaks?
-------------------------------------------
No. There still are circular references between [Class](http://docs.oracle.com/javase/7/docs/api/java/lang/Class.html) and [ClassLoader](http://docs.oracle.com/javase/7/docs/api/java/lang/ClassLoader.html). There is a good reason for this in the java language specification section [12.7. Unloading of Classes and Interfaces](http://docs.oracle.com/javase/specs/jls/se7/html/jls-12.html#jls-12.7). So leaks will stay, you'll just leak native memory instead of PermGen.

The move doesn't seem complete at this point as [VisualVM](http://visualvm.java.net/) still reports a non-empty PermGen.
