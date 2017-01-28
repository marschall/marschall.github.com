---
layout: post
title: Oracle Read Only Transactions
---

Oracle supports [read only transactions](https://docs.oracle.com/database/121/SQLRF/statements_10005.htm#SQLRF55418). They give you read consistency including [repeatable reads](https://en.wikipedia.org/wiki/Isolation_(database_systems)#Repeatable_reads). This means you only see changes that were committed before the read only transaction was started. You do not see any changes that are committed by other transactions while the read only transaction is running. In addition the allowed only queries are `SELECT` statements without the `FOR UPDATE` clause. Read only transactions can be a very useful feature as they give you a consistent view of the database across several queries.

Unfortunately read only Oracle transactions are hard to use using JDBC. Calling [Connection.setReadOnly(true)](https://docs.oracle.com/javase/8/docs/api/java/sql/Connection.html#setReadOnly-boolean-) with the 12c driver no longer establishes a read only transaction as it did the with 9i and 10g drivers (I am unsure about the behavior of the 11g drivers).

For Spring users this means that [read only Spring transactions](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html#readOnly--) no longer give you read only Oracle transactions. This implies that read only Spring transactions no longer give you read consistency with Oracle. Previously when using read only Spring transactions with the [DataSourceTransactionManager](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jdbc/datasource/DataSourceTransactionManager.html) you would get read consistency with an Oracle database. This feature never worked with the [JtaTransactionManager](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/transaction/jta/JtaTransactionManager.html) typically used in Java EE environments.

If you want a read only Oracle transaction you have to call

```sql
SET TRANSACTION READ ONLY;
```

If you want to avoid this but still want read consistency and repeatable reads with Oracle you have to use the [serializable](https://en.wikipedia.org/wiki/Isolation_(database_systems)#Serializable) isolation level (which theoretically is only snapshot isolation).

