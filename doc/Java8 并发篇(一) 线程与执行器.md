# Java8 并发篇(一) | 线程与执行器

> 本文翻译整理自 https://winterbe.com/posts/2015/04/07/java8-concurrency-tutorial-thread-executor-examples/

欢迎进入 Java8 并发篇系列。此系列大概包括三篇教程：

- [线程与执行器 Threads 与 Executors](#1)
- [同步与锁 Synchronization and Locks](#1)
- [原子变量与 ConcurrentMap](#1)

本篇博文是此系列的第一篇，接下来的 15 分钟里，我将会通过一些简单易懂的示例代码来教会你，如何在 Java8 中进行并发编程，学会如何通过 `Thread`,  `Runable` 和 `Executor` 来并行执行代码。

关于 JDK 中 [并发 API](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/package-summary.html) 是在 JDK1.5 中被首次引入的，并且在后续的版本中得到不断地增强。这篇文章中介绍的大部分名词概念同样适用于老版本，这一点你不用担心。本文的着重点在代码演示，如何使用 `lambda` 表达式以及新特性相关。

如果你对 `lambda` 表达式不是很熟悉，我推荐你先阅读我之前的文章：

## 目录

- [一、Threads 与 Runnables](#1)

- [二、执行器 Executor](#1)

- [三、Callable 和 Future](#1)

- [四、超时设置](#1)

- [五、invokeAll](#1)

- [六、invokeAny](#1)

- [七、任务调度 ScheduledExecutor](#1)

  ![](https://github.com/weiwosuoai/java8_guide/blob/master/images/20190121141956.png?raw=true)

## Threads 与 Runnables 

可以肯定的是，所有的现代操作系统支持并发的手段无外乎**进程**和**线程**。进程通常可以理解为独立运行的程序实例，打个比方，你启动一个 Java 程序，这个时候操作系统就会创建一个新的进程，它可以与其他程序并行运行。而在这些进程的内部，我们可以通过线程来并发执行一段代码，一段业务逻辑。这样做有啥好处？好处就是，我们可以最大程度的发挥机器多核 CPU 的优势。

Java 从 1.0 版本就开始支持线程了。可以说是最基本的功能了，在老版本中，我们通常会实现 `Runnable` 接口，重写 `run()` 方法, 在方法内编写一段业务代码：

```java
Runnable task = () -> {
    String threadName = Thread.currentThread().getName();
    System.out.println("Hello " + threadName);
};

task.run();

Thread thread = new Thread(task);
thread.start();

System.out.println("Done!");
```

因为 Runnable 接口是一个函数式接口，我们直接采用 lambda 表达式来书写它，内部的业务逻辑是打印当前的线程名，控制台的输出可能存在两种情况：

```java
Hello main
Hello Thread-0
Done!
```

或者：

```java
Hello main
Done!
Hello Thread-0
```

我们无法预测 `runnable` 是在主线程执行完成后执行还是之前执行，因为这种顺序的不确定性，使得在一个体量庞大的应用中，并发编程变得异常复杂。

我们可以在线程内部设置休眠时间，来模拟一个长时间运行的任务：

```java
Runnable runnable = () -> {
    try {
        String name = Thread.currentThread().getName();
        System.out.println("Foo " + name);
        // 休眠一秒
        TimeUnit.SECONDS.sleep(1);
        System.out.println("Bar " + name);
    }
    catch (InterruptedException e) {
        e.printStackTrace();
    }
};

Thread thread = new Thread(runnable);
thread.start();
```

运行上面的代码，你会发现两条打印语句之间会间隔一秒。`TimeUnit` 在处理单位时间的时候，是一个很实用的枚举类，你也可以通过调用 `Thread.sleep(1000)` 来达到同样的目的。

我们不得不承认，使用 `Thread` 类是很乏味的且容易出错的。由于 Java 并发相关的 API 是在 2004 年 Java5 发布的时候才被正式引入的。它们被放置在 `java.util.concurrent` 包下，里面包含了很多处理并发编程有用的类。

这些并发 API 被引入后，在后续的 Java 版本中，又得到不断的增强，Java8 甚至提供了新的并发类和方法来处理并发。

废话不多说，接下来，让我们进入并发 API 中最重要的执行器： `Executor`.

## 执行器 Executor

设计 `ExecutorService` 的目的是用来替代我们直接手动创建 `Thread`。`Executors` 支持运行异步任务，通常管理着一个线程池，线程池内部线程会得到复用，避免了频繁创建线程，销毁线程而带来的额外的系统开销。

```java
// 创建一个只包含一个线程的线程池
ExecutorService executor = Executors.newSingleThreadExecutor();
executor.submit(() -> {
	String threadName = Thread.currentThread().getName();
	System.out.println("Hello " + threadName);
});

// => Hello pool-1-thread-1
```

`Executors` 类提供了便利的工厂方法用来不同类型的线程池。上面的示例中我们创建了一个只包含单线程的线程池 `executor`。

代码的输出结果与上面手动 new Runnable() 输出一致，唯一不同的是，**Java 进程没有终止**！

![](https://github.com/weiwosuoai/java8_guide/blob/master/images/20190121141052.jpg?raw=true)

> 注意：`Executors` 必须显示的终止它，否则它们将持续监听是否有新的任务需要执行。这也是为什么我在研发组中推荐小组人员创建线程池务必要交给 Spring Ioc 容器管理的原因。

那么，如何优雅的关闭 `ExecutorService` 呢？

`ExecutorService` 提供了两个方法：

- 1.`shutdwon()` : 它会等待正在执行的任务执行完成后，再销毁线程池；
- 2.`shutdownNow()`： 它会立刻终止正在执行的任务，并销毁线程池；

关闭 `ExecutorService` 最佳实践：

```java
try {
    System.out.println("attempt to shutdown executor");
    executor.shutdown();
    // 指定关闭之前的等待时间 5s，达到“温柔滴”关闭
    executor.awaitTermination(5, TimeUnit.SECONDS);
}
catch (InterruptedException e) {
    System.err.println("tasks interrupted");
}
finally {
    if (!executor.isTerminated()) {
        System.err.println("cancel non-finished tasks");
    }
    executor.shutdownNow();
    System.out.println("shutdown finished");
}
```

## Callable 和 Future

除了 `Runnable`, `executor` 还支持另外一种任务类型 —— `Callable`。`Callable` 和 `Runnable` 类似，唯一不同的是，`Callable` 有返回值。

下面的示例代码中，我们通过 lambda 表达式定义了一个 `Callable`：休眠 1s 钟后，返回一个整数。

```java
Callable<Integer> task = () -> {
    try {
        TimeUnit.SECONDS.sleep(1);
        return 123;
    }
    catch (InterruptedException e) {
        throw new IllegalStateException("task interrupted", e);
    }
};
```

`Callable` 也可以像 `Runnable` 一样，作为入参，提交给 `executor`。这样的话，问题来了，我们怎么取到返回值呢？因为 `submit()` 方法不会阻塞等待任务完成的。

虽然 `executor` 不能直接返回 `Callable` 结果，不过，`executor` 可以返回一个 `Future` 类型的结果，他可以在稍后的某个时机取出实际的返回值。

```java
ExecutorService executor = Executors.newFixedThreadPool(1);
Future<Integer> future = executor.submit(task);

System.out.println("future done? " + future.isDone());

// 这里会阻塞等待返回结果
Integer result = future.get();

System.out.println("future done? " + future.isDone());
System.out.print("result: " + result);
```

上面的这段代码，`Callable` 提交给 `executor` 后，我们先通过调用 `isDone()` 检查 `future` 是否执行完成，结果当然是 `false`, 因为 `future` 在返回那个整数之前，会休眠 1s。

后面到执行 `get()` 方法，线程会阻塞等待返回结果。整个流程走完，我们再来看下控制台的输出：

```java
future done? false
future done? true
result: 123
```

你需要注意，如果你关闭 `executor`, 所有为关闭的 `future` 都会抛出异常。

```java
executor.shutdownNow();
future.get();
```

还有一点，你可能也注意到了，这次我们创建线程池并非使用 `newSingleThreadExecutor()`。而是通过`newFixedThreadPool(1)`来创建一个单线程线程池的 `executor`。 它等同于使用`newSingleThreadExecutor`，不过使用第二种方式我们动态的调整入参，比如传入一个比 1 大的值来增加线程池的大小。

## 超时 Timeouts

上面提到了，`future.get()` 那里会阻塞等待，直到 `callable` 返回值。我们试想一个最糟糕的情况，`callable` 持续运行导致你的程序没有响应，这显然是不能忍受的。我们可以通过传入一个超时时间来避免发生这种情况：

```java
ExecutorService executor = Executors.newFixedThreadPool(1);

Future<Integer> future = executor.submit(() -> {
    try {
        TimeUnit.SECONDS.sleep(2);
        return 123;
    }
    catch (InterruptedException e) {
        throw new IllegalStateException("task interrupted", e);
    }
});

// 设置超时时间为 1s
future.get(1, TimeUnit.SECONDS);
```

执行上面这段代码，会抛出一个 TimeoutException：

```java
Exception in thread "main" java.util.concurrent.TimeoutException
    at java.util.concurrent.FutureTask.get(FutureTask.java:205)
```

为什么抛出这个异常，原因很显然，我们指定的超时时间是 1s, 而任务中光休眠就是 2s 了。

## invokeAll

`Executors` 支持通过 `invokeAll` 方法一次性批量提交多个 `callable`。这个方法的返回结果是一个 `future` 集合：

```java
ExecutorService executor = Executors.newWorkStealingPool();

List<Callable<String>> callables = Arrays.asList(
        () -> "task1",
        () -> "task2",
        () -> "task3");

executor.invokeAll(callables)
    .stream()
    .map(future -> {
        try {
            // 拿到每个 future 的返回值
            return future.get();
        }
        catch (Exception e) {
            throw new IllegalStateException(e);
        }
    })
    .forEach(System.out::println); // for 循环输出结果
```

上面这段示例代码，我们先利用 Java8 的 Stream 流来处理 `invokeAll()` 方法返回的 `future` 集合，取出每个 `future` 的返回值将其映射到一个 `List<String>` 集合中，最后循环输出结果。

## invokeAny

批量提交的另外一种方式是 `invokeAny()` , 同样是批量提交，它与 `invokeAll()` 不同点在于：它会阻塞等待，当某个 `callable` 第一个执行完成，它会立刻返回执行结果，而不再等待那些正在执行中的 `callable` 了。

为了测试，我们定义一个方法，用来模拟创建拥有不同执行时间的 `callable`：

```java
Callable<String> callable(String result, long sleepSeconds) {
    return () -> {
        TimeUnit.SECONDS.sleep(sleepSeconds);
        return result;
    };
}
```

然后，我们创建一组 `callable`, 他们拥有不同的执行时间，1s 到 3s 的，通过`invokeAny()`将这些 callable 提交给`executor`，看看返回最快的 `callable` 的字符串结果是哪个：

```java
ExecutorService executor = Executors.newWorkStealingPool();

List<Callable<String>> callables = Arrays.asList(
callable("task1", 2),
callable("task2", 1),
callable("task3", 3));

String result = executor.invokeAny(callables);
System.out.println(result);

// => task2
```

最快的是 task2, 它的执行时间最短，所以输出是它。

上面这段示例代码中，创建 `ExecutorService` 的方式又变了，通过 `Executors.newWorkStealingPool()`。这个工厂方法是 Java8 才引入的，返回值是一个 `ForkJoinPool`类型的 `executor`，它和指定固定大小的线程池不同，`ForkJoinPools` 允许指定一个并行因子来创建，默认的值为物理机可用的 CPU 核心数。

`ForkJoinPools` 是在 Java7 中才被引入的，这将会在后面系列的教程中做详细介绍。敬请期待。

## 任务调度 ScheduledExecutor

上面我们已经学习了如何在一个 `executor` 中提交和运行一次任务。还有中场景是，我们需要多次运行一个常见的任务，这个时候，我们可以利用调度线程池。

`ScheduledExecutorService`支持任务调度，持续执行或者延迟一段时间后再执行。

下面的代码示例，指定一个任务延迟 3 分钟再执行：

```java
ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);

Runnable task = () -> System.out.println("Scheduling: " + System.nanoTime());
ScheduledFuture<?> future = executor.schedule(task, 3, TimeUnit.SECONDS);

TimeUnit.MILLISECONDS.sleep(1337);

long remainingDelay = future.getDelay(TimeUnit.MILLISECONDS);
System.out.printf("Remaining Delay: %sms", remainingDelay);
```

`ScheduledExecutorService` 会返回一个增强类型的 `future` 类型 —— `ScheduleFuture`，它除了保有原有 `Future` 提供的所有方法外，还提供了 `getDelay()` 方法, 用来获取距离目标时间的时间差，也就是剩余的延迟时间。

为了保证调度任务的持续执行，`executors` 提供了两个 API:

- 1.`scheduleAtFixedRate()` ：**固定周期执行一个任务, 不考虑任务的耗时，即使上个任务还没有执行完毕，到了时间，下个任务依然会去执行**;

```java
ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);

Runnable task = () -> System.out.println("Scheduling: " + System.nanoTime());

// 第一次执行的延迟时间
int initialDelay = 0;
// 周期频率
int period = 1;
// 指定任务立刻执行，且每次之间间隔为 1s
executor.scheduleAtFixedRate(task, initialDelay, period, TimeUnit.SECONDS);
```

> 注意：`initialDelay` 表示第一次执行的延迟时间，0 表示立即执行

- 2.`scheduleWithFixedDelay()` ：**固定延迟执行任务，就是说，必须要等到上个任务执行完成后才开始计时，达到设定的延迟时间后，下个任务才会被触发**；

```java
ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);

Runnable task = () -> {
    try {
        TimeUnit.SECONDS.sleep(2);
        System.out.println("Scheduling: " + System.nanoTime());
    }
    catch (InterruptedException e) {
        System.err.println("task interrupted");
    }
};

executor.scheduleWithFixedDelay(task, 0, 1, TimeUnit.SECONDS);
```

> 总结一下：`scheduleWithFixedDelay()` 和 `scheduleAtFixedRate()` 最大的区别就是，`scheduleWithFixedDelay()`需要等到前一个任务执行完成后才开始计延时，再触发下一个任务。

如上面的示例代码中这个调度任务，设置了 1s 的延时，初始化延时为 0，假设理想情况下任务的耗时就是 2s, 则任务的调度周期为： `0s -> 3s -> 6s -> 9s ....`, 它更适用于当你无法预测任务的执行时长的场景中使用。

## 总结

这篇文章主要探讨了如何在用 `lambda` 表达式，在 Java8 中使用线程和执行器，并演示了相关示例代码。除此之外，我们还学习了 `Callable` 和 `Future`, 超时设置，任务的批量提交 `invokeAll()` 和 `invokeAny()` API。最后, 了解了如何在 Java8 中使用任务调度 `ScheduledExecutor`，希望你能有所收获。