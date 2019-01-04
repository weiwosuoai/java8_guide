# Java8 新特性指导手册

> 本教程翻译整理自 [https://github.com/winterbe/java8-tutorial](https://github.com/winterbe/java8-tutorial)

---

> 想拥有更好的阅读体验，请访问 :corn:   [**Java8 新特性指导手册**](https://www.exception.site/course/3/chapter/1)   :corn:

## 教程目录：

- [一、接口内允许添加默认实现的方法](#一、接口内允许添加默认实现的方法)

- [二、Lambda 表达式](https://github.com/weiwosuoai/java8_guide/blob/master/markdown_doc/%E4%BA%8C%E3%80%81Lambda%20%E8%A1%A8%E8%BE%BE%E5%BC%8F.md)

- [三、函数式接口（Functional Interface）](https://github.com/weiwosuoai/java8_guide/blob/master/markdown_doc/%E4%B8%89%E3%80%81%E5%87%BD%E6%95%B0%E5%BC%8F%E6%8E%A5%E5%8F%A3%EF%BC%88Functional%20Interface%EF%BC%89.md)

- [四、便捷的引用类的构造器及方法](https://github.com/weiwosuoai/java8_guide/blob/master/markdown_doc/%E5%9B%9B%E3%80%81%E4%BE%BF%E6%8D%B7%E7%9A%84%E5%BC%95%E7%94%A8%E7%B1%BB%E7%9A%84%E6%9E%84%E9%80%A0%E5%99%A8%E5%8F%8A%E6%96%B9%E6%B3%95.md)

- [五、Lambda 访问外部变量及接口默认方法](https://github.com/weiwosuoai/java8_guide/blob/master/markdown_doc/%E4%BA%94%E3%80%81Lambda%20%E8%AE%BF%E9%97%AE%E5%A4%96%E9%83%A8%E5%8F%98%E9%87%8F%E5%8F%8A%E6%8E%A5%E5%8F%A3%E9%BB%98%E8%AE%A4%E6%96%B9%E6%B3%95.md)

    - [5.1 访问局部变量](https://github.com/weiwosuoai/java8_guide/blob/master/markdown_doc/5.1%20%E8%AE%BF%E9%97%AE%E5%B1%80%E9%83%A8%E5%8F%98%E9%87%8F.md)
    
    - [5.2 访问成员变量和静态变量](https://github.com/weiwosuoai/java8_guide/blob/master/markdown_doc/5.2%20%E8%AE%BF%E9%97%AE%E6%88%90%E5%91%98%E5%8F%98%E9%87%8F%E5%92%8C%E9%9D%99%E6%80%81%E5%8F%98%E9%87%8F.md)
    
    - [5.3 访问接口的默认方法](https://github.com/weiwosuoai/java8_guide/blob/master/markdown_doc/5.3%20%E8%AE%BF%E9%97%AE%E6%8E%A5%E5%8F%A3%E7%9A%84%E9%BB%98%E8%AE%A4%E6%96%B9%E6%B3%95.md)

- [六、内置的函数式接口](https://github.com/weiwosuoai/java8_guide/blob/master/markdown_doc/%E5%85%AD%E3%80%81%E5%86%85%E7%BD%AE%E7%9A%84%E5%87%BD%E6%95%B0%E5%BC%8F%E6%8E%A5%E5%8F%A3.md)

    - [6.1 Predicate (断言)](https://github.com/weiwosuoai/java8_guide/blob/master/markdown_doc/6.1%20Predicate(%E6%96%AD%E8%A8%80).md)
    
    - [6.2 Function](https://github.com/weiwosuoai/java8_guide/blob/master/markdown_doc/6.2%20Function.md)
    
    - [6.3 Supplier (生产者)](https://github.com/weiwosuoai/java8_guide/blob/master/markdown_doc/6.3%20Supplier(%E7%94%9F%E4%BA%A7%E8%80%85).md)
    
    - [6.4 Consumer（消费者）](https://github.com/weiwosuoai/java8_guide/blob/master/markdown_doc/6.4%20Consumer(%E6%B6%88%E8%B4%B9%E8%80%85).md)
    
    - [6.5 Comparator](https://github.com/weiwosuoai/java8_guide/blob/master/markdown_doc/6.5%20Comparator.md)

- [七、Optional](https://github.com/weiwosuoai/java8_guide/blob/master/markdown_doc/%E4%B8%83%E3%80%81Optional.md)

- 8.`Streams` 流；

- 9.并行流（`Parallel Streams`）;

- 10.`Maps`;

- 11.新添加的日期 API;

- 12.注解（`Annotations`）;

也希望学完本系列教程的小伙伴能够熟练掌握和应用 Java8 的各种特性，使其成为在工作中的一门利器。废话不多说，让我们一起开启 Java8 新特性之旅吧！

## 一、接口内允许添加默认实现的方法

Java 8 允许我们通过 `default` 关键字对接口中定义的抽象方法提供一个默认的实现。

请看下面示例代码：

```java
// 定义一个公式接口
interface Formula {
    // 计算
    double calculate(int a);

    // 求平方根
    default double sqrt(int a) {
        return Math.sqrt(a);
    }
}
```

在上面这个接口中，我们除了定义了一个抽象方法 `calculate`，还定义了一个带有默认实现的方法 `sqrt`。
我们在实现这个接口时，可以只需要实现 `calculate` 方法，默认方法 `sqrt` 可以直接调用即可，也就是说我们可以不必强制实现 `sqrt` 方法。

> 补充：通过 `default` 关键字这个新特性，可以非常方便地对之前的接口做拓展，而此接口的实现类不必做任何改动。

```java
Formula formula = new Formula() {
    @Override
    public double calculate(int a) {
        return sqrt(a * 100);
    }
};

formula.calculate(100);     // 100.0
formula.sqrt(16);           // 4.0
```
上面通过匿名对象实现了 `Formula` 接口。但是即使是这样，我们为了完成一个 `sqrt(a * 100)` 简单计算，就写了 6 行代码，很是冗余。

下个章节中，我们会学习 Lambda 表达式，更加优雅地书写代码。


