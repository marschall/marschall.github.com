---
layout: post
title: Anatomy of a CGLIB Proxy
---

CGLIB allows proxy class generation where normal JDK proxies are not an option. Most notably this covers the cases when a class rather than in interface needs to be proxied.

In order to find out what such a generated class looks like you have to set the `cglib.debugLocation` system property to a folder into which the generated classes should be stored. This can be done using the following JVM command line argument `-Dcglib.debugLocation=/home/user/cglib`.


So for a simple class 

```java
public class SimpleClass {

  @Cacheable
  public String method(String s) {
    return s;
  }

}
```

the generated proxy looks like this, the code has been reformatted a bit for readability.

```java
public class SimpleClass$$EnhancerByCGLIB$$2703a4e5 extends SimpleClass {

  public final String method(String s) {
    try {
      MethodInterceptor interceptor = this.CGLIB$CALLBACK_0;
      if (this.CGLIB$CALLBACK_0 == null) {
        CGLIB$BIND_CALLBACKS(this);
        interceptor = this.CGLIB$CALLBACK_0;
      }

      if (interceptor != null) {
        return (String) interceptor.intercept(this, CGLIB$method$0$Method, new Object[] { s }, CGLIB$method$0$Proxy);
      } else {
        return super.cacheable(s);
      }
    } catch (Error | RuntimeException e) {
      throw e;
    } catch (Throwable e) {
      throw new UndeclaredThrowableException(e);
    }
  }

  private boolean CGLIB$BOUND;
  private MethodInterceptor CGLIB$CALLBACK_0;
  private static final ThreadLocal<Callback[]> CGLIB$THREAD_CALLBACKS;
  private static final Callback[] CGLIB$STATIC_CALLBACKS;
  private static final Method CGLIB$method$0$Method;
  private static final MethodProxy CGLIB$cacheable$0$Proxy;

  private static final void CGLIB$BIND_CALLBACKS(Object toBind) {
    SimpleClass$$EnhancerByCGLIB$$2703a4e5 proxyToBind = (SimpleClass$$EnhancerByCGLIB$$2703a4e5) toBind;
    if (!proxyToBind.CGLIB$BOUND) {
      proxyToBind.CGLIB$BOUND = true;
      Object callbacks = CGLIB$THREAD_CALLBACKS.get();
      if (callbacks == null) {
        callbacks = CGLIB$STATIC_CALLBACKS;
        if (CGLIB$STATIC_CALLBACKS == null) {
          return;
        }
      }

      Callback[] callbacksArray = (Callback[]) callbacks;
      // …
      proxyToBind.CGLIB$CALLBACK_0 = (MethodInterceptor) callbacksArray[0];
    }

  }

}
```

A few things to note here:

1. The code does not use [safe publication](https://shipilev.net/blog/2014/safe-public-construction/) exhibits a [benign race](https://shipilev.net/blog/2016/close-encounters-of-jmm-kind/#wishful-benign-is-resilient) if _safe initialization_ rules are followed for the `MethodInterceptor`.
1. Apart from the lazy initialization the code is very similar to a [java interface proxy](https://marschall.github.io/2017/07/11/java-proxy-anatomy.html), the different being only an additional constant being passed.


