---
layout: post
title: Anatomy of a Hibernate Proxy
---

[Hibernate ORM](https://hibernate.org/orm/) is well known for its use of [proxy](https://en.wikipedia.org/wiki/Proxy_pattern) classes to enable lazy loading.

A Hibernate proxy class

- extends the entity class
- implements [org.hibernate.proxy.HibernateProxy](https://docs.jboss.org/hibernate/orm/current/javadocs/org/hibernate/proxy/HibernateProxy.html)
- implements [org.hibernate.proxy.ProxyConfiguration](https://docs.jboss.org/hibernate/orm/current/javadocs/org/hibernate/proxy/ProxyConfiguration.html)
- has a field of type [org.hibernate.proxy.ProxyConfiguration.Interceptor](https://docs.jboss.org/hibernate/orm/current/javadocs/org/hibernate/proxy/ProxyConfiguration.Interceptor.html)
- has a `static final` field of type `java.lang.reflect.Method` for every superclass method
- routes all methods to <a href="https://docs.jboss.org/hibernate/orm/current/javadocs/org/hibernate/proxy/ProxyConfiguration.InterceptorDispatcher.html#intercept(java.lang.Object,java.lang.reflect.Method,java.lang.Object%5B%5D,java.lang.Object,org.hibernate.proxy.ProxyConfiguration.Interceptor)">org.hibernate.proxy.ProxyConfiguration.InterceptorDispatcher#intercept</a> which then dispatches to <a href="https://docs.jboss.org/hibernate/orm/current/javadocs/org/hibernate/proxy/ProxyConfiguration.Interceptor.html#intercept(java.lang.Object,java.lang.reflect.Method,java.lang.Object%5B%5D)">org.hibernate.proxy.ProxyConfiguration.Interceptor#intercept</a>



the generated proxy class looks like this

```java
public class EntityClass$HibernateProxy$iPrgCr9u extends EntityClass implements HibernateProxy, ProxyConfiguration {

  private ProxyConfiguration.Interceptor $$_hibernate_interceptor;
   
  private static final Method cachedValue$xVIlXpzP$4cscpe1 = Object.class.getMethod("toString");
   
  private static final Method cachedValue$xVIlXpzP$uoilpf3 = ChildEntity.class.getMethod("setId", Long.class);

  private static final Method cachedValue$xVIlXpzP$o23rrk2 = HibernateProxy.class.getMethod("getHibernateLazyInitializer");
   
  private static final Method cachedValue$xVIlXpzP$5j4bem0 = Object.class.getMethod("equals", Object.class);
   
  private static final Method cachedValue$xVIlXpzP$gpia792 = HibernateProxy.class.getMethod("writeReplace");
   
  private static final Method cachedValue$xVIlXpzP$7m9oaq0 = Object.class.getDeclaredMethod("clone");
   
  private static final Method cachedValue$xVIlXpzP$9pqdof1 = Object.class.getMethod("hashCode");
   
  private static final Method cachedValue$xVIlXpzP$8j4rtp1 = ChildEntity.class.getMethod("getId");

  @Override
  public boolean equals(Object obj) {
    return (Boolean)InterceptorDispatcher.intercept(this, cachedValue$xVIlXpzP$5j4bem0, new Object[]{obj}, false, this.$$_hibernate_interceptor);
  }

  @Override
  public String toString() {
    return (String)InterceptorDispatcher.intercept(this, cachedValue$xVIlXpzP$4cscpe1, new Object[0], null, this.$$_hibernate_interceptor);
  }

  @Override
  public int hashCode() {
    return (Integer)InterceptorDispatcher.intercept(this, cachedValue$xVIlXpzP$9pqdof1, new Object[0], 0, this.$$_hibernate_interceptor);
  }

  @Override
  protected Object clone() throws CloneNotSupportedException {
    return InterceptorDispatcher.intercept(this, cachedValue$xVIlXpzP$7m9oaq0, new Object[0], null, this.$$_hibernate_interceptor);
  }

  @Override
  public void setId(Long id) {
    InterceptorDispatcher.intercept(this, cachedValue$xVIlXpzP$uoilpf3, new Object[]{id}, null, this.$$_hibernate_interceptor);
  }

  @Override
  public Long getId() {
    return (Long) InterceptorDispatcher.intercept(this, cachedValue$xVIlXpzP$8j4rtp1, new Object[0], null, this.$$_hibernate_interceptor);
  }

  @Override
  public Object writeReplace() {
    return InterceptorDispatcher.intercept(this, cachedValue$xVIlXpzP$gpia792, new Object[0], null, this.$$_hibernate_interceptor);
  }

  @Override
  public LazyInitializer getHibernateLazyInitializer() {
    return (LazyInitializer) InterceptorDispatcher.intercept(this, cachedValue$xVIlXpzP$o23rrk2, new Object[0], null, this.$$_hibernate_interceptor);
  }

  @Override
  public void $$_hibernate_set_interceptor(ProxyConfiguration.Interceptor interceptor) {
    this.$$_hibernate_interceptor = interceptor;
  }
}


```

A few things are important to note here:

1. All methods that can be intercepted are intercepted
1. Proxies can be cast to `HibernateProxy` or `ProxyConfiguration` and their methods can be invoked by anybody
1. Every object can have its own `Interceptor`

If you want to have a look at Hibernate proxy classes for yourself give [Hibernate Proxy Dumper](https://github.com/marschall/hibernate-proxy-dumper) a try.

