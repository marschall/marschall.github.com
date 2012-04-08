---
layout: post
title: EvenSource Sample Published
---

{{ page.title }}
================

I published a simple example on how to use [EventSource/Server-Sent Events](http://dev.w3.org/html5/eventsource/) on GitHub ([marschall/event-source-sample](https://github.com/marschall/event-source-sample)). Unlike other (mostly PHP) examples this does not require a server thread per connection.

Compared to [WebSockets](http://en.wikipedia.org/wiki/WebSocket) EvenSource is much simpler and more «lightweight». They are only unidirectional (<code>server → client</code>) but since all the reconnection logic in implemented in the browser you can use them without [CometD](http://cometd.org/) or [Socket.IO](http://socket.io/). The example doesn't use any JavaScript library at all. However EvenSource does not support binary data. 

Server Support
--------------
The example is implemented using [jetty-eventsource-servlet](https://github.com/jetty-project/jetty-eventsource-servlet) with uses [Jetty Continuations](http://wiki.eclipse.org/Jetty/Feature/Continuations) which are supported by any Jetty version starting with 6 or any Servlet 3 container.

Browser Support
---------------
EvenSource is supported by any modern browser [any browser except IE](http://caniuse.com/#feat=eventsource).

