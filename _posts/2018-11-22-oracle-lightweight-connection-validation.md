---
layout: post
title:  Lightweight Connection Validation with the Oracle 18.3 Driver and WildFly
---

The Oracle 18.3 driver has support for [Support for Lightweight Connection Validation](https://docs.oracle.com/en/database/oracle/oracle-database/18/jjdbc/JDBC-getting-started.html#GUID-6AFB71F0-EFA9-43A0-AF96-03E6FB2F3521). The relevant documentation says

> Starting from Oracle Database Release 18c, JDBC Thin driver supports lightweight connection validation. Lightweight connection validation enables JDBC applications to verify connection validity by sending a zero length NS data packet that does not require a round-trip to the database. For the earlier releases of Oracle Database, when you call the isValid(timeout) method to test the validity of a connection, Oracle JDBC driver uses a ping-pong protocol, which is an expensive operation as it makes a full round-trip to the database. In Oracle Database Release 18c, the isValid(timeout) method instead sends an empty packet to the database and does not wait to receive it back. So, connection validation is faster, which results in better application performance.

> Lightweight connection validation is disabled by default. To enable this feature, you must set the oracle.jdbc.defaultConnectionValidation connection property value to SOCKET. If this property is set, then the JDBC driver performs lightweight connection validation, when you call the isValid(timeout) method.

To use this from WildFly you have to configure the datasoruces subsystem accordingly. The important part here is to switch from to `OracleValidConnectionChecker` to `JDBC4ValidConnectionChecker`

```xml
<subsystem xmlns="urn:jboss:domain:datasources:5.0">
  <datasources>
     <datasource ...>
       <!-- ... -->
       <connection-property name="oracle.jdbc.defaultConnectionValidation">SOCKET</connection-property>
       <validation>
         <valid-connection-checker class-name="org.jboss.jca.adapters.jdbc.extensions.novendor.JDBC4ValidConnectionChecker"/>
         <!-- ... -->
       </validation>
    </datasource>
</subsystem>


