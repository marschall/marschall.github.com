---
layout: post
title: True Read Only Transactions with Spring 4.3.7
---

Spring 4.3.7 contains [SPR-15210](https://jira.spring.io/browse/SPR-15210) which for the first time provides true read only database transactions.  The feature has to be explicitly enabled by using [setEnforceReadOnly](https://docs.spring.io/spring/docs/4.3.7.BUILD-SNAPSHOT/javadoc-api/org/springframework/jdbc/datasource/DataSourceTransactionManager.html#setEnforceReadOnly-boolean-)

```java
DataSourceTransactionManager txManager = new DataSourceTransactionManager(dataSource);
txManager.setEnforceReadOnly(true);
```

this will result in

```sql
SET TRANSACTION READ ONLY
```

being executed. This should work for Oracle, MySQL and Postgres. With Oracle the feature will give you read consistency including repeatable reads. If your database has a different syntax then you can subclass `DataSourceTransactionManager` and override [prepareTransactionalConnection](https://docs.spring.io/spring/docs/4.3.7.BUILD-SNAPSHOT/javadoc-api/org/springframework/jdbc/datasource/DataSourceTransactionManager.html#prepareTransactionalConnection-java.sql.Connection-org.springframework.transaction.TransactionDefinition-).

Note that earlier versions of Spring would only call [Connection.setReadOnly(true)](https://docs.oracle.com/javase/8/docs/api/java/sql/Connection.html#setReadOnly-boolean-) which gives you a read only connection but not a read only transaction (it did with old Oracle drivers but this was a bug). This is also still the default behavior of `DataSourceTransactionManager`.


