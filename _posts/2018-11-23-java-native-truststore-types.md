---
layout: post
title: Using Native Truststores with Java
---

Besides the truststore shipped with the JDK in `cacerts` Java allows using the native truststore of the operating system.

On macOS the tuststore of type `KeychainStore` is the [macOS Keychain](https://en.wikipedia.org/wiki/Keychain_(software).

On Windows the tuststore of type `Windows-MY` is the truststore of the current user and `Windows-ROOT` is the truststore of the current computer.

Java can be switched to to use a different truststore using `-Djavax.net.ssl.trustStoreType=xxx`.

All available truststore types can be listed using:

```java
Arrays.stream(Security.getProviders())
  .flatMap(p -> p.entrySet().stream())
  .map(e -> (String) e.getKey())
  .filter(e -> e.startsWith("KeyStore."))
  .filter(e -> !e.endsWith("ImplementedIn"))
  .map(e -> e.substring("KeyStore.".length()))
  .sorted()
  .forEach(System.out::println);

```

