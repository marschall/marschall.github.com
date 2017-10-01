---
layout: post
title: A Better Hibernate Sequence Generator
---

Current versions of Hibernate (5.2 at the time of writing) recommend the `pooled` and `pooledlo` sequence generators for overall best performance. They reduce the number of database round trips for batch inserts, offer good concurrency and work well with third party systems. Unfortunately they require setting the `INCREMENT BY` value to the size of the identifier pool. Care has to be taken that these values don't deviate.

We offer a sequence generator called [hibernate-batch-sequence-generator](https://github.com/marschall/hibernate-batch-sequence-generator) that offers the same advantages as `pooled` and `pooledlo` but works with the default `INCREMENT BY` of 1. This means single inserts that use the sequence directly cause not gaps in the generated values. This generator works by fetching multiple sequence values in one database round trip by using a [recursive common table expression](https://en.wikipedia.org/wiki/Hierarchical_and_recursive_queries_in_SQL). For best performance the `CACHE` value on the sequence should be set to the same value as the identifier pool size but performance is still good even when this is not done.
This sequence generator works for the databases Firebird, H2, HSQLDB, Oracle, Postgres and SQL Sever. Unfortunately MariaDB 10.3 is currently not supported because Hibernate does [not yet](https://github.com/hibernate/hibernate-orm/pull/1930) know that it supports sequences.

Using the sequence generator is a bit verbose as Hibernate does not support [meta annotations](https://github.com/spring-projects/spring-framework/wiki/Spring-Annotation-Programming-Model)

```java
@Id
@GenericGenerator(
        name = "some_column_name_id_generator",
        strategy = "com.github.marschall.hibernate.batchsequencegenerator.BatchSequenceGenerator",
        parameters = {
            @Parameter(name = "sequence", value = "SOME_SEQUENCE_NAME"),
            @Parameter(name = "fetch_size", value = "SOME_FETCH_SIZE_VALUE")
        })
@GeneratedValue(generator = "some_column_name_id_generator")
@Column(name = "SOME_COLUMN_NAME")
private Long someColumnName;

```


