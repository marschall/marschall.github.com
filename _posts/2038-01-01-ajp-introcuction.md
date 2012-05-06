---
layout: post
title: AJP Introduction
published: false
---

{{ page.title }}
================

In this post we'll have a high level look at the  JServ Protocol version 1.3 (aka AJP aka ajp13). We won't discuss the low level details. We'll also compare it to other, similar protocols.

AJP is a protocol for connecting a front end web server to a back end web server. 
  HTTP(S) -> Front end <- AJP -> Backend
Why is there a distinction between a front end and a back end web server at all? Traditionally a front end web server is a "classical" web server meaning designed to serve static files. These are usually deamons written in C take Apache or Ngix as an example. Generating dynamic content is usually left to a server written in a high level language.

This leaves the question on how the front and back end servers should be connected.

CGI
---
Back in the day a common way was CGI. Meaning server would just invoke an executable that would generate the dynamic content. While simple this approach has serveral disadvantages. Slow one time work like parsing source file or VM start up has do be done on every request. Database connection pooling is not possible at all. Caching values between invocations requires an external deamon like memcached.

In Process
----------
An other option would be to run as a module in-process in the front end web server. However this afffects the resource consumption and stability of the front end server negatively.

HTTP(S)
-------
What we have seen more and more in recent year is using HTTP(S) as protocol for connecting web servers. While may seem like an intuitive solution at first it has some disadvantages. First some information gets lost. The original HOST header is no longer available so some servers add the option to insert a X-Forwarded-For header. The IP address and port of the user agent making the original request also gone as well as all SSL information (session id, cypher suite, cipher, key size). It's not even possible to find out whether the original request used SSL at all. Second lot of optimization potential.

SCGI
---
Makes a new TCP connection for every request. This alone disqualifies it as a low performance solution.

FastCGI
-------
FastCGI requires an additional Apache module. While this is not a big deal on your Linux developer workstation it can be an issue on production machines that forbid the installation of a toolchain (eg. PCI-DSS). AJP had the same issue until Apache 2.2. Back then mod_jk had to be installed.

AJP
---
So how does AJP stack up against these other protcols?
 * persistent TCP connections are used
 * mod_proxy_ajp is part of Apache fron 2.2 onwards. AJP is now as easy to set up as a reverse HTTP proxy. You only need to change the protocol from http:// to ajp://. Yet it still supports sessions aware load balancing without modifying the response.
 * common HTTP header names are compressed to two bytes
 * values come pre-parsed in XDR-like type-length-value format making them easy and fast to parse
 * information about the original connection like IP and port is available
 * SSL information about the original connection like session id, cipher and key size is available
 * Apache variables starting with AJP_ are available
 * last but not least Wireshark supports AJP

Further Reading
---------------
 * [TAJPv13](http://tomcat.apache.org/connectors-doc/ajp/ajpv13a.html) describes the AJP13 protocol and goes into some of its history
 * [AJPv13 extensions Proposal](http://tomcat.apache.org/connectors-doc/ajp/ajpv13ext.html) proposes extesions in areas where AJP13 is currently lacking
 * [Apache Module mod_proxy_ajp](http://httpd.apache.org/docs/2.4/mod/mod_proxy_ajp.html) describes the protocol and the Apache module
 * [Deciding between mod_jk, mod_proxy_http and mod_proxy_ajp](http://www.tomcatexpert.com/blog/2010/06/16/deciding-between-modjk-modproxyhttp-and-modproxyajp)
 * []()

