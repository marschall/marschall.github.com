---
layout: post
title: JDK 11 Builds with Shenandoah GC
---

The following JDK 11 builds have the [Shenandoah GC](https://wiki.openjdk.java.net/display/shenandoah/Main) [backport](https://bugs.openjdk.java.net/browse/JDK-8250784) enabled using `-XX:+UseShenandoahGC`


| Build                                                                                            | Shenandoah GC |
| -----------------------------------------------------------------------------------------------: | :-: |
| [AdoptOpenJDK](https://github.com/alibaba/dragonwell11/releases)                                 | no  |
| [Alibaba Dragonwell](https://github.com/alibaba/dragonwell11/releases)                           | ??? |
| [Amazon Coretto](https://docs.aws.amazon.com/corretto/latest/corretto-11-ug/downloads-list.html) | yes |
| [Azul Zulu CE](https://www.azul.com/downloads/zulu-community/)                                   | yes |
| [BellSoft Liberica JDK](https://bell-sw.com/pages/downloads/)                                    | no  |
| [OpenJDK Project Builds](https://adoptopenjdk.net/upstream.html)                                 | no  |
| [RedHat OpenJDK Builds](https://developers.redhat.com/products/openjdk/download)                 | ??? |
| [SapMachine](https://sap.github.io/SapMachine/)                                                  | no  |

