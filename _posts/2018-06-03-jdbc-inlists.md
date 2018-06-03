---
layout: post
title:  Using JDBC Inlists with Arrays
published: false
---

JDBC does not directly support inlists meaning when you have a query like this

```sql
SELECT val
FROM inlist_test_table 
WHERE id IN(?)
```

you can pass only one value to the `PreparedStatement` there is no option to pass multiple values with either a `Collection` or a Java array. Meaning if you want to pass two values you have to rewrite the query to

```sql
SELECT val
FROM inlist_test_table 
WHERE id IN(?, ?)
```
and so forth. This is very inconvenient. Some people use `NamedParameterJdbcTemplate` from Spring JDBC. However requires parsing and rewriting the query every time the query is executed with a different number of elements in the inlist. In addition it makes the cursor cache on the database server less effective and also makes an potential statement caches on the client side less effective. Both of these together can result in additional parsing overhead on the database server side.

However there is an easy solution using SQL arrays. The query can be rewritten to

```sql
SELECT val
FROM inlist_test_table 
WHERE id = ANY(?)
```

and then a `java.sql.Array` can be passed to the `PreparedStatement`. This works with the following databases

* H2
* PostgreS
* Oracle 18c

With earlier versions of Oracle a slightly different syntax has to be used


```sql
SELECT val
FROM inlist_test_table 
WHERE id = ANY(SELECT column_value FROM TABLE(?))
```

There is an additional caveat with Oracle in that Oracle does not support anonymous arrays and instead custom array types have to be created.
