---
layout: post
title: Use UTC or TIMESTAMP WITH TIME ZONE
---

If you do business in a time zone that uses [daylight saving time](https://en.wikipedia.org/wiki/Daylight_saving_time) and you want to store dates with a time component in an Oracle database you likely want to store them in [UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time) with the [DATE](http://docs.oracle.com/database/121/SUTIL/GUID-79BAD970-F406-4D29-A78A-A23F63F4D360.htm#SUTIL1167) or [TIMESTAMP](http://docs.oracle.com/database/121/SUTIL/GUID-E2DB6A6E-FBCA-49B1-B8F0-D13A5B8BCB14.htm#SUTIL1170) datatypes or use the [TIMESTAMP WITH TIME ZONE](http://docs.oracle.com/database/121/SUTIL/GUID-DBCD234A-8355-4130-A5B3-B8231F0088AF.htm#SUTIL1171) datatype. You likely do not want to use the `DATE` or `TIMESTAMP` datatypes with a time zone other than UTC. You may think that you do not have to deal with time zones because you operate only in one time zone and therefore do not need to store the time zone. But in reality you are operating in two time zones: summer time and winter time. If you want to get the same values back from the database that you stored you have to store the time zone or accept silent data truncation.

Oracle stores the [DATE](http://docs.oracle.com/database/121/JAJDB/oracle/sql/DATE.html) and [TIMESTAMP](http://docs.oracle.com/database/121/JAJDB/oracle/sql/TIMESTAMP.html) in local time without any time zone information.

For example [European Summer Time](https://en.wikipedia.org/wiki/Summer_Time_in_Europe) ends at three o’clock on 25 October 2015. At three o’clock in the morning the clock jumps back to two o’clock. This meens the hour between two and three o’clock happens twice and you can not distinguish them without knowing the time zone.

As a concrete example these two timestamps

    2015-10-25T02:45:00+02:00
    2015-10-25T02:45:00+01:00

both get stored as `DATE '2015-10-25 02:45:00'` in the database. This means there is a whole hour (or how much your clock jumps) every year which you can not store in the database. Values that you do store silently get an hour subtracted.

Storing values in UTC avoids these issues because UTC has no daylight saving time. However using UTC is not without its downsides. You have to remember to convert to UTC when storing values and convert again when reading values. Also when viewing values directly with a tool like sqlplus you have to remember that the values displayed are in UTC and not local time (unless you live in [Iceland](https://en.wikipedia.org/wiki/Iceland)).

Only since version [4.2](https://docs.oracle.com/javase/8/docs/technotes/guides/jdbc/jdbc_42.html) does JDBC support reading and writing time zones (by use of the [ZonedDateTime](https://docs.oracle.com/javase/8/docs/api/java/time/ZonedDateTime.html) and [OffsetDateTime](https://docs.oracle.com/javase/8/docs/api/java/time/OffsetDateTime.html) data types). Unfortunately the Oracle JDBC driver does not yet implement JDBC 4.2 (as of [Oracle 12.1c](http://docs.oracle.com/database/121/JJDBC/release_changes.htm#JJDBC29140)). You are therefore forced to use [TIMESTAMPTZ](http://docs.oracle.com/database/121/JAJDB/oracle/sql/TIMESTAMPTZ.html) if you want to read and write dates with time zones in Java.

