---
layout: post
title: Disabling the Gradle Build Cache
---

The [Gradle build cache](https://docs.gradle.org/current/userguide/build_cache.html) may be a great thing when you're regularly building large projects with Gradle. However when only occasionally building open source projects it can quickly become large.

To [disable](https://docs.gradle.org/current/userguide/build_cache.html) the Gradle build cache add the following line to `~/.gradle/gradle.properties`

```
org.gradle.caching=false
```

You can clean the existing cache with

```
rm -rf $HOME/.gradle/caches/
rm -rf $HOME/.gradle/wrapper/
```

