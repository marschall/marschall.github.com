---
layout: post
title: Integration Testing Spring Batch Applications without Committing
published: false
---

The support of Spring Batch for integration testing may be disappointing compared to other Spring projects like Web MVC. In this post we explore why this is and what can be done about it.

An integration test in Spring Batch creates Spring application context to launch a step or job. Spring Batch has support for this in the form of the `@SpringBatchTest` annotation and the `JobLauncherTestUtils` and `JobRepositoryTestUtils` utility classes. In addition there are the map based DAO implementations (`MapJobInstanceDao`, `MapJobExecutionDao`, `MapStepExecutionDao` and `MapExecutionContextDao`) designed to help creating tests that do not write to the database. However these have been deprecated with the intent to remove them in Spring Batch 5. The recommendation is to use an embedded in-memory database instead.

In the Spring Test Context Framework test in general roll back database transactions. This is often preferrable as no side effects of test execution remain on the database and the test developer does not have to write clean up code. However due to the design of `AbstractBatchConfiguration` this is hard to achive in Spring Batch.

- `AbstractBatchConfiguration` requires a single `DataSource`. This makes it hard to impossible to use a different `DataSource` for the application code and the Spring Batch JDBC repositories.
- `AbstractBatchConfiguration` requires a single `PlatformTransactionManager`. This `PlatformTransactionManager` is used to commit transactions. Therefore Spring Test Context framework can no longer roll back transactions as they have already been committed. It is also not possible to use a `ResourceLessTransactionManager` for just the `StepBuilderFactory`.

This means using a `ResourceLessTransactionManager` just for the map based DAO implementations and a `DataSourceTransactionManager` for the application code will not work. It also means that using an in-memory database for the Spring Batch JDBC DAOs will force the application code to also run on this in-memory database. And even then a rollback of the database transaction will still not be possible.

The map based DAO implementations are really just usable in cases where the application code preforms no JDBC data access. In these cases they can be used together with a `ResourceLessTransactionManager`. Note that even in these cases a `DataSource` bean still has to be present as `JobRepositoryTestUtils` requires one.



- Completely replace `AbstractBatchConfiguration` with a different Configuration that sets up `StepBuilderFactory`
- `SimpleJobRepository` and `SimpleJobExplorer` with a `ResourcelessTransactionManager` 
- uses a different `DataSource` and `DataSourceTransactionManager`

`JobRepositoryTestUtils#removeJobExecutions()`


`JobRepositoryTestUtils` requires a `DataSource` bean to be present.

Oracle commits on `Connection#close()`

TODO ResourceLessTransactionManager for everything? No rollback (lingering transactions)? Oracle commits on `Connection#close()`

TODO InMemoryBatchConfigurerTests -> does StepBuilderFactory get DataSourceTransactionManager?

TODO MapJobRegistry

