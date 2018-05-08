---
layout: post
title:  Loading Java Strings into an Oracle Table
---

Sometimes when analyzing Java heap dumps it can be helpful to have all the the strings in a Java application in a relational database so that you have SQL available for analysis.

All strings from a heap dump can be [extracted](https://help.eclipse.org/mars/index.jsp?topic=%2Forg.eclipse.mat.ui.help%2Ftasks%2Fqueryingheapobjects.html) with [Eclipse MAT](https://www.eclipse.org/mat/) using the following [OQL](https://help.eclipse.org/neon/index.jsp?topic=%2Forg.eclipse.mat.ui.help%2Freference%2Foqlsyntax.html) query.

```
SELECT toString(s), s.@retainedHeapSize
FROM java.lang.String s
```

The result can be exported to a CSV called `strings.csv`.

If you are using [string deduplication](http://openjdk.java.net/jeps/192) it may be worthwhile to also extract the address of of the backing character array.

```
SELECT toString(s), s.@retainedHeapSize, s.value.@objectAddress
FROM java.lang.String s
```

We need to create a table where we store the strings. We use the `VARCHAR2` data type instead of `CLOB` so that we can use `GROUP BY` expressions. Unfortunately that means we can not analyse strings that are larger than 4000 bytes in the database encoding. 

```sql
CRETE TABLE dump_string (
  string_value VARCHAR2(4000),
  retained_size NUMBER(9)
);
```

We use [SQL*Loader](https://docs.oracle.com/en/database/oracle/oracle-database/18/sutil/oracle-sql-loader.html) to load the file into the database table. For that a control file has to be created, we name ours `strings.ctl`.

```
LOAD DATA
INFILE 'strings.csv'
INTO TABLE dump_string
APPEND
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
(string_value, retained_size)
```

```
sqlldr SCOTT/TIGER@ORCL control=strings.ctl rows=100000 errors=2000000
```

You should check the error log for the rejected strings.

You can then run queries like the following.

```sql
SELECT s.string_value, sum(s.retained_size), count(1)
FROM dump_string s
GROUP BY s.string_value
ORDER BY sum(s.retained_size) DESC;
```

Once you have identified intersting strings you can find them again in MAT using the following OQL query.

```
SELECT *
FROM java.lang.String s
WHERE toString(s) = "interesting value"
```

