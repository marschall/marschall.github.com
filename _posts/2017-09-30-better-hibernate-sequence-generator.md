---
layout: post
title: A Better Hibernate Sequence Generator
published: false
---

Current versions of Hibernate (5.2 at the time of writing) recommend the `pooled` and `pooledlo` sequence generators for overall best performance. They reduce the number of database round trips for batch inserts, offer good concurrency and work well with third party systems. Unfortunately they require setting the `INCREMENT BY` value to the size of the the pool. The value configured in Java and the value configured on sequence iteself need to be the same value.

We offer a sequence generator called that offers the same advantages as `pooled` and `pooledlo` but works with the default `INCREMENT BY` of 1. This means for single inserts no gaps in the generated values are created. This generator works by fetching multiple sequence values in one round trip from the database by using a [recursive common table expression](). For best performance the `CACHE` value on the squence should be set to the same value as the pool but this is not a requirement.
This sequence generator works for the databases , , , and . Unfortunatley MariaDB 10.3 is currently not supported because Hibernate does not yet know that it supports sequences.

Using the sequence generator is a bit verbose as Hibernate does not yet support [meta annotations]()

```java
```


