---
layout: post
title:  Thread Safety of SecureRandom
published: false
---

Before Java 9 the thread safety contract of `SecureRandom` and  `SecureRandomSPI` was not specified (see [JDK-7004967](https://bugs.openjdk.java.net/browse/JDK-7004967)). As of Java 8 the situation was that `SecureRandom` was thread safe by virtue of being `synchronized` internally. This of course limits parallelism.

With Java 9 the situation has now changed (see [JDK-8169871](https://bugs.openjdk.java.net/browse/JDK-8169871) and [JDK-8165115](https://bugs.openjdk.java.net/browse/JDK-8165115)). A [SecureRandomSpi](https://docs.oracle.com/javase/9/docs/api/java/security/SecureRandomSpi.html) can now optionally document that it is thread safe. If it is marked as thread safe then [SecureRandom](https://docs.oracle.com/javase/9/docs/api/java/security/SecureRandom.html) will not synchronize access to it. If the `SecureRandomSPI` is no marked as thread safe then the behavior is the same as in Java 8 and `SecureRandom` will synchronize all access to it.

All thread safe `SecureRandomSPI`s can be found with


```java
Arrays.stream(Security.getProviders())
  .flatMap(p -> p.entrySet().stream())
  .filter(e -> ((String) e.getKey()).startsWith("SecureRandom."))
  .filter(e -> ((String) e.getKey()).endsWith(" ThreadSafe"))
  .filter(e -> e.getValue().equals("true"))
  .map(e -> (String) e.getKey())
  .map(e -> e.substring("SecureRandom.".length(), e.length() - " ThreadSafe".length()))
  .sorted()
  .forEach(System.out::println);
```

With Oracle JDK 9.0.1 on macOS this prints

```
DRBG
NativePRNG
NativePRNGBlocking
NativePRNGNonBlocking
SHA1PRNG
```

Note that even though a `SecureRandomSPI` may be marked as thread safe the implementation may still acquire a lock like the `NativePRNG` variants.

