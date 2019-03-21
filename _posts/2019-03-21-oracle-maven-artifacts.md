---
layout: post
title: Getting Oracle JDBC Drivers through Maven
---

Oracles [JDBC drivers](https://www.oracle.com/technetwork/database/application-development/jdbc/downloads/index.html) are infamous for not being in the [Maven Central Repository](https://maven.apache.org/repository/). What is less known is that there is an [Oracle Maven Repository](https://maven.oracle.com/) or how to use the Oracle Maven Repository. This article describes how to do use the Oracle Maven Repository to get the Oracle JDBC drivers for a developer setup. In a corporate setup you would use a repository manager like Nexus or Artifactory.

The first thing we need is an [Oracle OTN account](https://www.oracle.com/technical-resources/), we will need this to download most Oracle software. We are going to use [Maven Password Encryption](https://maven.apache.org/guides/mini/guide-encryption.html) to store our OTN password. This isn't strictly necessary and the setup is simpler without it but it is good practice.


Next, if not already done, we create a [Maven master password](https://maven.apache.org/guides/mini/guide-encryption.html#How_to_create_a_master_password)

```
mvn --encrypt-master-password <maven-master-password>
```

where `<maven-master-password>` is replaced with our Maven master password. We take the output of that and paste it into `${HOME}/.m2/settings-security.xml`

```xml
<settingsSecurity>
  <master>{jSMOWnoPFgsHVpMvz5VrIt5kRbzGpI8u+9EF1iFQyJQ=}</master>
</settingsSecurity>
```

where `{jSMOWnoPFgsHVpMvz5VrIt5kRbzGpI8u+9EF1iFQyJQ=}` is replaced with the output of creating the Maven password. Then we encrypt our OTN password with

```
mvn --encrypt-password <otn-password>
```

where `<otn-password>` is replaced with our OTN password. We take the output of that and paste it into `${HOME}/.m2/settings.xml`

```xml
<settings>

  <!-- ... -->
  <servers>
    <server>
      <id>maven.oracle.com</id>
      <username>scott@oracle.com</username>
      <password>{COQLCE6DU6GtcS5P=}</password>
      <configuration>
        <basicAuthScope>
          <host>ANY</host>
          <port>ANY</port>
          <realm>OAM 11g</realm>
        </basicAuthScope>
        <httpConfiguration>
          <all>
            <params>
              <property>
                <name>http.protocol.allow-circular-redirects</name>
                <value>%b,true</value>
              </property>
            </params>
          </all>
        </httpConfiguration>
      </configuration>
    </server>
  </servers>

  <profiles>
    <profile>
      <id>download-from-oracle</id>
      <activation>
        <activeByDefault>false</activeByDefault>
      </activation>
      <repositories>
        <repository>
          <id>maven.oracle.com</id>
          <url>https://maven.oracle.com</url>
          <layout>default</layout>
          <releases>
            <enabled>true</enabled>
          </releases>
        </repository>
      </repositories>
    </profile>
  </profiles>

</settings>
```

where `{COQLCE6DU6GtcS5P=}` is the output of encrypting our OTN password and `scott@oracle.com` is replaced with our OTN login.

This sets up a `download-from-oracle` profile that is available in all our local Maven projects. The profile adds the Oracle Maven repository but not by default. This has the following advantages. First we don't have to add a `<repository>` to our projects `pom.xml`. Adding a `<repository>` to our `pom.xml` is not recommended. Second the Oracle Maven repository is only used when we explicitly ask for it:

```
mvn dependency:resolve -Pdownload-from-oracle
```

And with that we can use the Oracle divers from Maven using

```xml
<dependency>
  <groupId>com.oracle.jdbc</groupId>
  <artifactId>ojdbc8</artifactId>
  <version>18.3.0.0</version>
</dependency>
```


Further Reading
---------------

This article is based on the following sources.

* [Maven](https://maven.apache.org/guides/mini/guide-encryption.html)
* [Get Oracle JDBC drivers and UCP from Oracle Maven Repository (without IDEs)](https://blogs.oracle.com/dev2dev/get-oracle-jdbc-drivers-and-ucp-from-oracle-maven-repository-without-ides)
* [Configuring the Oracle Maven Repository](https://docs.oracle.com/middleware/12212/lcm/MAVEN/GUID-19C4FA37-9F37-4F59-8FAF-57FF47698E3A.htm#MAVEN9009)

