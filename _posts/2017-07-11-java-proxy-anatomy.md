---
layout: post
title: Anatomy of a Java Proxy
---

The [Dynamic Proxy Classes](https://docs.oracle.com/javase/8/docs/technotes/guides/reflection/proxy.html) facility of Java allows to define dynamic [proxies](https://en.wikipedia.org/wiki/Proxy_pattern) for interfaces at runtime. It is important to note that dynamic proxies in Java require almost no JVM support. They are normal Java classes whose byte code is generated at runtime. The only JVM support required is for turning that byte code into a class. In Java 9 there is an API for this in the form of `MethodHandles.Lookup#defineClass(byte[])` but Oracle keeps on using the internal `Unsafe` class (Java 8 uses JNI code).

Two classes are essential for understanding how proxies work in Java. [java.lang.reflect.Proxy](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/Proxy.html) is the superclass of all proxies and the allows to create new proxy instances and [java.lang.reflect.InvocationHandler](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/InvocationHandler.html) is then called by the proxy.

In order to find out what such a generated class looks like you have to set the `sun.misc.ProxyGenerator.saveGeneratedFiles` system property to `true` using the following JVM command line argument `-Dsun.misc.ProxyGenerator.saveGeneratedFiles=true`. This will save all generated proxy classes to files in a package directory structure in the working directory of the JVM.


So for a simple interface

```java
interface SampleInterface {

  String method();

}
```

the generated proxy looks like this

```java
final class $Proxy4 extends Proxy implements SampleInterface {

  private static Method m1;
  private static Method m2;
  private static Method m3;
  private static Method m0;

  public $Proxy4(InvocationHandler var1)  {
    super(var1);
  }

  public final boolean equals(Object var1) {
    try {
      return (Boolean) super.h.invoke(this, m1, new Object[]{var1});
    } catch (RuntimeException | Error var3) {
      throw var3;
    } catch (Throwable var4) {
      throw new UndeclaredThrowableException(var4);
    }
  }

  public final String toString() {
    try {
      return (String) super.h.invoke(this, m2, (Object[]) null);
    } catch (RuntimeException | Error var2) {
      throw var2;
    } catch (Throwable var3) {
      throw new UndeclaredThrowableException(var3);
    }
  }

  // this method is defined in SampleInterface
  public final String method() {
    try {
      return (String) super.h.invoke(this, m3, (Object[]) null);
    } catch (RuntimeException | Error var2) {
      throw var2;
    } catch (Throwable var3) {
      throw new UndeclaredThrowableException(var3);
    }
  }

  public final int hashCode() {
    try {
      return (Integer) super.h.invoke(this, m0, (Object[]) null);
    } catch (RuntimeException | Error var2) {
      throw var2;
    } catch (Throwable var3) {
      throw new UndeclaredThrowableException(var3);
    }
  }

  static {
    try {
      m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[]{Class.forName("java.lang.Object")});
      m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
      m3 = Class.forName("com.acme.SampleInterface").getMethod("method", new Class[0]);
      m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
    } catch (NoSuchMethodException var2) {
      throw new NoSuchMethodError(var2.getMessage());
    } catch (ClassNotFoundException var3) {
      throw new NoClassDefFoundError(var3.getMessage());
    }
  }
}
```

A few things are important to note here:

1. `super.h.` is the `InvocationHandler` stored in the superclass (`Proxy`).
1. The proxy classes are cached per classloader and interface array pair. That means if you create a new proxy and there has already been a proxy class generated for this classloader and these interfaces then that class will be instantiated instead of a new one being generated.
1. All the `java.lang.reflect.Method` instances for all interface methods are kept in constants. This means they are [live](http://www.memorymanagement.org/glossary/l.html#live) as long as the classloader for which the proxy was generated is live.

An interesting detail is that when annotations are accessed through the Java reflection API a proxy class is generated for every annotation class. So for an annotation like this 

```java
@Retention(RUNTIME)
@interface SampleAnnotation {

  String value();

}
```

a proxy class like this is generated

```java
final class $Proxy5 extends Proxy implements SampleAnnotation {

  private static Method m1;
  private static Method m2;
  private static Method m4;
  private static Method m0;
  private static Method m3;

  public $Proxy5(InvocationHandler var1) {
    super(var1);
  }

  public final boolean equals(Object var1) {
    try {
      return (Boolean) super.h.invoke(this, m1, new Object[]{var1});
    } catch (RuntimeException | Error var3) {
      throw var3;
    } catch (Throwable var4) {
      throw new UndeclaredThrowableException(var4);
    }
  }

  public final String toString() {
    try {
      return (String) super.h.invoke(this, m2, (Object[]) null);
    } catch (RuntimeException | Error var2) {
      throw var2;
    } catch (Throwable var3) {
      throw new UndeclaredThrowableException(var3);
    }
  }

  public final Class annotationType() {
    try {
      return (Class) super.h.invoke(this, m4, (Object[]) null);
    } catch (RuntimeException | Error var2) {
      throw var2;
    } catch (Throwable var3) {
      throw new UndeclaredThrowableException(var3);
    }
  }

  public final int hashCode() {
    try {
      return (Integer) super.h.invoke(this, m0, (Object[]) null);
    } catch (RuntimeException | Error var2) {
      throw var2;
    } catch (Throwable var3) {
      throw new UndeclaredThrowableException(var3);
    }
  }

  // method defined in the annotation
  public final String value() {
    try {
      return (String) super.h.invoke(this, m3, (Object[]) null);
    } catch (RuntimeException | Error var2) {
      throw var2;
    } catch (Throwable var3) {
      throw new UndeclaredThrowableException(var3);
    }
  }

  static {
    try {
      m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[]{Class.forName("java.lang.Object")});
      m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
      m4 = Class.forName("com.acme.SampleAnnotation").getMethod("annotationType", new Class[0]);
      m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
      m3 = Class.forName("com.acme.SampleAnnotation").getMethod("value", new Class[0]);
    } catch (NoSuchMethodException var2) {
      throw new NoSuchMethodError(var2.getMessage());
    } catch (ClassNotFoundException var3) {
      throw new NoClassDefFoundError(var3.getMessage());
    }
  }
}
```

The actual values for the annotation attributes are then stored in a `Map` in the `InvocationHandler`.


