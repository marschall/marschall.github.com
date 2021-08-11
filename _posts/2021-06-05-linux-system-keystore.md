---
layout: post
title: Using the Linux System Truststore in Java
---

Java offers the `KeychainStore`, `Windows-MY` and `Windows-ROOT` keystore types to use the system trustores on macOS and Windows. Unfortunately there is no equivalent for Linux. On Linux the root certificates are often stored in [PEM format](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail) in folders like `/etc/ssl/certs` or `/etc/pki/tls/certs`. For such cases the [directory-keystore](https://github.com/marschall/directory-keystore) library allows you to use a folder with certificates as a keystore in Java.

To configure Java to use the system truststore on Linux use the following JVM options

```
-cp directory-keystore-1.1.0.jar
-Djava.security.properties=$(pwd)/conf/security/additional.java.security \
-Djavax.net.ssl.trustStore=$(pwd)/conf/security/etcsslcerts \
-Djavax.net.ssl.trustStoreType=directory
```

The library needs to be added either to the classpath using `-cp` or the modulepath using `-p`.

The `additional.java.security` installs the `directory` security provider. The value `13` is the positon of the security provider. It depends on the Java version you use an which other security provider you have installed. `13` works for OpenJDK 11 on Linux. The location and name of the file is application dependent, we chose `conf/security/additional.java.security` here.

```
security.provider.13=directory
```

The `etcsslcerts` is a redirect file that contains the location of the folder containing the certificates. This indirection is necessary due to the way Java loads keystores. The location and name of the file is application dependent, we chose `conf/security/etcsslcerts` here.

```
/etc/ssl/certs
```

For a complete running example check out [directory-keystore-demo](https://github.com/marschall/directory-keystore-demo)

