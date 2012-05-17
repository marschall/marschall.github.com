---
layout: post
title: AJP Introduction
---

In this post we'll have a high level look at the  JServ Protocol version 1.3 (aka AJP aka ajp13). We'll also compare it to other, similar protocols. But we won't discuss the low level details. 

AJP is a protocol for connecting a front end web server to a back end web server.
 
    HTTP(S) → Front End ← AJP → Backend

Why is there a distinction between a front end and a back end web server at all? Traditionally a front end web server is a "classical" web server meaning designed to serve static files. These are usually daemons written in C like [Apache httpd](http://httpd.apache.org/) or [Ngix](http://nginx.org/). Generating dynamic content is usually left to a server written in a high level language.

This leaves the question on how the front and back end servers should be connected.

CGI
---
Back in the day a common way was [CGI](http://en.wikipedia.org/wiki/Common_Gateway_Interface). Meaning server would just invoke an executable that would generate the dynamic content. While simple this approach has several disadvantages. Slow work that could in theory only be done once like parsing source files or VM start up has do be done on every request. Database connection pooling is not possible at all. Caching values between invocations requires an external daemon like memcached. However on the positive it's not an issue when an application leaks memory because the process is terminated after each request.

In Process
----------
One way to address the shortcomings of CGI is to run as a module in-process in the front end web server like [mod_perl](http://perl.apache.org/), [mod_python](http://www.modpython.org/) or [mod_php](http://dan.drydog.com/apache2php.html). This addresses the issues of CGI mentioned in the previous paragraph and allows for much tighter integration with the web server. However this also affects the resource consumption and stability of the front end server negatively.

FastCGI
-------
[FastCGI](http://www.fastcgi.com/drupal/node/6?q=node/15) tries to avoid the downsides of CGI (high request overhead) and in process modules (missing isolation) that the same time. FastCGI introduces a protocol that is used by the front end web server and the back end processes that generate the dynamic content. The back end processes are kept alive over several requests. On the down side FastCGI requires an additional Apache module. While this is not a big deal on a Linux developer workstation it can be an issue on production machines that forbid the installation of a tool chain (eg. PCI-DSS). AJP had the same issue until Apache 2.2. Before that mod_jk had to be installed.

SCGI
----
[SCGI](http://www.python.ca/scgi/protocol.txt) is similar to FastCGI but designed to be simpler to implement. One of the biggest differences is that SCGI opens a new TCP connection for every request. This negatively impacts its performance.

HTTP(S)
-------
What we have seen more and more in recent years is using HTTP(S) as a protocol for connecting web servers. While it may seem like an intuitive solution at first it has some disadvantages. First some information gets lost. The original HOST header is no longer available unless the front end server is specially configured. The IP address and port of the user agent making the original request also gone. As a work around some servers add the option to insert a X-Forwarded-For header. Also SSL/TLS information (session id, cypher suite, cipher, key size) is gone. It's not even possible to find out whether the original request used SSL/TLS at all. Second lot of optimization potential like compressing the HTTP header is not used.

AJP
---
AJP can be seen as another alternative to FastCGI. Compared to other protocols AJP combines the following advantages:
 * Persistent TCP connections between the front end and back end server are used.
 * [mod_proxy_ajp](http://httpd.apache.org/docs/2.4/mod/mod_proxy_ajp.html) is part of Apache from 2.2 onwards. AJP is now as easy to set up as a reverse HTTP proxy. You only need to change the protocol from <code>http://</code> to <code>ajp://</code>.
 * mod_proxy_ajp works together with [mod_proxy_balancer](http://httpd.apache.org/docs/2.4/mod/mod_proxy_balancer.html) to support sessions aware load balancing without modifying the response.
 * Common HTTP header names are compressed to two bytes.
 * Values come pre-parsed in XDR-like <code>type-length-value</code> format making them easy and fast to parse.
 * Information about the original connection like IP address and port is available.
 * SSL/TLS information about the original connection like session id, cipher and key size is available
 * Apache variables starting with <code>AJP_</code> are available.
 * It's supported by third party tools like Wireshark.

Further Reading
---------------
 * [AJPv13](http://tomcat.apache.org/connectors-doc/ajp/ajpv13a.html) describes the AJP13 protocol and goes into some of its history
 * [AJPv13 extensions Proposal](http://tomcat.apache.org/connectors-doc/ajp/ajpv13ext.html) proposes extensions in areas where AJP13 is currently lacking
 * [Deciding between mod_jk, mod_proxy_http and mod_proxy_ajp](http://www.tomcatexpert.com/blog/2010/06/16/deciding-between-modjk-modproxyhttp-and-modproxyajp)

