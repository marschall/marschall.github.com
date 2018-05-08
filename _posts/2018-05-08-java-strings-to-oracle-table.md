---
layout: post
title:  Loading Java Strings into an Oracle Table
published: false
---

```
SELECT toString(s), s.@retainedHeapSize
FROM java.lang.String s
```

[string deduplication](http://openjdk.java.net/jeps/192)

```
SELECT toString(s), s.@retainedHeapSize, s.value.@objectAddress
FROM java.lang.String s
```

```sql
CRETE TABLE dump_string (
  string_value VARCHAR2(4000),
  retained_size NUMBER(9)
);
```

```
LOAD DATA
INFILE 'strings.csv'
INTO TABLE dump_string
APPEND
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
(string_value, retained_size)
```

```
sqlldr SCOTT/TIGER@DCBO20 control=filein.ctl rows=100000 errors=2000000
```


