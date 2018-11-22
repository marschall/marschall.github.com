---
layout: post
title:  Using JDBC Inlists with Arrays
---

JDBC does not directly support inlists meaning when you have a query like this

```sql
SELECT val
FROM inlist_test_table 
WHERE id IN (?)
```

you can pass only one value to the `PreparedStatement`. There is no option to pass multiple values neither as `Collection` nor as Java array. Meaning if you want to pass two values you have to rewrite the query to

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

Also HSQLDB requires a different syntax and there are some constraints on datatypes. See [this stackoverflow discussion](https://stackoverflow.com/questions/50665451/hsqldb-any-array-function-not-working/50684110) for more information.


```sql
SELECT val
FROM inlist_test_table 
WHERE id IN(UNNEST(?))
```

There is an additional caveat with Oracle in that Oracle does not support anonymous arrays, instead custom array types have to be created. Additionally the Oracle JDBC driver only supports creating JDBC arrays using proprietary APIs. If you are using Spring `JdbcTemplate` then the [SqlOracleArrayValue](https://static.javadoc.io/com.github.ferstl/spring-jdbc-oracle/2.0.0/com/github/ferstl/spring/jdbc/oracle/SqlOracleArrayValue.html) class from [ferstl/spring-jdbc-oracle](https://github.com/ferstl/spring-jdbc-oracle/) does the binding for you and can be used as a bind parameter.


```java
jdbcOperations.queryForObject(SQL,
  new SqlOracleArrayValue("MYARRAYTYPE", array),
  Integer.class);
```

Further Reading
---------------

* [SQL IN Predicate: With IN List or With Array? Which is Faster?](https://blog.jooq.org/2017/03/30/sql-in-predicate-with-in-list-or-with-array-which-is-faster/)
* [When Using Bind Variables is not Enough: Dynamic IN Lists](https://blog.jooq.org/2018/04/13/when-using-bind-variables-is-not-enough-dynamic-in-lists/)
* [IN-list Padding](https://www.jooq.org/doc/latest/manual/sql-building/dsl-context/custom-settings/settings-in-list-padding/)

