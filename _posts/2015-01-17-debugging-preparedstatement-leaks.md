---
layout: post
title: Debugging Oracle PreparedStatement Leaks
---

Introduction
------------

Over the past several weeks I debugged several `PreparedStatement` leaks in an application from a third party vendor. This post summarizes several techniques I found effective in tracking down the piece of code that exhibits those bugs. I will also show several approaches how you can monitor or avoid `PreparedStatement` leaks. 

What is a `PreparedStatement` leak, how does it happen and why is it bad? A `PreparedStatement` leak happens when you forget to call `#close()` on a `PreparedStatement` this prevents

 * the `PreparedStatement` from releasing its associated Java resources (which can be significant)
 * the `PreparedStatement` from being garbge collected
 * the database from releasing any associated resources

The simplest possible `PreparedStatement` leak looks something like this:

{% highlight java %}
PreparedStatement statement = connection.prepareStatement("SELECT X FROM dual");
ResultSet result = statement.executeQuery();
while (result.next()) {
    System.out.println(result.getString("X"));
}
// forget to call close() on result
// forget to call close() on statement
{% endhighlight %}

As leaked `PreparedStatement` are never garbage collected and can have quite large Java resources associated with them (at least with the 11g driver) `PreparedStatement` leaks tend to result in an application crash caused by a `OutOfMemoryError`. Other possibilities are failing connections due to too many open cursors for a database session.

Post Mortem
-----------

The final manifestion of a `PreparedStatement` leak is an application crash caused by a `OutOfMemoryError`. Running your JVM with `-XX:+HeapDumpOnOutOfMemoryError` ensures that if a `OutOfMemoryError` occurs you have a heap dump which can be analyzed. If no heap dump is available fixing a `java.lang.OutOfMemoryError` becomes much more difficult. The first thing I do when analyzing a heap dump is load it into [Eclipse MAT]() and verify it is indeed a `PreparedStatement` leak. This can be done looking at the number of `T4CPreparedStatement` instances in the heap. If there are a couple of thousand then we are looking at a prepared statement leak. Now that we have established that we are indeed dealign with a `PreparedStatement` leak the next step is to extract the SQL used to create these statements. From the SQL it is often easy to search the code base for the place that exhibits the bug. When using the ojdbc driver the SQL of all the `PreparedStatement` instances in the heap can be found with the following OQL query:

{% highlight sql %}
SELECT DISTINCT toString(oracle_sql.value)
FROM oracle.jdbc.driver.OracleSql
{% endhighlight %}

after exporting the result to a CSV and importing it into [SQLite Manager](https://addons.mozilla.org/en-US/firefox/addon/sqlite-manager/) the result can be postprocessed, trailing characters and be removed and the queries can be grouped and counted.

Monitoring
----------

It is important to note that the previously described technique only works once you have a `PreparedStatement` leak in production in the worst case crashing you application. It is best to find `PreparedStatement` leaks as realy as possible, ideally during development but at least before they crash your application.
While developing there are various static anaysis tools that you can run that can detect `PreparedStatement` leaks. [Findbugs](http://findbugs.sourceforge.net/bugDescriptions.html#ODR_OPEN_DATABASE_RESOURCE) is a popular one but many IDEs like [Eclipse](http://help.eclipse.org/juno/index.jsp?topic=%2Forg.eclipse.jdt.doc.user%2Ftasks%2Ftask-avoiding_resource_leaks.htm) also offer build in checks for leaks. It is important to note that none of these tools are perfect and eventually a `PreparedStatement` leak my slip into production unnoticed. Also for third party applications you usually do not have the source code available.

At runtime there are several options how you can monitor your application for `PreparedStatement` leak. Many [APM](http://en.wikipedia.org/wiki/Application_performance_management) products offer built in checks. But there are several other options you can employ.

A simple option is to count the number of `PreparedStatement` instances in the heap. You can do this using `jcmd`.

    jcmd <pid> GC.class_histogram | grep PreparedStatement | wc -l

beware that while this command runs all Java code in the JVM is stopped. An application with more than a thousand `PreparedStatement` objects in the heap likely has a `PreparedStatement` leak.


Oracle also allows you to monitor the open cursors per sessions. If a database session has more than one hundred cursors open you likely have a `PreparedStatement` leak. Your SysDBA knows more.

And finally the Tomcat connection pool (jdbc-pool) allows you to tack and close the open `PreparedStatement` per request. This can be a band aid to keep your application running despite having `PreparedStatement` leaks.

Avoiding PreparedStatement Leaks
--------------------------------

There are several ways how you can design your code so that it never leaks `PreparedStatement`s.

 * You can use Java 7 try-with-resources. Java 7 is the oldest version of Java supported by Oracle, there is no reason to not bet at least using Java 7.
 * You can use an [ORM](http://en.wikipedia.org/wiki/Object-relational_mapping) or a different database access framework like [jOOQ](http://www.jooq.org/), [MyBatis](http://mybatis.github.io/mybatis-3/) or [QueryDSL](http://www.querydsl.com/). These abstract JDBC away and manage JDBC resources for you.
 * You can use a utility class like Spring [JdbcTemplate]() or [Apache Commons DbUtils](http://commons.apache.org/proper/commons-dbutils/). These do not abstract JDBC away but still manage JDBC resources for you. Note that `JdbcTemplate` does not require a Spring application context, so you are not forced to use Spring if you want to use `JdbcTemplate`.

Further Reading
---------------

 * [Oracle JDBC Memory Management](http://www.oracle.com/technetwork/database/application-development/jdbc-memory-management-12c-1964666.pdf)

