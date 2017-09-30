---
layout: post
title: A Better Spring Cache KeyGenerator
published: false
---

The default key generators for Spring Cache [SimpleKeyGenerator](https://github.com/spring-projects/spring-framework/blob/master/spring-context/src/main/java/org/springframework/cache/interceptor/SimpleKeyGenerator.java) and [SimpleKey](https://github.com/spring-projects/spring-framework/blob/master/spring-context/src/main/java/org/springframework/cache/interceptor/SimpleKey.java) only consider the argument types of a method and not the method itself. This means that if you have two different methods with the same argument types (eg. `Integer` or `String`) and the arguments themselves are equal then Spring Cache will return results from one methods in calls to the other method if the same cache is used for both methods.

The following key generator solves this issue and also works correctly in the face of multiple implementations of the same interface method.


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
class WorkingKeyGenerator implements KeyGenerator {

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

Configure Spring Cache to use this key generator.


```java
public class P2XCachingConfigurer extends CachingConfigurerSupport {

  @Override
  public KeyGenerator keyGenerator() {
    return new WorkingKeyGenerator();
  }

}
```

