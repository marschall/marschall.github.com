---
layout: post
title:  Using JDBC Inlists with Arrays
published: false
---

JDBC does not directly support inlists meaning when you have a query like this

```sql
SELECT val
FROM inlist_test_table 
WHERE id IN (?)
```

you can pass only one value to the `PreparedStatement`. There is no option to pass multiple values as either a `Collection` or a Java array. Meaning if you want to pass two values you have to rewrite the query to

```sql
SELECT val
FROM inlist_test_table 
WHERE id IN (?, ?)
```
and so forth. This is very inconvenient. It also not very efficient for server side statement caches. This can result in additional parsing overhead on the database server side. Some people use [NamedParameterJdbcTemplate](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jdbc/core/namedparam/NamedParameterJdbcTemplate.html) from Spring JDBC. However `NamedParameterJdbcTemplate` requires parsing and rewriting the query every time the query is executed with a different number of elements in the inlist.

There is however an easy solution using SQL arrays. The query can be rewritten to

```sql
SELECT val
FROM inlist_test_table 
WHERE id = ANY(?)
```

and then a `java.sql.Array` can be passed to the `PreparedStatement`. This works with the following databases

* H2
* HSQLDB
* PostgreS
* Oracle

With Oracle 12c and earlier versions a slightly different syntax has to be used


```sql
SELECT val
FROM inlist_test_table 
WHERE id = ANY(SELECT column_value FROM TABLE(?))
```

Also HSQLDB requires a different syntax also there are some constraints on datatypes see [this stackoverflow discussion](https://stackoverflow.com/questions/50665451/hsqldb-any-array-function-not-working/50684110).


```sql
SELECT val
FROM inlist_test_table 
WHERE id IN ( UNNEST(?) )
```

There is an additional caveat with Oracle in that Oracle does not support anonymous arrays and instead custom array types have to be created.

* [SQL IN Predicate: With IN List or With Array? Which is Faster?](https://blog.jooq.org/2017/03/30/sql-in-predicate-with-in-list-or-with-array-which-is-faster/)
* [When Using Bind Variables is not Enough: Dynamic IN Lists](https://blog.jooq.org/2018/04/13/when-using-bind-variables-is-not-enough-dynamic-in-lists/)
* [IN-list Padding](https://www.jooq.org/doc/latest/manual/sql-building/dsl-context/custom-settings/settings-in-list-padding/)
