---
layout: post
title:  Reducing WildFly Startup Time and Footprint with AppCDS
---

Application Class-Data Sharing, or AppCDS for short, is available in OpenJDK 10 as per [JEP 310](http://openjdk.java.net/jeps/310). Its goals are to reduce application startup time and reduce the memory overhead of running multiple instances of the same Java application on the same host.

I'll be heavily relying on Volker Simonis [Class Data Sharing](https://fosdem.org/2018/schedule/event/class_data_sharing/) talk at FOSDEM 2018 and his [cl4cds](https://simonis.github.io/cl4cds/) tool. I recommend checking them out. 

To get started you need to download an [OpenJDK 10 early access build](http://jdk.java.net/10/). It's important not to download an OracleJDK as AppCDS is missing there.

The first step is to dump the list of loaded classes

```
export PREPEND_JAVA_OPTS="-Xlog:class+load=debug:file=/tmp/wildfly.classtrace"
./bin/standalone.sh
```

This is followed by converting that to a class list suitable for AppCDS, this is where the cl4cds tool comes in

```
$JAVA_HOME/bin/java -jar ~/git/cl4cds/target/cl4cds-1.0.0-SNAPSHOT.jar /tmp/wildfly.classtrace /tmp/wildfly.cls
```

Then the shared class archive can be created

```
export PREPEND_JAVA_OPTS="-Xshare:dump -XX:+UseAppCDS -XX:SharedClassListFile=/tmp/wildfly.cls -XX:+UnlockDiagnosticVMOptions -XX:SharedArchiveFile=/tmp/wildfly.jsa"
./bin/standalone.sh
```

and finally WildFly can be started with the shared archive

```
export PREPEND_JAVA_OPTS="-Xshare:on -XX:+UseAppCDS -XX:+UnlockDiagnosticVMOptions -XX:SharedArchiveFile=/tmp/wildfly.jsa"
./bin/standalone.sh
```

On my machine the startup time reported by WildFly went down from about 2000ms to about 1500ms, an improvement of 25%. It has to be noted that this was an empty WildFly with no deployment. If you have a deployment more WildFly classes would be loaded as well as classes from your deployment. Including classes of your deployment in a shared archive improves startup time even more, however on the downside you then have to create a new shared archive every time you create a new version of your deployment.


