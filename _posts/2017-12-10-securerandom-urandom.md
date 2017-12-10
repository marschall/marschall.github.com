---
layout: post
title: Using /dev/urandom with SecureRandom
---

If you want to use a `SecureRandom` in Java that uses `/dev/urandom` you can the an instance with

or simply

```java
SecureRandom.getInstance("NativePRNGNonBlocking");
```

This will use `/dev/urandom` for both random number generation and seeds. `"NativePRNGBlocking"` will use `/dev/random` for both random number generation and seeds. `"NativePRNG"` will use `/dev/urandom` for random number generation and the EDG file or `/dev/random` for seeds if the EGD file is not available and readable. `"NativePRNG"` is the default unless the EDG file is not set or set to something other than `file:/dev/random` (the default) or `file:/dev/urandom` then `"SHA1PRNG"` will be the default.

The EDG file is the value of he `java.security.egd` system property or the value of the `securerandom.source` if the former is not available. The latter can be configured in the `java.security` file.

For details see `sun.security.provider.NativePRNG` and `sun.security.provider.SunEntries` and [SecureRandom Implementations](https://docs.oracle.com/javase/9/security/oracleproviders.htm#JSSEC-GUID-9DC4ADD5-6D01-4B2E-9E85-B88E3BEE7453).

