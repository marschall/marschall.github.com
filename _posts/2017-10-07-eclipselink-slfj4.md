---
layout: post
title: EclipseLink Logging over SLF4J
---

Starting with [EclipseLink 2.7.0](http://www.eclipse.org/eclipselink/releases/2.7.php) bugs [296391](https://bugs.eclipse.org/bugs/show_bug.cgi?id=296391) and [494612](https://bugs.eclipse.org/bugs/show_bug.cgi?id=494612) are fixed which means that EclipseLink now has out of the box support for logging over [SLF4J](https://www.slf4j.org).

You need the following dependency

```xml
<dependency>
  <groupId>org.eclipse.persistence</groupId>
  <artifactId>org.eclipse.persistence.extension</artifactId>
  <version>2.7.0</version>
</dependency>
```

and you need to set the following persistence property

```xml
<property name="eclipselink.logging.logger" value="org.eclipse.persistence.logging.slf4j.SLF4JLogger"/>
```

