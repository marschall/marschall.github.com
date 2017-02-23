---
layout: post
title: Work Around SECURITY-864
---

If you're using WildFly and EJB security you'll likely run into [SECURITY-864](https://issues.jboss.org/browse/SECURITY-864). The symptom is a `javax.naming.NameNotFoundException` happening on every EJB invocation. You won't see them in your logs unless you're using the `TRACE` log level but they will show up in monitoring tools like [Java Mission Control](http://www.oracle.com/technetwork/java/javaseproducts/mission-control/java-mission-control-1998576.html) or [OverOps](https://www.overops.com).

As a work around we provide [policy-registration-factory](https://github.com/marschall/policy-registration-factory) which allows you to register a [ObjectFactory](https://docs.oracle.com/javase/8/docs/api/javax/naming/spi/ObjectFactory.html) under `java:/policyRegistration`. You have the choice between a real one and a stub one. The project pages goes into details about the installation.

