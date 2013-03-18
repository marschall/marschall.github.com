---
layout: post
title: Tales of a Java 7 Migration
---

This week we rolled out a new release of a large, 6 to 7 year old Java I application. Among other things we migrated to Java 7 right before Java 6 became [end of life](http://www.oracle.com/technetwork/java/eol-135779.html). The migration was fairly straight forward and simple. The biggest reason why we could't migrate earlier is because our whole tool chain ([Eclipse](http://eclipse.org/eclipse/), [FindBugs](http://findbugs.sourceforge.net/), [Checkstyle](http://checkstyle.sourceforge.net/) and [Sonar](http://www.sonarsource.org/)) had to be Java 7 ready.

As documented in the [Java SE 7 and JDK 7 Compatibility](http://www.oracle.com/technetwork/java/javase/compatibility-417013.html) Java 7 is mostly backwards compatible, still there were minor changes that affected us.

The order of the methods returned by reflection [has changed](http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=7023180). We were not affected by this but we ran into [JBAS-6981](https://issues.jboss.org/browse/JBAS-6981) which is fixed in JBoss AS 6 and we back ported to JBoss AS 5.1.

Java 7 byte code (major 51.0) [has to be verified using the type-checking verifier](http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=6693236) meaning stack map frames have to be present. This was not an issue for us directly but we use two libraries that use [CGLib](http://cglib.sourceforge.net/) for class file generation ([Spring](http://www.springsource.org/spring-framework) and [EasyMock](http://www.easymock.org/)) and CGLib 2.x was not able to produce stack map frames. CGlib 3.0 fixes this and Spring 3.2 ships with a [custom](https://jira.springsource.org/browse/SPR-7484) CGLib version. However this version is in the Spring name space and therefore not available to EasyMock and our code coverage reports were failing because of this. As CGLib 3.0 is not available in [Maven Central](http://search.maven.org/) we manually deployed it to our in-house [Nexus](http://www.sonatype.org/nexus/). We sill ran into a [bug in CGLib 3.0](http://sourceforge.net/tracker/?func=detail&aid=3601081&group_id=56933&atid=482368) so we had to make a custom release. Unfortunately CGLib does not seem actively maintained so we don't have much hope of our fix being merged soon.


The following code compiles with Java 6 but not with Java 7

{% highlight java %}
public interface MultiValueMap<K, V> extends Map<K, Collection<V>> {

  void put(K key, V value);

}
{% endhighlight %}
we ended up renaming the method.

The class <code>sun.management.ManagementFactory</code> is no longer available, we switched to <code>java.lang.management.ManagementFactory</code> (which we should have used in the first place).

The class <code>java.text.SimpleDateFormat</code> seems to behave sightly differently when used in not-documented ways (if you follow the contract, it will behave the same in Java 6 and 7).

We used a couple of regexes to migrate our code to [diamond operator](http://docs.oracle.com/javase/7/docs/technotes/guides/language/type-inference-generic-instance-creation.html), [try-with-resources](http://docs.oracle.com/javase/7/docs/technotes/guides/language/try-with-resources.html) and [multi-catch](http://docs.oracle.com/javase/7/docs/technotes/guides/language/catch-multiple.html) which worked OK. Then we found out that [IntelliJ IDEA](http://www.jetbrains.com/idea/) can do this automatically for us so we ran it over the whole code base which worked out very well.
