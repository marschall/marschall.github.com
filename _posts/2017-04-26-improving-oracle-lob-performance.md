---
layout: post
title: Improving Oracle LOB Performance
---

If you're using LOBs with Oracle and want to trade memory footprint for performance there are two options you have.

On the database side if you're frequently reading and writing LOBs you can [enable LOB caching](https://docs.oracle.com/database/122/ADLOB/LOB-storage-with-applications.htm#ADLOB45283) this causes them to be placed in the buffer cache like any other row data. This can improve performance by reading LOBs directly from memory rather than storage.

On the client (JDBC) side you can increase [LOB prefetching](https://docs.oracle.com/database/122/JJDBC/performance-extensions.htm#JJDBC23210) by setting the `oracle.jdbc.defaultLobPrefetchSize` connection property. This can improve read performance by reducing the number of database round trips when reading LOBs. By default [fetch size](https://docs.oracle.com/database/122/JJDBC/resultset.htm#JJDBC28621) rows are read in a single database round trip. However if the rows contain LOBs an additional round trip has to be performed for every LOB. Which this option a certain amount of LOB data is prefetched so that no additional round trips have to be performed for this data.


