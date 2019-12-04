---
layout: post
title: TLS Debugging with JFR
---

Sometimes when debugging SSL/TLS connection issues in Java [JSSE debug logging](https://docs.oracle.com/en/java/javase/11/security/java-secure-socket-extension-jsse-reference-guide.html#GUID-31B7E142-B874-46E9-8DD0-4E18EC0EB2CF) may not be available because it requires a JVM restart and a a change to JVM arguments. In such cases [Java Flight Recorder (JFR) Security Events](https://bugs.openjdk.java.net/browse/JDK-8148188) may be used. JFR Security Events are Java available in Java 12+, 11.0.5+ and 8u231+. To generate JFR events you need a configuration file like this one: 


```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration version="2.0" label="TLS Debugging" description="JFR Events for TLS debugging" provider="marschall.github.io">

  <event name="jdk.SecurityPropertyModification">
    <setting name="enabled">true</setting>
    <setting name="stackTrace">true</setting>
  </event>

  <event name="jdk.TLSHandshake">
    <setting name="enabled">true</setting>
    <setting name="stackTrace">true</setting>
  </event>

  <event name="jdk.X509Validation">
    <setting name="enabled">true</setting>
    <setting name="stackTrace">true</setting>
  </event>

  <event name="jdk.X509Certificate">
    <setting name="enabled">true</setting>
    <setting name="stackTrace">true</setting>
  </event>

</configuration>
```

Analysis may not be very comfortable as the information available is limited.

![all JFR TLS events](/img/2019-12-04-tls-debugging-with-jfr/all_events.png)

So it may pay to create a page with all certificates, this way you can search by certificate id.

![page with certificate events](/img/2019-12-04-tls-debugging-with-jfr/page_with_certificates.png)

Extensions like [SAN](https://en.wikipedia.org/wiki/Subject_Alternative_Name) are not available so you may to to inspect the certificate with a different tool.

Demo code can be found under [jfr-handshake](https://github.com/marschall/jfr-handshake).
