---
layout: post
title: PageSpeed Insights Improvements
---

Various improvements to this page have been made in order to get a better score in [Google PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/?url=http%3A%2F%2Fmarschall.github.io%2F&tab=mobile). This page now scores 96 / 100 for mobile speed and 97 / 100 for desktop speed. We do not get 100 / 100 because you can not set HTTP cache headers in [GitHub Pages](https://pages.github.com/).

Most notably all render-blocking JavaScript and CSS in above-the-fold content has been eliminated. An initial page rendering can now be done after the first server exchange (unless you use the old [marschall.github.com](https://marschall.github.com) domain which gets a redirect). This has been done by loading the [Twitter Bootstrap CSS](http://getbootstrap.com/css/) dynamically after loading the HTML. In order for the page to look OK without CSS applied the navbar gets hidden until the CSS is loaded. Together these changes should result in much faster page loading on 3G and slower mobile networks.

<ins>Update</ins> switching to a [custom](http://getbootstrap.com/customize/) Twitter Bootstrap CSS makes the CSS small enough to be inlined.

