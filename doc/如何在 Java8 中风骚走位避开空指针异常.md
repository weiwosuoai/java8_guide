# 如何在 Java8 中风骚走位避开空指针异常

> 文章整理翻译自 [https://winterbe.com/posts/2015/03/15/avoid-null-checks-in-java/](https://winterbe.com/posts/2015/03/15/avoid-null-checks-in-java/)

![](https://exception-image-bucket.oss-cn-hangzhou.aliyuncs.com/154773518012868)

要说 Java 编程中哪个异常是你印象最深刻的，那 `NullPointerException` 空指针可以说是臭名昭著的。不要说初级程序员会碰到，
即使是中级，专家级程序员稍不留神，就会掉入这个坑里。

`Null` 引用的发明者 [Tony Hoare](http://en.wikipedia.org/wiki/Tony_Hoare) 曾在 2009 年作出道歉声明，声明中表示，到目前为止，空指针异常大约给企业已造成数十亿美元的损失。

下面是 Tony Hoare 的原话：

> 我将 Null 引用的设计称为是一个数十亿美元的错误。1965 那年，我正在用面向对象语言(ALGOL W) 设计首个功能全面的系统。当时我的考量是，确保所有被使用的引用都是安全的，编译器会自动进行检查。但是，我没有抵住诱惑，加入了 Null 引用，仅仅是为了实现起来省事。这之后，它导致了数不清的 bug、错误和系统崩溃，也为企业导致了不可估量的损失。

事已至此，我们必须学会面对它。So, 我们要如何防止空指针异常呢？

唯一的办法就是对可能为 Null 的对象添加检查。但是 Null 检查是繁琐且痛苦的。所以一些比较新的语言为了处理 Null 检查，特意添加了特殊的语法，如[空合并运算符](http://en.wikipedia.org/wiki/Null_coalescing_operator)。

> 在 [Groovy](http://groovy-lang.org/operators.html#_elvis_operator) 或 [Kotlin](http://kotlinlang.org/docs/reference/null-safety.html) 这样的语言中也被称为 `Elvis` 运算符。

不幸的是，在老版本的 Java 中并没有提供这样的语法糖。Java8 中在这方面做了改进。所以，这篇文章就特意来介绍一下如何在 Java8 中利用新特性来编写防止 `NullPointerException`的发生。

## Java8 中如何加强对 Null 对象的检查？

在上篇文章 [Java8 新特性指导手册]() 中简单的提了一下如何通过 `Optional` 类来对对象做空校验。接下来，我们再细说一下：

![](https://exception-image-bucket.oss-cn-hangzhou.aliyuncs.com/154773333584293)

在业务系统中，对象中嵌套对象是经常发生的场景，如下示例代码：

```java
// 最外层对象
class Outer {
    Nested nested;
    Nested getNested() {
        return nested;
    }
}
// 第二层对象
class Nested {
    Inner inner;
    Inner getInner() {
        return inner;
    }
}
// 最底层对象
class Inner {
    String foo;
    String getFoo() {
        return foo;
    }
}
```

业务中，假设我们需要获取 `Outer` 对象对底层的 `Inner` 中的 `foo` 属性，我们必须写一堆的非空校验，来防止发生 `NullPointerException`：

```java
// 繁琐的代码
Outer outer = new Outer();
if (outer != null && outer.nested != null && outer.nested.inner != null) {
    System.out.println(outer.nested.inner.foo);
}
```

## 通过 Optional

在 Java8 中，我们有更优雅的解决方式，那就是使用 `Optional`是说，我们可以在一行代码中，进行流水式的 `map` 操作。而 **map 方法内部会自动进行空校验**：

```java
Optional.of(new Outer())
    .map(Outer::getNested)
    .map(Nested::getInner)
    .map(Inner::getFoo
    .ifPresent(System.out::println); // 如果不为空，最终输出 foo 的值
```

## 通过 suppiler 函数自定义增强 API

上面这种方式个人感觉还是有点啰嗦，我们可以利用 `suppiler` 函数来出一个终极解决方案：

```java
public static <T> Optional<T> resolve(Supplier<T> resolver) {
    try {
        T result = resolver.get();
        return Optional.ofNullable(result);
    }
    catch (NullPointerException e) {
        // 可能会抛出空指针异常，直接返回一个空的 Optional 对象
        return Optional.empty();
    }
}
```

利用上面的 `resolve` 方法来重构上述的非空校验代码段：

```java
Outer obj = new Outer();
// 直接调用 resolve 方法，内部做空指针的处理
resolve(() -> obj.getNested().getInner().getFoo());
    .ifPresent(System.out::println); // 如果不为空，最终输出 foo 的值
```

## 最后

你需要知道的是，上面这两个解决方案并没传统的 `null` 检查性能那么高效。但在绝大部分业务场景下，舍弃那么一丢丢的性能来方便编码，是完全可取，
除非是那种对性能有严格要求的场景，我们才不建议使用。

> 个人觉得，真要拿这点性能说事，还不如去优化优化 sql 语句，业务逻辑等。

## 小哈的微信公众号，欢迎关注

![](https://exception-image-bucket.oss-cn-hangzhou.aliyuncs.com/154753049695780)
