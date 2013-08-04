---
layout: post
title: Webapp classloaders: are we there yet? 
---

Much has been written about classloader issues in Java web applications. One way to classify them is into two types, conflicting libraries in the web application itself and conflicting libraries from "somewhere else" — either the application server or an other application in the same server. The first is "your fault" and a packaging issue by the application. The server can do nothing to fix this. The second is either a server bug or a server configuration issue (some servers can be configured to make certain libraries available).

To avoid as many issues a possible a server should make only as few classes as possible available to the application, ideally only the API classes form the JavaEE specifications. Especially a server should not make libraries it uses itself availble to an application. Making them available will likely cause issues when the application uses a different version of the same library. After debugging an issue in this area I chose to invesitage which application servers make the (Eclipse JDT compiler)[http://www.eclipse.org/jdt/core/] which is used for JSP compilation visible to the application. If your application packages its own version of the Eclipse JDT compiler this will likely result in a conflict. For example the (Apache Cocoon)[http://cocoon.apache.org] framework depends on the Eclipse JDT compiler.

| Server             | JDT visible | 
| -----------------: | ----------- |
| Tomcat 7           | yes         |
| TomEE 1.5.2        | yes         |
| JBoss AS 7.1.1     | yes         |
| JBoss EAP 6.1      | yes         |
| WildFly AS Alpha 8 | no          |
| GlassFish 4        | no \(*)\    |
| Jetty 9.0.4        | yes         |
| Resin 4.0.36       | no          |
| Geronimo 3.0.1     | no          |
 
\(*\) The GlassFish JSP compiler does not use the Eclipse JDT compiler but makes its JSP compiler available nevertheless

As we can see most servlet containers and application servers make the Eclipse JDT compiler visible to the application. The ones doing best seem to be ones getting the least attention. This is a case where application servers with some kind of module system can play their strengths.

(WFLY-1770)[https://issues.jboss.org/browse/WFLY-1770]

