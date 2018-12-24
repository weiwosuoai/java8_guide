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
