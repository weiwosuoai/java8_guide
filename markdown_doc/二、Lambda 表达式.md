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


