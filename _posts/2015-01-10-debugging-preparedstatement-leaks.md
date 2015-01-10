---
layout: post
title: Debugging Oracle PreparedStatement Leaks
published: false
---

Introduction
------------

Over the past several weeks I debugged several `PreparedStatement` leaks in an application from a third party vendor. This post summarizes several techniques I found effective in tracking down the piece of code that exhibits those bugs. I will also show several approaches how you can avoid `PreparedStatement` leaks. 

What is a `PreparedStatement` leak, how does it happen and why is it bad? A `PreparedStatement` leak happens when you forget to call `#close()` on a `PreparedStatement` this prevents
 * the `PreparedStatement` from releasing its associated Java resources (which can be significant)
 * the `PreparedStatement` from being garbge collected
 * the database to release any associated resources

The simplest possible `PreparedStatement` leak looks something like this:

```java
PreparedStatement statement = connection.prepareStatement("SELECT X FROM dual");
ResultSet result = statement.executeQuery();
while (result.next()) {
    System.out.println(result.getString("X"));
}
// forget to call close() on result
// forget to call close() on statement
```

As leaked `PreparedStatement` are never garbage collected and can have quite large Java resources associated with them (at least with the 11g driver) `PreparedStatement` leaks tend to cause an application crash caused by a `java.lang.OutOfMemoryError`. Other possibilities are failing connections due to too many open cursors for a database session.

Post Mortem
-----------

The final manifestion of a `PreparedStatement` leak is an application crash caused by a `java.lang.OutOfMemoryError`. Running all you JVMs with `-XX:+HeapDumpOnOutOfMemoryError` ensures that if a `java.lang.OutOfMemoryError` occurs you have a heap dump which can be analyzed. If no heap dump is available fixing a `java.lang.OutOfMemoryError` becomes much more difficult. The first thing I do when analyzing a heap dump is load it into [Eclipse MAT]() and verify it is indeed a `PreparedStatement` leak. This can be done looking at the number of XXX instances in the heap. If it is a couple of thousand then we are looking at a prepared statement leak. Now that we have established that we are indeed dealign with a `PreparedStatement` leak the next step is to extract the SQL used to create these statements. From the SQL it is often easy to search the code base for the place that exhibits the bug. When using the ojdbc driver the SQL of all the `PreparedStatement`s in the heap can be found with the following OQL query:


```sql
SELECT DISTINCT toString(oracle_sql.value)
FROM oracle.jdbc.driver.OracleSql
```

after exporting the result to a CSV and importing it into [SQLite Manager](https://addons.mozilla.org/en-US/firefox/addon/sqlite-manager/) the result can be post processed, trailing characters and be removed and the queries can be grouped and counted.

Monitoring
----------

While developing

At runtime


    cmd <pid> GC.class_histogram | grep PreparedStatement | wc -l



Tomcat Connection Pool
Oracle open cursors per session

Avoiding PreparedStatement Leaks
--------------------------------

There are several ways how you can design you code so that it never leaks `PreparedStatement`s.

 * You can use Java 7 try-with-resources. Java 7 is the oldest version of Java supported by Oracle, there is no reason to not at least use Java 7.
 * You can use an [ORM]() or a different database access framework like [jOOQ](), [MyBatis]() or [QueryDSL](). These abstract JDBC away and manage JDBC resources for you.
 * You can use a utility class like Spring [JdbcTemplate]() or [Apache commons Database](). These do not abstract JDBC away but still manage JDBC resources for you. Note that `JdbcTemplate` does not require a Spring application context, so you are not forced to use Spring if you want to use `JdbcTemplate`.

Further Reading
---------------

 * Oracle 12c JDBC driver managment

