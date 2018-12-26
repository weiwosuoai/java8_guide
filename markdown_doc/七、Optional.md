首先，`Optional` 它不是一个函数式接口，设计它的目的是为了防止空指针异常（`NullPointerException`），要知道在 Java 编程中，
空指针异常可是臭名昭著的。

让我们来快速了解一下 `Optional` 要如何使用！你可以将 `Optional` 看做是包装对象（可能是 `null`, 也有可能非 `null`）的容器。当你定义了
一个方法，这个方法返回的对象可能是空，也有可能非空的时候，你就可以考虑用 `Optional` 来包装它，这也是在 Java 8 被推荐使用的做法。

```java
Optional<String> optional = Optional.of("bam");

optional.isPresent();           // true
optional.get();                 // "bam"
optional.orElse("fallback");    // "bam"

optional.ifPresent((s) -> System.out.println(s.charAt(0)));     // "b"
```

 

