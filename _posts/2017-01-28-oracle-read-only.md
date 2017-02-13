---
layout: post
title: Oracle Read Only Transactions
---

The Oracle database supports [read only transactions](https://docs.oracle.com/database/121/SQLRF/statements_10005.htm#SQLRF55418). They give you read consistency including <a href="https://en.wikipedia.org/wiki/Isolation_(database_systems)#Repeatable_reads">repeatable reads</a>. This means you only see changes that were committed before the read only transaction was started. You do not see any changes that are committed by other transactions while the read only transaction is running. In addition the only allowed queries are `SELECT` statements without the `FOR UPDATE` clause. Read only transactions can be a very useful feature as they give you a consistent view of the database across several queries.

## JDBC

Unfortunately read only Oracle database transactions are hard to use with JDBC. Calling [Connection.setReadOnly(true)](https://docs.oracle.com/javase/8/docs/api/java/sql/Connection.html#setReadOnly-boolean-) with the 12c driver no longer establishes a read only transaction as it did the with 9i and 10g drivers (I am unsure about the behavior of the 11g drivers). This is an intentional change as establishing a read only transaction is not the purpose of the method.

## Spring

Spring supports the concept of [read only Spring transactions](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/TransactionDefinition.html#isReadOnly--), typically used with [@Transactional(readOnly=true)](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html#readOnly--). When read only transactions Spring are used together with the  [DataSourceTransactionManager](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jdbc/datasource/DataSourceTransactionManager.html) or the [HibernateTransactionManager](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/orm/hibernate5/HibernateTransactionManager.html) a call to `Connection.setReadOnly(true)` is made. This doesn't happen with the [JtaTransactionManager](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/transaction/jta/JtaTransactionManager.html) typically used in Java EE environments. But as we said before because `Connection.setReadOnly(true)` does not give you a read only transaction, read only Spring transactions do not give you read only Oracle database transactions.

## Summary

If you want a read only Oracle database transaction you have to call

```sql
SET TRANSACTION READ ONLY;
```

If you want to avoid this but still want read consistency and repeatable reads with Oracle you have to use the <a href="https://en.wikipedia.org/wiki/Isolation_(database_systems)#Serializable">serializable</a> isolation level (which theoretically is only snapshot isolation).

### Update
This has changed in [Spring 4.3.7](2017-02-13-spring-read-only.md).

