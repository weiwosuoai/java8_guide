# Java8 新特性指导手册

> 本教程翻译整理自 [https://github.com/winterbe/java8-tutorial](https://github.com/winterbe/java8-tutorial)

---

> 想拥有更好的阅读体验，请访问 :corn:   [**Java8 新特性指导手册**](https://www.exception.site/course/3/chapter/1)   :corn:

## 教程目录：

- [一、接口内允许添加默认实现的方法](#接口内允许添加默认实现的方法)

- [二、Lambda 表达式](#Lambda-表达式)

- [三、函数式接口（Functional Interface）](#函数式接口（Functional Interface）)

- [四、便捷的引用类的构造器及方法](#便捷的引用类的构造器及方法)

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

## 接口内允许添加默认实现的方法

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

## Lambda 表达式

在学习 `Lambda` 表达式之前，我们先来看一段老版本的示例代码，其对一个含有字符串的集合进行排序：

```java
List<String> names = Arrays.asList("peter", "anna", "mike", "xenia");

Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return b.compareTo(a);
    }
});
```

`Collections` 工具类提供了静态方法 `sort` 方法，入参是一个 `List` 集合，和一个 `Comparator` 比较器，以便对给定的 `List` 集合进行
排序。上面的示例代码创建了一个匿名内部类作为入参，这种类似的操作在我们日常的工作中随处可见。

Java 8 中不再推荐这种写法，而是推荐使用 Lambda 表达：

```java
Collections.sort(names, (String a, String b) -> {
    return b.compareTo(a);
});
```

正如你看到的，上面这段代码变得简短很多而且易于阅读。但是我们还可以再精炼一点：

```java
Collections.sort(names, (String a, String b) -> b.compareTo(a));
```

对于只包含一行方法的代码块，我们可以省略大括号，直接 `return` 关键代码即可。追求极致，我们还可以让它再短点：

```java
names.sort((a, b) -> b.compareTo(a));
```

`List` 集合现在已经添加了 `sort` 方法。而且 Java 编译器能够根据**类型推断机制**判断出参数类型，这样，你连入参的类型都可以省略啦，怎么样，是不是感觉很强大呢！

## 函数式接口（Functional Interface）

抛出一个疑问：在我们书写一段 Lambda 表达式后（比如上一章节中匿名内部类的 Lambda 表达式缩写形式），Java 编译器是如何进行类型推断的，它又是怎么知道重写的哪个方法的？

需要说明的是，不是每个接口都可以缩写成 Lambda 表达式。只有那些函数式接口（Functional Interface）才能缩写成 Lambda 表示式。

那么什么是函数式接口（Functional Interface）呢？

所谓函数式接口（Functional Interface）就是只包含一个抽象方法的声明。针对该接口类型的所有 Lambda 表达式都会与这个抽象方法匹配。

> 注意：你可能会有疑问，Java 8 中不是允许通过 defualt 关键字来为接口添加默认方法吗？那它算不算抽象方法呢？答案是：不算。因此，你可以毫无顾忌的添加默认方法，它并不违反函数式接口（Functional Interface）的定义。

总结一下：只要接口中仅仅包含一个抽象方法，我们就可以将其改写为 Lambda 表达式。为了保证一个接口明确的被定义为一个函数式接口（Functional Interface），我们需要为该接口添加注解：`@FunctionalInterface`。这样，一旦你添加了第二个抽象方法，编译器会立刻抛出错误提示。

示例代码：

    @FunctionalInterface
    interface Converter<F, T> {
        T convert(F from);
    }
	
示例代码2：

	Converter<String, Integer> converter = (from) -> Integer.valueOf(from);
	Integer converted = converter.convert("123");
	System.out.println(converted);    // 123
	
> 注意：上面的示例代码，即使去掉 `@FunctionalInterface` 也是好使的，它仅仅是一种约束而已。

## 便捷的引用类的构造器及方法

小伙伴们，还记得上一个章节这段示例代码么：

```java
@FunctionalInterface
interface Converter<F, T> {
    T convert(F from);
}
```

```java
Converter<String, Integer> converter = (from) -> Integer.valueOf(from);
Integer converted = converter.convert("123");
System.out.println(converted);    // 123
```

上面这段代码，通过 Java 8 的新特性，进一步简化上面的代码：

```java
Converter<String, Integer> converter = Integer::valueOf;
Integer converted = converter.convert("123");
System.out.println(converted);   // 123
```

Java 8 中允许你通过 `::` 关键字来引用类的方法或构造器。上面的代码简单的示例了如何引用静态方法，当然，除了静态方法，我们还可以引用普通方法：

```java
class Something {
    String startsWith(String s) {
        return String.valueOf(s.charAt(0));
    }
}
```

```java
Something something = new Something();
Converter<String, String> converter = something::startsWith;
String converted = converter.convert("Java");
System.out.println(converted);    // "J"
```
接下来，我们再来看看如何通过 `::` 关键字来引用类的构造器。首先，我们先来定义一个示例类，在类中声明两个构造器：

```java
class Person {
    String firstName;
    String lastName;

    Person() {}

    Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
}
```

然后，我们再定义一个工厂接口，用来生成 `Person` 类：

```java
// Person 工厂
interface PersonFactory<P extends Person> {
    P create(String firstName, String lastName);
}
```

我们可以通过 `::` 关键字来引用 `Person` 类的构造器，来代替手动去实现这个工厂接口：

```java
// 直接引用 Person 构造器
PersonFactory<Person> personFactory = Person::new;
Person person = personFactory.create("Peter", "Parker");
```
`Person::new` 这段代码，能够直接引用 `Person` 类的构造器。然后 Java 编译器能够根据上下文选中正确的构造器去实现 `PersonFactory.create` 方法。





