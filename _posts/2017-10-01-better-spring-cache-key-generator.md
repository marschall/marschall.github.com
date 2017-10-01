---
layout: post
title: A Better Spring Cache KeyGenerator
---

The default key generators for Spring Cache [SimpleKeyGenerator](https://github.com/spring-projects/spring-framework/blob/master/spring-context/src/main/java/org/springframework/cache/interceptor/SimpleKeyGenerator.java) and [SimpleKey](https://github.com/spring-projects/spring-framework/blob/master/spring-context/src/main/java/org/springframework/cache/interceptor/SimpleKey.java) only consider the argument types of a method and not the method itself. This means that if you have two different methods with the same argument types (eg. `Integer` or `String`) and the arguments themselves are equal then Spring Cache will return results from one methods in calls to the other method if the same cache is used for both methods.

To illustarte the issue consider the following example

```java
@CacheConfig(cacheNames = "default")
public class SampleService {

  @Cacheable
  public Model1 getModel1(Integer id) {
    return // ...
  }

  @Cacheable
  public Model2 getModel2(Integer id) {
    return // ...
  }

}
```

If you first call `#getModel1` with `1` and then call `#getModel2` with `1` then Spring Cache will return the value that `#getModel1` returned because all the method arguments are equal. This is almost certainly not what you want.


The following key generator solves this issue. It also works correctly in the face of multiple implementations of the same interface method.


```java
/**
 * Considers the method in addition to the method arguments.
 *
 * <p>{@link org.springframework.cache.interceptor.SimpleKeyGenerator} only
 * considers the method arguments for caching. So if you have two different
 * methods that take the same {@link Integer} or {@link String} they will
 * interfere. In addition when you have two implementations of the same
 * interface the {@link Method} object that you will get in the
 * {@link #generate(Object, Method, Object...)} method will be the interface
 * method and therefore be the same for both implemetations.</p>
 */
final class WorkingKeyGenerator implements KeyGenerator {

  @Override
  public Object generate(Object target, Method method, Object... params) {
    return new WorkingKey(target.getClass(), method.getName(), params);
  }

  /**
   * Like {@link org.springframework.cache.interceptor.SimpleKey} but considers the method.
   */
  static final class WorkingKey {

    private final Class<?> clazz;
    private final String methodName;
    private final Object[] params;
    private final int hashCode;


    /**
     * Initialize a key.
     *
     * @param clazz the receiver class
     * @param methodName the method name
     * @param params the method parameters
     */
    WorkingKey(Class<?> clazz, String methodName, Object[] params) {
      this.clazz = clazz;
      this.methodName = methodName;
      this.params = params;
      int code = Arrays.deepHashCode(params);
      code = 31 * code + clazz.hashCode();
      code = 31 * code + methodName.hashCode();
      this.hashCode = code;
    }

    @Override
    public int hashCode() {
      return this.hashCode;
    }

    @Override
    public boolean equals(Object obj) {
      if (this == obj) {
        return true;
      }
      if (!(obj instanceof WorkingKey)) {
        return false;
      }
      WorkingKey other = (WorkingKey) obj;
      if (this.hashCode != other.hashCode) {
        return false;
      }

      return this.clazz.equals(other.clazz)
          && this.methodName.equals(other.methodName)
          && Arrays.deepEquals(this.params, other.params);
    }

  }

}

```

Then configure Spring Cache to use this key generator.


```java
public class WorkingCachingConfigurer extends CachingConfigurerSupport {

  @Override
  public KeyGenerator keyGenerator() {
    return new WorkingKeyGenerator();
  }

}
```

