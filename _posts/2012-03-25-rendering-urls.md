---
layout: post
title: Rendering URLs Considered Hard
published: false
---

{{ page.title }}
================

One of the ways I judge the maturity of web frameworks is how are they rendering URLs. Are they just slamming strings together or are they doing «The Right Thing»™.

Correctly rendering an URL takes several steps. Cutting corners is likely to cause trouble with non-ASCII content.

Break the URL Into Subcomponents
--------------------------------
First you have to break the URL into subcomponents. If you treat an URL as an opaque string you don't have enough contextual information to render the individual parts correctly, we learned this the hard way. A & in a path element has to be treated differently than the & separating parameters. So if you start from a string, you first have to parse it into a high level URL object.

Translate to Octets
-------------------
That's the step that most often gets ignored or done wrong. You cannot directly encode a character only bytes aka octets. This means you'll first have to find the right encoding. So which one is it. <a href="http://www.ietf.org/rfc/rfc1738.txt">RFC 1738</a> assumes you'll only use US-ASCII, <a href="http://www.rfc-editor.org/rfc/rfc2396.txt">RFC 2396</a> says you're free to use whatever you want which probably often was either ISO-8859-1 or CP1251 (which are not the same BTW) and <a href="http://www.rfc-editor.org/rfc/rfc3986.txt">RFC 3986</a> says UTF-8. So what should you take? Whatever you server uses to decode the GET parameters. This is often the same as the page encoding but not always.

In addition for the domain name you have to use <a href="http://en.wikipedia.org/wiki/Punycode">Punycode</a>. This includes so much Unicode that if you're not on Java or .NET you have to use <a href="http://www.icu-project.org/">ICU</a>.

Percent Encode
--------------
Most people know about this step, everything that's not "safe" has to be turned into <code>%hex-value</code>. Either here in the next step you have to turn the octests you got after the previous step back into characters.

HTML Escape
-----------
URLs like every other page content has to be properly HTML escaped. This escpecially means turning <code>&amp;</code> into <code>&amp;amp;</code> forgetting this is one of the main causes for invalid html.

Response Encode
---------------
And finally like every other page content the URL has to be encoded using the page encoding. This can be the same as an step two but doesn't have to be.

Further Information
-------------------
 * [useBodyEncodingForURI](http://tomcat.apache.org/tomcat-7.0-doc/config/ajp.html) in the Tomcat configuration 
 * [Tomcat Character Encoding FAQ](http://wiki.apache.org/tomcat/FAQ/CharacterEncoding)
 * [java.net.URLEncode#encode(String)](http://docs.oracle.com/javase/7/docs/api/java/net/URLEncoder.html#encode%28java.lang.String%29)
 * [UTF-8 Sampler](http://www.columbia.edu/~fdc/utf8/)


