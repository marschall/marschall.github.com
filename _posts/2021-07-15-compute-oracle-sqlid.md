---
layout: post
title: How to Compute the Oracle SQL_ID
---

The Oracle SQL_ID is computed using the following algorithm

1. append a `0x00` byte to the native SQL query
1. compute the MD5 hash (it's unclear whether UTF-8 or the database encoding should be used)
1. create a 64 bit long value out last two 32 integer values of the hash using big endian order
1. convert to Base32, 5 bits at a time starting with the most significant bit, using the alphabet `0123456789abcdfghjkmnpqrstuvwxyz`

The native SQL query can be accessed in Java either using [Connection#nativeSQL](https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/Connection.html#nativeSQL(java.lang.String)) or using [OracleDatabaseException#getSql](https://docs.oracle.com/en/database/oracle/oracle-database/21/jajdb/oracle/jdbc/OracleDatabaseException.html#getSql__).

For a Java implementation check out [marschall/sqlid](https://github.com/marschall/sqlid)

