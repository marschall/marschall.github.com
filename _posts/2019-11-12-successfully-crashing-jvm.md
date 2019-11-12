---
layout: post
title: Successfully Crashing a JVM in Production
---

JVMs can occasionally crash in production. To be able to debug the cause it helps to be prepared. One of the most important things is to have [core dumps](http://man7.org/linux/man-pages/man5/core.5.html) enabled. This can be done by setting or raising the core dump limit. The core dump limit can be checked with `ulimit -c`, a value of `0` means core dumps are disabled, a value of `unlimited` means there is no size limit for core dumps.


```$ulimit -c
unlimited
```

A core dump will be larger than your heap, at least the size of the [RSS](https://en.wikipedia.org/wiki/Resident_set_size) so make sure you have enough space left of the core dump partition. As a rule of thumb, the core dump partition should a least be the size of the physical memory plus swap.

As with everything else it worth testing this to make sure everything works. A simple way to crash a Java program is to send [SIGSEGV](https://en.wikipedia.org/wiki/Segmentation_fault) to a Java process. `SIGSEGV` has the value of 11, we can use `kill` to send this signal.

```
kill -11 <pid>
```

```
#
# A fatal error has been detected by the Java Runtime Environment:
#
#  SIGSEGV (0xb) at pc=0x00007feec256acd5 (sent by kill), pid=3875, tid=3875
#
# JRE version: JRE version
# Java VM: Java VM
# Problematic frame:
# C  [libpthread.so.0+0xacd5]  __GI___pthread_timedjoin_ex+0x225
#
# Core dump will be written. Default location: /var/crash/core/%e_%u_%g_%t_%s_%p
#
# An error report file with more information is saved as:
# /opt/tomcat/hs_err_pid3875.log
#
# If you would like to submit a bug report, please visit:
#   http://www.vendor.com/support/
#
```

If everything is successful two files should have been created. A `hs_err_pid.log` in the working directory of the crashed Java process and a core sump file. Make sure these are on persistent volumes. Keep in mind these files contain your command line arguments and all the data loaded into the application, treat them accordingly.

Once a crash happened make sure to move the core dump partition to make sure there is enough space left for the next crash. Often a next step is to create a back trace using [GDB](https://www.gnu.org/software/gdb/)

```
gdb /path/to/java-X.Y.Z/bin/java CORE_FILE
# only if debug symbols not in JVM folder
set debug-file-directory DEBUG_SYMBOL_FOLDER
# only if java binary at a different place
set solib-search-path /lib64:/path/to/java-X.Y.Z/lib:/path/to/java-X.Y.Z/lib/server:/path/to/java-X.Y.Z/lib/jli
set height 0
set logging file bt_all.txt
set logging redirect on
set logging on
thread apply all bt
quit

```


