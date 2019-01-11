# Java8 新特性指导手册

> 本教程翻译整理自 [https://github.com/winterbe/java8-tutorial](https://github.com/winterbe/java8-tutorial)

---

> 想拥有更好的阅读体验，请访问 :corn:   [**Java8 新特性指导手册**](https://www.exception.site/course/3/chapter/1)   :corn:

## 教程目录：

- [一、接口内允许添加默认实现的方法](#接口内允许添加默认实现的方法)
- [二、Lambda 表达式](#Lambda-表达式)
- [三、函数式接口 Functional Interface](#函数式接口-Functional-Interface)
- [四、便捷的引用类的构造器及方法](#便捷的引用类的构造器及方法)
- [五、Lambda 访问外部变量及接口默认方法](#Lambda-访问外部变量及接口默认方法)
    - [5.1 访问局部变量](#访问局部变量)
    - [5.2 访问成员变量和静态变量](#访问成员变量和静态变量)
    - [5.3 访问接口的默认方法](#访问接口的默认方法)
- [六、内置的函数式接口](#内置的函数式接口)
    - [6.1 Predicate 断言](#Predicate-断言)
    - [6.2 Function](#Function)
    - [6.3 Supplier 生产者](#Supplier-生产者)
    - [6.4 Consumer 消费者](#Consumer-消费者)
    - [6.5 Comparator](#Comparator)
- [七、Optional](#Optional)
- [八、Streams 流](#Stream-流)
    - [8.1 Filter 过滤](#Filter-过滤)
    - [8.2 Sorted 排序](#Sorted-排序)
    - [8.3 Map](#Map-转换)
    - [8.4 Match](#Match-匹配)
    - [8.5 Count](#Count-计数)
    - [8.6 Reduce](#Reduce)
- [九、Parallel Streams 并行流](#Parallel-Streams-并行流)
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

## 函数式接口 Functional Interface

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

## Lambda 访问外部变量及接口默认方法

在本章节中，我们将会讨论如何在 lambda 表达式中访问外部变量（包括：局部变量，成员变量，静态变量，接口的默认方法.），它与匿名内部类访问外部变量很相似。

### 访问局部变量

在 Lambda 表达式中，我们可以访问外部的 `final` 类型变量，如下面的示例代码：

```java
// 转换器
@FunctionalInterface
interface Converter<F, T> {
    T convert(F from);
}
```

```java
final int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);

stringConverter.convert(2);     // 3
```

与匿名内部类不同的是，我们不必显式声明 `num` 变量为 `final` 类型，下面这段代码同样有效：

```java
int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);

stringConverter.convert(2);     // 3
```

但是 `num` 变量必须为隐式的 `final` 类型，何为隐式的 `final` 呢？就是说到编译期为止，`num` 对象是不能被改变的，如下面这段代码，就不能被编译通过：

```java
int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);
num = 3;
```

在 lambda 表达式内部改变 `num` 值同样编译不通过，需要注意, 比如下面的示例代码：

```java
int num = 1;
Converter<Integer, String> converter = (from) -> {
	String value = String.valueOf(from + num);
	num = 3;
	return value;
};
```
    
### 访问成员变量和静态变量

上一章节中，了解了如何在 Lambda 表达式中访问局部变量。与局部变量相比，在 Lambda 表达式中对成员变量和静态变量拥有读写权限：

```java
    @FunctionalInterface
    interface Converter<F, T> {
        T convert(F from);
    }
```

```java
class Lambda4 {
        // 静态变量
        static int outerStaticNum;
        // 成员变量
        int outerNum;

        void testScopes() {
            Converter<Integer, String> stringConverter1 = (from) -> {
                // 对成员变量赋值
                outerNum = 23;
                return String.valueOf(from);
            };

            Converter<Integer, String> stringConverter2 = (from) -> {
                // 对静态变量赋值
                outerStaticNum = 72;
                return String.valueOf(from);
            };
        }
    }
```
    
### 访问接口的默认方法

还记得第一章节中定义的那个 `Formula` (公式) 接口吗？

```java
@FunctionalInterface
interface Formula {
	// 计算
	double calculate(int a);

	// 求平方根
	default double sqrt(int a) {
		return Math.sqrt(a);
	}
}
```

当时，我们在接口中定义了一个带有默认实现的 `sqrt` 求平方根方法，在匿名内部类中我们可以很方便的访问此方法：

```java
Formula formula = new Formula() {
	@Override
	public double calculate(int a) {
		return sqrt(a * 100);
	}
};
```

但是在 lambda 表达式中可不行：

```java
Formula formula = (a) -> sqrt(a * 100);
```

带有默认实现的接口方法，是**不能**在 lambda 表达式中访问的，上面这段代码将无法被编译通过。

## 内置的函数式接口

JDK 1.8 API 包含了很多内置的函数式接口。其中就包括我们在老版本中经常见到的 Comparator 和 Runnable，Java 8 为他们都添加了 @FunctionalInterface 注解，以用来支持 Lambda 表达式。

值得一提的是，除了 Comparator 和 Runnable 外，还有一些新的函数式接口，它们很多都借鉴于知名的 [Google Guava](https://github.com/google/guava) 库。

对于它们，即使你已经非常熟悉了，还是最好了解一下的：

### Predicate 断言

`Predicate` 是一个可以指定入参类型，并返回 boolean 值的函数式接口。它内部提供了一些带有默认实现的方法，可以
被用来组合一个复杂的逻辑判断（`and`, `or`, `negate`）：

```java
Predicate<String> predicate = (s) -> s.length() > 0;

predicate.test("foo");              // true
predicate.negate().test("foo");     // false

Predicate<Boolean> nonNull = Objects::nonNull;
Predicate<Boolean> isNull = Objects::isNull;

Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isNotEmpty = isEmpty.negate();
```
    
### Function

`Function` 函数式接口的作用是，我们可以为其提供一个原料，他给生产一个最终的产品。通过它提供的默认方法，组合,链行处理(`compose`, `andThen`)：

```java
Function<String, Integer> toInteger = Integer::valueOf;
Function<String, String> backToString = toInteger.andThen(String::valueOf);

backToString.apply("123");     // "123"
```
    
### Supplier 生产者

`Supplier` 与 `Function` 不同，它不接受入参，直接为我们生产一个指定的结果，有点像生产者模式：

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

```java
Supplier<Person> personSupplier = Person::new;
personSupplier.get();   // new Person
```
    
### Consumer 消费者

对于 `Consumer`，我们需要提供入参，用来被消费，如下面这段示例代码：

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

```java
Consumer<Person> greeter = (p) -> System.out.println("Hello, " + p.firstName);
greeter.accept(new Person("Luke", "Skywalker"));
```
    
### Comparator

`Comparator` 在 Java 8 之前是使用比较普遍的。Java 8 中除了将其升级成了函数式接口，还为它拓展了一些默认方法：

```java
Comparator<Person> comparator = (p1, p2) -> p1.firstName.compareTo(p2.firstName);

Person p1 = new Person("John", "Doe");
Person p2 = new Person("Alice", "Wonderland");

comparator.compare(p1, p2);             // > 0
comparator.reversed().compare(p1, p2);  // < 0
```

## Optional

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

## Stream 流

这一章节，我们开始步入学习 `Stream` 流。

_什么是 `Stream` 流？_

简单来说，我们可以使用 `java.util.Stream` 对一个包含一个或多个元素的集合做各种操作。这些操作可能是 _中间操作_ 亦或是 _终端操作_。
终端操作会返回一个结果，而中间操作会返回一个 `Stream` 流。

需要注意的是，你只能对实现了 `java.util.Collection` 接口的类做流的操作。

> `Map` 不支持 `Stream` 流。

`Stream` 流支持同步执行，也支持并发执行。 

让我们开始步入学习的旅程吧！Go !

### Filter 过滤

首先，我们创建一个 `List` 集合：

```java
List<String> stringCollection = new ArrayList<>();
stringCollection.add("ddd2");
stringCollection.add("aaa2");
stringCollection.add("bbb1");
stringCollection.add("aaa1");
stringCollection.add("bbb3");
stringCollection.add("ccc");
stringCollection.add("bbb2");
stringCollection.add("ddd1");
```

`Filter` 的入参是一个 `Predicate`, 上面已经说到，`Predicate` 是一个断言的中间操作，它能够帮我们筛选出我们需要的集合元素。它的返参同样
是一个 `Stream` 流，我们可以通过 `foreach` 终端操作，来打印被筛选的元素：

```java
stringCollection
    .stream()
    .filter((s) -> s.startsWith("a"))
    .forEach(System.out::println);

// "aaa2", "aaa1"
```

> 注意：`foreach` 是一个终端操作，它的返参是 `void`, 我们无法对其再次进行流操作。

### Sorted 排序

`Sorted` 同样是一个中间操作，它的返参是一个 `Stream` 流。另外，我们可以传入一个 `Comparator` 用来自定义排序，如果不传，则使用默认的排序规则。

```java
stringCollection
    .stream()
    .sorted()
    .filter((s) -> s.startsWith("a"))
    .forEach(System.out::println);

// "aaa1", "aaa2"
```

需要注意，`sorted` 不会对 `stringCollection` 做出任何改变，`stringCollection` 还是原有的那些个元素，且顺序不变：

```java
System.out.println(stringCollection);
// ddd2, aaa2, bbb1, aaa1, bbb3, ccc, bbb2, ddd1
```

### Map 转换

中间操作 `Map` 能够帮助我们将 `List` 中的每一个元素做功能处理。例如下面的示例，通过 `map` 我们将每一个 `string` 转成大写：

```java
stringCollection
    .stream()
    .map(String::toUpperCase)
    .sorted((a, b) -> b.compareTo(a))
    .forEach(System.out::println);

// "DDD2", "DDD1", "CCC", "BBB3", "BBB2", "AAA2", "AAA1"
```

另外，我们还可以做对象之间的转换，业务中比较常用的是将 `DO`（数据库对象） 转换成 `DTO`（业务对象） 。

### Match 匹配

顾名思义，`match` 用来做匹配操作，它的返回值是一个 `boolean` 类型。通过 `match`, 我们可以方便的验证一个 `list` 中是否存在某个类型的元素。

```java
// 验证 list 中 string 是否有以 a 开头的, 匹配到第一个，即返回 true
boolean anyStartsWithA =
    stringCollection
        .stream()
        .anyMatch((s) -> s.startsWith("a"));

System.out.println(anyStartsWithA);      // true

// 验证 list 中 string 是否都是以 a 开头的
boolean allStartsWithA =
    stringCollection
        .stream()
        .allMatch((s) -> s.startsWith("a"));

System.out.println(allStartsWithA);      // false

// 验证 list 中 string 是否都不是以 z 开头的,
boolean noneStartsWithZ =
    stringCollection
        .stream()
        .noneMatch((s) -> s.startsWith("z"));

System.out.println(noneStartsWithZ);      // true
```

### Count 计数

`count` 是一个终端操作，它能够统计 `stream` 流中的元素总数，返回值是 `long` 类型。

```java
// 先对 list 中字符串开头为 b 进行过滤，让后统计数量
long startsWithB =
    stringCollection
        .stream()
        .filter((s) -> s.startsWith("b"))
        .count();

System.out.println(startsWithB);    // 3
```

### Reduce

`Reduce` 中文翻译为：_减少、缩小_。通过入参的 `Function`，我们能够将 `list` 归约成一个值。它的返回类型是 `Optional` 类型。

```java
Optional<String> reduced =
    stringCollection
        .stream()
        .sorted()
        .reduce((s1, s2) -> s1 + "#" + s2);

reduced.ifPresent(System.out::println);
// "aaa1#aaa2#bbb1#bbb2#bbb3#ccc#ddd1#ddd2"
```

## Parallel-Streams 并行流





