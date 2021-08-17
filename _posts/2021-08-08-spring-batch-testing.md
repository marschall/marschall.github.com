---
layout: post
title: Integration Testing Spring Batch Applications without Committing
published: false
---

The support of Spring Batch for integration testing may be disappointing compared to other Spring projects like Web MVC. In this post we explore why this is and what can be done about it.

An integration test in Spring Batch creates Spring application context to launch a step or job. Spring Batch has support for this in the form of the `@SpringBatchTest` annotation and the `JobLauncherTestUtils` and `JobRepositoryTestUtils` utility classes. In addition there are the map based DAO implementations (`MapJobInstanceDao`, `MapJobExecutionDao`, `MapStepExecutionDao` and `MapExecutionContextDao`) designed to help writing tests that do not write to the database. However these have been [deprecated](https://github.com/spring-projects/spring-batch/issues/3780) with the intent to remove them in Spring Batch 5. The recommendation is to use an embedded in-memory database instead.

In the [Spring TestContext Framework](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html) test in general [roll back](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-tx) database transactions. This is often preferrable as no side effects of test execution remain on the database and the test developer does not have to write clean up code. However due to the design of `AbstractBatchConfiguration` this is hard to achive in Spring Batch.

- `AbstractJobRepositoryFactoryBean` adds a transactional proxy with isolation level `PROPAGATION_REQUIRES_NEW` around every `JobRepository`. This happens for both `JobRepositoryFactoryBean` as well as the deprecated `MapJobRepositoryFactoryBean`. `JobRepositoryFactoryBean` and `MapJobRepositoryFactoryBean` are used by `DefaultBatchConfigurer`
- `AbstractBatchConfiguration` requires a single `DataSource`. This makes it hard to impossible to use a different `DataSource` for the application code and the Spring Batch JDBC repositories.
- `AbstractBatchConfiguration` requires a single `PlatformTransactionManager`. This makes it hard to impossible to use a different `PlatformTransactionManager` (eg. `ResourceLessTransactionManager`) for the transaction proxy for `JobRepository`. Therefore Spring Test Context framework can no longer roll back transactions as they have already been committed by the proxy around the `JobRepository`.

This means using a `ResourceLessTransactionManager` just for the map based DAO implementations and a `DataSourceTransactionManager` for the application code will not work. It also means that using an in-memory database for the Spring Batch JDBC DAOs will force the application code to also run on this in-memory database. And even then a rollback of the database transaction will still not be possible because they have to share the `PlatformTransactionManager` in addition to the `DataSource`. These issues have been well documented [spring-boot#7259](https://github.com/spring-projects/spring-boot/issues/7259), [spring-batch#816](https://github.com/spring-projects/spring-batch/issues/816), [spring#batch](https://github.com/spring-projects/spring-batch/issues/3942).

The map based DAO implementations are really just usable in cases where the application code performs no JDBC data access. In these cases they can be used together with a `ResourceLessTransactionManager`. Note that even in these cases a `DataSource` bean still has to be present as `JobRepositoryTestUtils` requires one, see [#3767](https://github.com/spring-projects/spring-batch/issues/3767).

Spring Batch offers the methods `JobRepositoryTestUtils#removeJobExecutions()`, `JobLauncherTestUtils#getUniqueJobParameters()` and `JobLauncherTestUtils.getUniqueJobParametersBuilder()` but these are really just work arounds to be able to deal with the side effects of committed transactions instead of fixes for the root issue.

Mitigating these issues requires replacing `AbstractBatchConfiguration` or `DefaultBatchConfigurer` depending the solution you're looking for.

To mitigate all these issues we have created the [Spring Batch In-Memory](https://github.com/marschall/spring-batch-inmemory) project. It comes with:

- In-memory implementations of `JobRepository` and `JobExplorer`.
- Null implementations of `JobRepository` and `JobExplorer` that have not attached storage.
- A replactement for `AbstractBatchConfiguration` that sets up a `StepBuilderFactory` with a `ResourcelessTransactionManager`.
- A null implementation of `DataSource` that allows creating a `JobRepositoryTestUtils`.

To use the project you need the following dependency

```xml
<dependency>
  <groupId>com.github.marschall</groupId>
  <artifactId>spring-batch-inmemory</artifactId>
  <version>0.1.0</version>
</dependency>
```

You can then write a test that rolls back DML operations like this

```java
@Transactional // only needed of you have JDBC DML operations that you want to rollback
@Rollback // only needed of you have JDBC DML operations that you want to rollback
@SpringBatchTest
class MySpringBatchIntegrationTests {

  @Configuration
  @Import({
    MyDataSourceConfiguration.class, // where your DataSource is defined
    MyJobConfiguration.class, // the configuration class of the Spring Batch job or step you want to test
    InMemoryBatchConfiguration.class
  })
  static class ContextConfiguration {

    @Autowired
    private DataSource dataSource;

    // used to roll back the inserts
    @Bean
    public PlatformTransactionManager txManager() {
      return new DataSourceTransactionManager(this.dataSource);
    }

    @Bean
    public JdbcOperations jdbcOperations() {
      return new JdbcTemplate(this.dataSource);
    }

  }

}
```


