---
layout: post
title: How to Create Passwordless PKCS 12 Truststores with Java
---

The default truststore format for current versions of Java is [PKCS #12](https://en.wikipedia.org/wiki/PKCS_12). As PKCS #12 can also be used for keystores it often has a password.

However this makes little sense for truststores as unlike keystores they do not contain confidential data. In addition it complicates operations as a password has to be managed.

Since [JDK 18](https://bugs.openjdk.org/browse/JDK-8274862) Java supports creating PKCS #12 truststores without a password through the [KeyStore API](https://docs.oracle.com/en/java/javase/18/docs/specs/security/standard-names.html).

```
KeyStore.store(OutputStream, null);
```

[Earlier versions](https://bugs.openjdk.org/browse/JDK-8076190) need the following two system properties in order for this to work

```
-Dkeystore.pkcs12.certProtectionAlgorithm=NONE -Dkeystore.pkcs12.macAlgorithm=NONE
```

Unfortunately this is a JVM wide setting.

[marschall/truststore-maven-plugin](https://github.com/marschall/truststore-maven-plugin) supports generating passwordless PKCS 12 truststores since 0.7.0 either on JDK 18 by setting the above mentioned system properties.

