# 第十三章 异步任务

现代工作负载需要更多关于如何高效利用系统资源的考虑。异步任务是提高应用响应性的绝佳工具，可以避免性能瓶颈。

Java 8 引入了新类型`CompletableFuture<T>`，它在以前的`Future<T>`类型基础上进行了改进，通过声明式和函数式方法创建异步任务。

本章介绍了为什么以及如何利用异步编程，以及`CompletableFuture<T>`相比 JDK 之前的异步任务更灵活和功能更强的方法。

# 同步与异步

同步和异步任务的概念并不局限于软件开发。

例如，面对面会议或电话会议是同步活动，至少是如果你专心的话。除了参与和可能做笔记外，你无法做其他任何事情。每个其他任务在会议/电话结束之前都被*阻塞*。如果会议/电话本应该是一封电子邮件——正如我大多数会议本应该的那样——则你当前的任务不会因需要立即注意而中断以前的任务。因此，电子邮件是*非阻塞*通信。

软件开发中也适用同样的原则。同步执行的任务按顺序运行，阻塞后续工作直到它们完成。从单线程的角度看，阻塞任务意味着等待结果，可能会浪费资源，因为在任务完成之前不做任何其他事情。

异步任务是启动一个在“其他地方”处理的任务，并在完成时得到通知。通过并发技术将它们的工作分发出去——通常是到另一个线程——以便它们不必等待完成。因此，当前线程不被阻塞，可以继续执行其他任务，正如图 13-1 中所示。

![同步与异步执行的比较](img/afaj_1301.png)

###### 图 13-1 同步与异步执行的比较

并行执行，正如我在第八章中讨论的，追求最大吞吐量作为其主要目标；单个任务的完成时间通常不是大计划中的重点。另一方面，像`CompletableFuture`这样的异步执行模型专注于系统的整体延迟和响应性。分发任务确保了响应迅速的系统，即使在单线程或资源受限的环境中也是如此。

# Java Futures

Java 5 引入了接口 `java.util.concurrent.Future<T>` 作为异步计算结果的容器类型。要创建一个 `Future`，需要将任务以 `Runnable` 或 `Callable<T>` 的形式提交给 `ExecutorService`，后者在单独的线程中启动任务，但立即返回一个 `Future` 实例。这样，当前线程可以继续执行更多工作，而不必等待 `Future` 计算的最终结果。

可以通过在 `Future<T>` 实例上调用 `get` 方法来检索结果，如果计算尚未完成，可能会阻塞当前线程。通常的流程示例在 示例 13-1 中有所体现。

##### 示例 13-1\. `Future<T>` 执行流程

```java
var executor = Executors.newFixedThreadPool(10); ![1](img/1.png)

Callable<Integer> expensiveTask = () -> { ![2](img/2.png)

    System.out.println("(task) start");

    TimeUnit.SECONDS.sleep(2);

    System.out.println("(task) done");

    return 42;
};

System.out.println("(main) before submitting the task");

var future = executor.submit(expensiveTask); ![3](img/3.png)

System.out.println("(main) after submitting the task");

var theAnswer = future.get(); ![4](img/4.png)

System.out.println("(main) after the blocking call future.get()");

// OUTPUT:
// (main) before submitting the task
// (task) start
// (main) after submitting the task
// ~~ 2 sec delay ~~
// (task) done
// (main) after the blocking call future.get()
```

![1](img/#co_asynchronous_tasks_CO1-1)

需要显式的 `ExecutorService` 来启动 `Callable<T>` 或 `Runnable`。

![2](img/#co_asynchronous_tasks_CO1-2)

`Callable<T>` 接口在函数接口引入 lambda 之前就已经存在了。它的预期用法相当于 `Supplier<T>`，但在其单一抽象方法中抛出 `Exception`。

![3](img/#co_asynchronous_tasks_CO1-3)

`expensiveTask` 的计算立即开始，并反映在输出中。

![4](img/#co_asynchronous_tasks_CO1-4)

此时计算尚未完成，因此在 `future` 上调用 `get` 方法会阻塞当前线程，直到完成为止。

虽然 `Future<T>` 类型达到了作为 *非阻塞* 异步计算容器的基本要求，但其功能集仅限于几种方法：检查计算是否完成、取消计算和检索结果。

为了拥有一个多功能的异步编程工具，还有很多功能有待改进：

+   更容易地获取结果，如在完成或失败时的回调。

+   在函数组合的精神中链接和组合多个任务。

+   集成错误处理和恢复可能性。

+   无需 `ExecutorService`，可以手动创建或完成任务。

Java 8 在 Futures 的基础上进行了改进，通过引入接口 `CompletionStage<T>` 和其唯一实现 `CompletableFuture<T>` 来弥补缺失的功能。它们位于同一包 `java.util.concurrent` 中，是构建异步任务管道的多功能工具，功能集比它们之前的 Futures 更丰富。其中，`Future<T>` 是一个异步计算最终值的容器类型，而 `CompletionStage<T>` 表示异步管道的单个阶段，具有超过 70 个方法的大量 API！

# 使用 CompletableFuture<T> 设计异步管道

CompletableFuture 的一般设计哲学与 Streams 类似：两者都是基于任务的管道，提供接受常见函数接口的参数化方法。新的 API 添加了大量的协调工具，返回`CompletionStage<T>`或+CompletableFuture<T>的新实例。这种异步计算和协调工具的结合提供了以前缺失的所有功能，以流畅的组合和声明性 API。

由于庞大的`CompletableFuture<T>`API 和异步编程的复杂思维模型，让我们从一个简单的比喻开始：*做早餐*。

想象的早餐由咖啡，面包和鸡蛋组成。按照同步 — 或*阻塞* — 顺序准备早餐并没有太多意义。在咖啡壶完成或面包机完成之前等待开始煎鸡蛋是对可用资源的浪费，这将不必要地增加总准备时间，让你在坐下来吃饭时已经饿了。相反，你可以在咖啡壶和烤面包机工作时开始煎鸡蛋，只有当烤面包机弹出或咖啡壶完成时才对它们做出反应。

编程也是如此。可用资源应根据需要分配，不要浪费在*昂贵*且耗时长的任务上等待。这种异步管道的基本概念在许多语言中都以不同的名称存在，也许更常见的是*Promises*。

## 承诺一个值

*Promises*是具有内置协调工具的异步管道的构建模块，允许链接和组合多个任务，包括错误处理。这样一个构建块要么是*pending*（未解决），要么是*resolved*（已解决且计算完成），要么是*rejected*（已解决，但处于错误状态）。在组合管道中在状态之间移动是通过在两个通道之间切换：*数据*和*错误*来完成的，如图 13-2 所示。

![Promise data and error channels](img/afaj_1302.png)

###### 图 13-2. 承诺数据和错误通道

数据通道是“快乐路径”，如果一切顺利的话。然而，如果一个承诺失败，管道将切换到错误通道。这样，失败就不会像流（Streams）那样使整个管道崩溃，可以优雅地处理，甚至可以恢复并将管道切换回数据通道。

正如你所看到的，CompletableFuture API 就是另一个名字的 Promise。

## 创建一个 CompletableFuture<T>

与其前身`Future<T>`一样，新的`CompletableFuture<T>`类型不提供任何构造函数来创建实例。新的`Future<T>`实例是通过将任务提交给`java.util.concurrent.ExecutorService`来创建的，后者会返回一个已经启动了任务的实例。

`CompletableFuture<T>`遵循相同的原则。然而，它不一定需要一个显式的`ExecutorService`来调度任务，这要归功于它的`static`工厂方法：

+   `CompletableFuture<Void> runAsync(Runnable runnable)`

+   `CompletableFuture<U> supplyAsync(Supplier<U> supplier)`

这两种方法还支持第二个参数，接受一个`java.util.concurrent.Executor`，这是`ExecutorService`类型的基础接口。如果选择不带`Executor`的变体，就会使用通用的 ForkJoinPool，就像在“流作为并行功能管道”中解释的并行流管道一样。

###### 注意

将任务提交给`ExecutorService`以创建`Future<T>`最明显的区别是使用`Supplier<T>`代替`Callable<T>`。后者在其方法签名中明确抛出异常。因此，`supplyAsync`不能完全替代将`Callable<T>`提交给`Executor`。

创建`CompletableFuture<T>`实例几乎等同于创建`Future<T>`实例，如示例 13-2 所示。示例没有使用类型推断，因此返回类型是可见的。通常情况下，你会更喜欢使用`var`关键字而不是显式类型。

##### 示例 13-2\. 使用便捷方法创建`CompletableFuture`

```java
// FUTURE<T>

var executorService = ForkJoinPool.commonPool();

Future<?> futureRunnable =
  executorService.submit(() -> System.out.println("not returning a value"));

Future<String> futureCallable =
  executorService.submit(() -> "Hello, Async World!");

// COMPLETABLEFUTURE<T>

CompletableFuture<Void> completableFutureRunnable =
  CompletableFuture.runAsync(() -> System.out.println("not returning a value"));

CompletableFuture<String> completableFutureSupplier =
  CompletableFuture.supplyAsync(() -> "Hello, Async World!");
```

尽管在`Future<T>`和`CompletableFuture<T>`的实例创建之间存在相似之处，但后者更为简洁，不一定需要`ExecutorService`。然而，更大的区别在于，`CompletableFuture<T>`实例提供了`CompletionStage<T>`实例的声明式和函数式管道的起点，而不是`Future<T>`的单一孤立的异步任务。

## 组合和合并任务

在使用`CompletableFuture<T>`实例开始后，可以进一步组合和组成它们，以创建更复杂的管道。

可用于构建异步管道的广泛操作可以根据其接受的参数和预期的使用案例分为三组：

转换结果

类似于流和可选项的`map`操作，CompletableFuture API 提供了类似的`thenApply`方法，它使用`Function<T, U>`来转换类型为`T`的前一个结果，并返回另一个`CompletionStage<U>`。如果转换函数返回另一个`CompletionStage`，使用`thenCompose`方法可以防止额外的嵌套，类似于流和可选项的`flatMap`操作。

消费结果

如其名称所示，`thenAccept`方法需要一个`Consumer<T>`来处理类型为`T`的前一个结果，并返回一个新的`CompletionStage<Void>`。

执行完成后

如果不需要访问前一个结果，`thenRun`方法会执行一个`Runnable`并返回一个新的`CompletionStage<Void>`。

由于存在太多方法，特别是额外的 `-Async` 方法，无法详细讨论每一个。其中大多数方法都有两个额外的 `-Async` 变体：一个与非 `-Async` 匹配，另一个带有额外的 `Executor` 参数。

非 `-Async` 方法在与前一个任务相同的线程中执行其任务，尽管这并不保证，如后文 “关于线程池和超时” 所述。 `-Async` 变体将使用一个新线程，由公共 `ForkJoinPool` 创建，或由提供的 `Executor` 创建。

为了保持简单，我将主要讨论非 `-Async` 变体。

### 组合任务

组合任务创建了一个连接的 CompletionStages 串行管道。

所有组合操作都遵循一个通用的命名方案：

```java
<operation>Async
```

`<操作>` 名称来源于操作类型及其参数，主要使用功能接口的 SAM 前缀 `then` 加上它们接受的功能接口的名称：

+   `CompletableFuture<Void> thenAccept(Consumer<? super T> action)`

+   `CompletableFuture<Void> thenRun(Runnable action)`

+   `CompletableFuture<U> thenApply(Function<? super T,? extends U> fn)`

由于 API 的良好命名方案，使用任何操作都会产生流畅且简单的调用链。例如，想象一个书签管理器，它会扫描其网站以存储永久副本。整个任务可以异步运行，因此不会阻塞 UI 线程。任务本身包括三个步骤：下载网站、准备内容以供离线使用，最后存储，如 示例 13-3 所示。

##### 示例 13-3\. 异步书签管理器工作流程

```java
var task = CompletableFuture.supplyAsync(() -> this.downloadService.get(url))
                            .thenApply(this.contentCleaner::clean)
                            .thenRun(this.storage::save);
```

组合操作仅支持 1:1，意味着它们接受前一阶段的结果并执行其预期的工作。如果您的任务管道需要多个流汇聚，您需要组合任务。

### 组合任务

将互相连接的 futures 组合以创建更复杂的任务可能非常有帮助。然而，有时不同的任务不需要或可以串行运行。在这种情况下，您可以使用接受另一个阶段的操作来组合 `CompletionStage` 实例。

它们的命名方案类似于之前的 1:1 组合操作：

```java
<operation><restriction>Async
```

额外的 `restriction` 表示操作是否对两个阶段或任一阶段有效，使用适当命名的后缀 `-Both` 和 `-Either`。

表格 13-1 列出了可用的 2:1 操作。

表格 13-1\. 组合操作

| 方法 | 参数 | 注意事项 |
| --- | --- | --- |
| `thenCombine` | `BiFunction<T, U, V>` | 在 *两个* 阶段正常完成后应用 `BiFunction`。 |
| `thenAcceptBoth` | `BiConsumer<T, U>` | 类似于 `thenCombine`，但不生成任何值。 |
| `runAfterBoth` | `Runnable` | 在两个给定阶段都正常完成后评估 `Runnable`。 |
| `applyToEither` | `Function<T, U>` | 将`Function`应用于第一个完成的阶段。 |
| `acceptEither` | `Consumer<T, U>` | 类似于`applyToEither`，但不生成任何值。 |
| `runAfterEither` | `Runnable` | 在给定阶段之一正常完成后评估`Runnable`。 |

与其他功能性 Java 特性一样，许多不同的操作归功于 Java 的静态类型系统及其如何解析通用类型。与 JavaScript 等其他语言不同，方法不能在单个参数或返回类型中接受多种类型。

可以轻松混合组合操作，如图 13-3 所示。

![组合和组合任务](img/afaj_1303.png)

###### 图 13-3\. 组合和组合任务

可用的操作提供了几乎所有用例的各种功能。但是，在 Java 的异步 API 中仍然存在一些盲点，特别是缺少特定变体：使用`BiFunction`组合两个阶段的结果，返回另一个阶段而不创建嵌套的`CompletionStage`。

`thenCombine`的行为类似于 Java 中的其他`map`操作。在嵌套返回值的情况下，需要类似于`flatMap`的操作，但对于`CompletableFuture<T>`而言，这种操作是缺失的。因此，您需要额外的`thenCompose`操作来展平嵌套的值，如示例 13-4 所示。

##### 示例 13-4\. 取消包装嵌套阶段

```java
CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(() -> 42); ![1](img/1.png)
CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(() -> 23); ![1](img/1.png)

BiFunction<Integer, Integer, CompletableFuture<Integer>> task = ![2](img/2.png)
  (lhs, rhs) -> CompletableFuture.supplyAsync(() -> lhs + rhs);

CompletableFuture<Integer> combined =
  future1.thenCombine(future2, task) ![3](img/3.png)
         .thenCompose(Function.identity()); ![4](img/4.png)
```

![1](img/#co_asynchronous_tasks_CO2-1)

应该组合其结果的两个阶段。

![2](img/#co_asynchronous_tasks_CO2-3)

消耗前一阶段组合结果的任务。

![3](img/#co_asynchronous_tasks_CO2-4)

`task`的返回值通过`thenCombine`包装成另一个阶段，导致不需要的`CompletionStage<CompletionStage<Integer>>`。

![4](img/#co_asynchronous_tasks_CO2-5)

使用`Function.identity()`的`thenCompose`调用取消包装嵌套阶段，管道再次成为`CompletionStage<Integer>`。

如果任务返回的是`CompletableFuture`本身，而不是依赖调用者通过必要时包装成`CompletableFuture`来异步处理它，则此方法很有帮助。

### 同时运行超过两个 CompletableFuture<T>

前面讨论的操作允许您运行最多两个 CompletableFutures 以创建一个新的 CompletableFuture。然而，处理超过两个的情况，如使用`thenCombine`等组合操作而不创建嵌套方法调用的噩梦，是不可能的。这就是为什么`CompletableFuture<T>`类型具有两个用于同时处理多个实例的静态便利方法的原因：

+   `CompletableFuture<Void> allOf(CompletableFuture<?>…​ cfs)`

+   `CompletableFuture<Object> anyOf(CompletableFuture<?>…​ cfs)`

`allOf` 和 `anyOf` 方法协调预先存在的实例。因此，它们都不提供匹配的 `-Async` 变体，因为每个给定的 `CompletableFuture` 实例已经有其指定的 `Executor`。协调性质的另一个方面是它们的限制性返回类型。因为两者都接受任何类型的 `CompletableFuture` 实例，由泛型边界 `<?>` 表示，所以无法确定整体结果的明确 `T`，因为类型可以自由混合。`allOf` 的返回类型是 `CompletableFuture<Void>`，因此您无法在后续阶段访问给定实例的任何结果。但是，可以创建支持将结果作为备选项返回的辅助方法。我将向您展示如何在 “创建 CompletableFuture 辅助程序” 中执行此操作，但现在，让我们先通过 CompletableFuture 的其他操作。

## 异常处理

到目前为止，我已经向您展示了仅在“快乐路径”上行进而没有任何故障的管道。但是，如果在管道中发生异常，则承诺可能被拒绝，或者在 Java 中称为*异常完成*。

与流或可选项在异常情况下炸毁整个管道不同，CompletableFuture API 将异常视为一等公民，并且是其工作流程的重要部分。这就是为什么异常处理不强加于任务本身，并且有多个可用于处理可能被拒绝的 Promise 的操作的原因：

+   `CompletionStage<T> exceptionally(Function<Throwable, T> fn)`

+   `CompletionStage<U> handle(BiFunction<T, Throwable, U> fn)`

+   `CompletionStage<T> whenComplete(BiConsumer<T, Throwable> action)`

使用 `exceptionally` 操作将异常钩子添加到管道中，如果之前的阶段没有发生异常，则会正常完成，并带有上一阶段的结果。在被拒绝的阶段，它的异常会应用到钩子的 `fn` 上，进行恢复尝试。要进行恢复，`fn` 需要返回任何类型为 `T` 的值，这将把管道切换回数据通道。如果没有可能的恢复，抛出一个新的异常，或者重新抛出已应用的异常，将使管道保持在异常完成状态并在错误通道上。

更灵活的 `handle` 操作将 `exceptionally` 和 `thenApply` 的逻辑合并为一个单独的操作。`BiFunction` 参数取决于前一阶段的结果。如果它被拒绝，那么第二个类型为 `Throwable` 的参数是非 `null` 的。否则，类型为 `T` 的第一个参数具有值。请注意，它仍然可能是一个 `null` 值。

最后一个操作 `whenComplete` 类似于 `handle` 但不提供一种恢复被拒绝 Promise 的方式。

### 数据和错误通道再访

即使我解释了 Promise 实际上有两个通道，即数据和错误，但 CompletableFuture 管道实际上是操作的一条直线，就像 Streams 一样。每个管道阶段都寻找下一个兼容的操作，具体取决于当前阶段已完成的状态。在正常完成的情况下，下一个`then`/`run`/`apply`/等就会执行。对于异常完成的阶段，这些操作是“通过”操作，并且管道会进一步寻找下一个`exceptionally`/`handle`/`whenComplete`/等操作。

CompletableFuture 管道可能是通过流畅调用创建的一条直线，将其视为两个通道，如以前在图 13-2 中所做的那样，可以更好地了解发生了什么。每个操作存在于数据通道或错误通道中的一个，除了`handle`和`whenComplete`操作，它们存在于中间，因为它们无论管道的状态如何都会执行。

### 被拒绝的 Either 任务

通过使用组合操作注入另一个 CompletableFuture，可以获得一个直线管道。您可能会认为后缀`-Either`可能意味着*要么*管道可能正常完成以创建一个新的非拒绝阶段。嗯，您会感到惊讶！

如果先前的阶段被拒绝，那么`acceptEither`操作将保持被拒绝状态，无论另一个阶段是否正常完成，如示例 13-5 所示。

##### 示例 13-5\. Either 操作和被拒绝的阶段

```java
CompletableFuture<String> notFailed =
  CompletableFuture.supplyAsync(() -> "Success!");

CompletableFuture<String> failed =
  CompletableFuture.supplyAsync(() -> { throw new RuntimeException(); });

// NO OUTPUT BECAUSE THE PREVIOUS STAGE FAILED

var rejected = failed.acceptEither(notFailed, System.out::println);

// OUTPUT BECAUSE THE PREVIOUS STAGE COMPLETED NORMALLY
var resolved = notFailed.acceptEither(failed, System.out::println);
// => Success!
```

记住的要点是，除了错误处理操作之外的所有操作都需要一个非拒绝的前一个阶段才能正常工作，即使对于`-Either`操作也是如此。如果有疑问，请使用错误处理操作来确保管道仍处于数据通道上。

## 终端操作

到目前为止，任何操作都会返回另一个`CompletionStage<T>`以进一步扩展管道。基于`Consumer`的操作可能满足许多用例，但是在某些时候，即使可能阻塞当前线程，您也需要实际的值。

与`Future<t>`类型相比，`CompletionStage<T>`类型本身不提供任何额外的检索方法。但是，它的实现`CompletableFuture<T>`提供了两个选项：`getNow`和`join`方法。这将终端操作数量增加到四个，如表 13-2 中所列。

表 13-2\. 从管道获取值

| 方法签名 | 用例 | 异常 |
| --- | --- | --- |

| `T get()` | 阻塞当前线程直到管道完成。 | `InterruptedException`（已检查）`ExecutionException`（已检查）

`CancellationException`（未检查）|

| `T get(long timeout, TimeUnit unit)` | 阻塞当前线程直到管道完成，但在达到`timeout`后抛出异常。 | `TimeoutException`（已检查）`InterruptedException`（已检查）

`ExecutionException`（已检查）

`CancellationException`（未检查）|

| `T getNow(T valueIfAbsent)` | 如果正常完成则返回管道的结果，或者抛出`CompletionException`。如果结果仍然挂起，则立即返回提供的回退值`T`而不取消管道。 | `CompletionException`（未检查） `CancellationException`（未检查） |
| --- | --- | --- |
| `join()` | 阻塞当前线程直到管道完成。 | 如果发生异常完成，对应的异常会被包装成`CompletionException`。 |

类型`CompletableFuture<T>`还添加了另一个管道协调方法，`isCompletedExceptionally`，给你总共四个影响或检索管道状态的方法，如表 13-3 所列。

表 13-3\. 协调方法

| 方法签名 | 返回 |
| --- | --- |
| `boolean cancel(boolean mayInterruptIfRunning)` | 用`CancellationException`异常异常完成尚未完成的阶段。参数`mayInterruptIfRunning`被忽略，因为与`Future<T>`不同，中断不用于控制。 |
| `boolean isCancelled()` | 如果阶段在完成之前被取消，则返回`true`。 |
| `boolean isDone()` | 如果阶段已经以任何状态完成，则返回`true`。 |
| `boolean isCompletedExceptionally()` | 返回`true`如果阶段已经异常完成，或者已经处于拒绝状态。 |

这是一个非常庞大的 API，涵盖了许多用例。尽管如此，根据您的要求，可能会缺少一些边缘案例。但是，添加您的辅助程序以填补任何差距是很容易的，所以让我们来做吧。

## 创建一个 CompletableFuture Helper

虽然 CompletableFuture API 非常庞大，但仍然缺少某些用例。例如，如前文所述在“组合任务”中，静态辅助程序`allOf`的返回类型是`CompletableFuture<Void>`，因此您无法在后续阶段访问给定实例的任何结果。它是一种灵活的仅协调方法，接受任何类型的`CompletableFuture<?>`作为其参数，但是以不访问任何结果的代价来弥补这一点。为此，您可以根据需要创建一个辅助程序来补充现有的 API。

让我们创建一个类似于`allOf`的辅助程序，一次运行多个`CompletableFuture`实例，但仍然可以访问它们的结果：

```java
static CompletableFuture<List<T>> eachOf(CompletableFuture<T> cfs...)
```

提议的辅助`eachOf`运行所有给定的`CompletableFuture`实例，就像`allOf`一样。然而，与`allOf`不同的是，新的辅助程序使用了泛型类型`T`而不是`?`（问号）。将此限制为单一类型使`eachOf`方法实际上可以返回一个`CompletableFuture<List<T>>`而不是无结果的`CompletableFuture<Void>`。

### 辅助支架

需要一个方便的`class`来保存任何辅助方法。这些辅助方法对于一些特定边缘情况非常有用，否则无法以简洁的方式或者根本无法解决。最惯用且安全的方式是使用一个带有`private`构造函数的`class`，如下所示，以防止任何人意外扩展或实例化该类型。

```java
public final class CompletableFutures {

  private CompletableFutures() {
    // SUPPRESS DEFAULT CONSTRUCTOR
  }
}
```

###### 注意

具有`private`默认构造函数的辅助类本身不必`final`，以防止其可扩展性。扩展类在没有可见的隐式`super`构造函数的情况下无法编译。然而，将辅助类设为`final`表明了所需的意图，而不依赖隐式行为。

### 设计`eachOf`

`eachOf`的目标几乎与`allOf`相同。这两种方法都协调一个或多个`CompletableFuture`实例。然而，`eachOf`进一步管理结果。这导致以下要求：

+   返回包含所有给定实例的`CompletableFuture`，就像`allOf`一样。

+   给予对成功完成实例结果的访问。

第一个要求由`allOf`方法实现。然而，第二个要求需要额外的逻辑。需要你检查给定的实例并聚合它们的结果。

任何逻辑在前一个阶段以任何方式完成后运行的最简单方法是使用`thenApply`操作，如下所示：

```java
public static <T> CompletableFuture<List<T>> eachOf(CompletableFuture<T>... cfs) {

  return CompletableFuture.allOf(cfs)
                          .thenApply(???);
}
```

利用你在本书中学到的知识，可以通过创建一个流数据处理管道来聚合成功完成的`CompletableFuture`实例的结果。

让我们逐步进行创建这样一个管道所需的步骤。

首先，必须从给定的`CompletableFuture<T>`实例创建流。它是一个`vararg`方法参数，因此对应一个数组。当处理`vararg`时，辅助方法`Arrays#stream(T[] arrays)`是明显的选择：

```java
Arrays.stream(cfs)
```

接下来，过滤成功完成的实例。虽然没有显式方法询问实例是否正常完成，但由于`Predicate.not`，可以询问其相反情况：

```java
Arrays.stream(cfs)
      .filter(Predicate.not(CompletableFuture::isCompletedExceptionally))
```

有两种方法可以立即从`CompletableFuture`中获取结果：`get()` 和 `join()`。在这种情况下，后者更可取，因为它不会抛出已检查异常，简化了流管道，如第十章所讨论的那样：

```java
Arrays.stream(cfs)
      .filter(Predicate.not(CompletableFuture::isCompletedExceptionally))
      .map(CompletableFuture::join)
```

使用`join`方法会阻塞当前线程以获取结果。然而，流管道在`allOf`完成后运行，因此所有结果已经可用。并且通过预先过滤非成功完成的元素，不会抛出可能导致管道崩溃的异常。

最后，结果被聚合到一个`List<T>`中。这可以通过`collect`操作完成，或者如果使用 Java 16+，可以使用`Stream<T>`类型的`toList`方法完成：

```java
Arrays.stream(cfs)
      .filter(Predicate.not(CompletableFuture::isCompletedExceptionally))
      .map(CompletableFuture::join)
      .toList();
```

现在可以使用流水线在 `thenApply` 调用中收集结果。`CompletableFutures` 及其 `eachOf` 辅助方法的完整实现如 示例 13-6 所示。

##### 示例 13-6\. `eachOf` 的完整实现

```java
public final class CompletableFutures {

  private final static Predicate<CompletableFuture<?>> EXCEPTIONALLY = ![1](img/1.png)
    Predicate.not(CompletableFuture::isCompletedExceptionally);

  public static <T> CompletableFuture<List<T>> eachOf(CompletableFuture<T>... cfs) {

    Function<Void, List<T>> fn = unused -> ![2](img/2.png)
      Arrays.stream(cfs)
            .filter(Predicate.not(EXCEPTIONALLY))
            .map(CompletableFuture::join)
            .toList();

    return CompletableFuture.allOf(cfs) ![3](img/3.png)
                            .thenApply(fn);
  }

  private CompletableFutures() {
    // SUPPRESS DEFAULT CONSTRUCTOR
  }
}
```

![1](img/#co_asynchronous_tasks_CO3-1)

用于测试成功完成的 `Predicate` 不绑定到特定的 `CompletableFuture` 实例，因此可重用为 `final static` 字段。

![2](img/#co_asynchronous_tasks_CO3-2)

结果收集操作由 `Function<Void, List<T>>` 表示，该函数与 `allOf` 的返回类型的内部类型和 `eachOf` 的预期返回类型相匹配。

![3](img/#co_asynchronous_tasks_CO3-3)

整体任务仅仅是调用预先存在的 `allOf` 并将其与结果聚合管道组合。

就是这样！我们为某些用例创建了 `allOf` 的替代方案，以便轻松访问结果。

最终实现是解决问题的功能方法的示例。每个任务本身都是独立的，可以单独使用。但是通过组合它们，您可以创建由较小部分构建的更复杂的解决方案。

### 改进 `CompletableFutures` 助手

`eachOf` 方法与 `allOf` 互为补充，其工作方式与预期相同。如果给定的任何 `CompletableFuture` 实例失败，返回的 `CompletableFuture<List<T>>` 也将异常完成。

对于“fire & forget”用例，您可能只关注成功完成的任务，而不关心任何失败。但是，如果尝试使用 `get` 或类似方法提取其值，则失败的 `CompletableFuture` 将抛出异常。因此，让我们添加一个基于 `eachOf` 的 `bestEffort` 辅助方法，始终成功完成并仅返回成功的结果。

主要目标与 `eachOf` 几乎相同，除非 `allOf` 调用返回了异常完成的 `CompletableFuture<Void>`，必须恢复。通过插入 `exceptionally` 操作添加异常钩子是显而易见的选择：

```java
public static
<T> CompletableFuture<List<T>> bestEffort(CompletableFuture<T>... cfs) {

  Function<Void, List<T>> fn = ...; // no changes to Stream pipeline

  return CompletableFuture.allOf(cfs)
                          .exceptionally(ex -> null)
                          .thenApply(fn);
}
```

一开始，`exceptionally` lambda `ex -> null` 看起来可能有些奇怪。但是如果您检查其底层方法签名，其意图会变得更清晰。

在这种情况下，`exceptionally` 操作需要一个 `Function<Throwable, Void>` 来通过返回 `Void` 类型的值而不是抛出异常来恢复 `CompletableFuture`。这通过返回 `null` 来实现。之后，从 `eachOf` 到聚合流水线的聚合流水线被使用来收集结果。

###### 提示

使用 `handle` 操作也可以实现相同的行为，并在单个 `BiFunction` 中处理成功或拒绝的两种状态。不过，将状态分开处理可以使管道更具可读性。

现在我们有两个具有共享逻辑的辅助方法，将共同逻辑提取到它们自己的方法中可能是合理的。这基于将孤立逻辑组合在一起以创建更复杂和完整任务的功能方法。`Futures`的可能重构实现在示例 13-7 中展示。

##### 示例 13-7\. 使用`eachOf`和`bestEffort`重构的 Futures 实现

```java
public final class CompletableFutures {

  private final static Predicate<CompletableFuture<?>> EXCEPTIONALLY = ![1](img/1.png)
    Predicate.not(CompletableFuture::isCompletedExceptionally);

  private static <T> Function<Void, List<T>>
                     gatherResultsFn(CompletableFuture<T>... cfs) { ![2](img/2.png)

    return unused -> Arrays.stream(cfs)
                      .filter(Predicate.not(EXCEPTIONALLY))
                      .map(CompletableFuture::join)
                      .toList();
  }

  public static <T> CompletableFuture<List<T>> eachOf(CompletableFuture<T>... cfs) { ![3](img/3.png)
    return CompletableFuture.allOf(cfs)
                            .thenApply(gatherResultsFn(cfs));
  }

  public static <T> CompletableFuture<List<T>> bestEffort(CompletableFuture<T>... cfs) { ![3](img/3.png)
    return CompletableFuture.allOf(cfs)
                            .exceptionally(ex -> null)
                            .thenApply(gatherResultsFn(cfs));
  }

  private CompletableFutures() {
    // SUPPRESS DEFAULT CONSTRUCTOR
  }
}
```

![1](img/#co_asynchronous_tasks_CO4-1)

`Predicate`未更改。

![2](img/#co_asynchronous_tasks_CO4-2)

结果收集逻辑被重构为一个`private`工厂方法，以确保在`eachOf`和`bestEffort`中的一致处理。

![3](img/#co_asynchronous_tasks_CO4-3)

两个`public`辅助方法都简化到最低限度。

重构后的`CompletableFutures`辅助程序比以前更简单、更健壮。任何可共享的复杂逻辑都得到重用，因此它提供了始终保持一致行为并最小化所需文档的方法，这些文档肯定应该添加以将预期功能沟通给任何调用方。

# 手动创建和完成

创建`Future<T>`实例的唯一方式（除了自己实现接口）是向`ExecutorService`提交任务。`CompletableFuture<T>`的`static`方便工厂方法`runAsync`或`supplyAsync`非常相似。与其前身不同，它们不是唯一创建实例的方式。

## 手动创建

由于`CompletableFuture<T>`类型是一个实际的实现而不是一个接口，它有一个构造函数，您可以使用它来创建一个未完成的实例，如下所示：

```java
CompletableFuture<String> unsettled = new CompletableFuture<>();
```

然而，没有附加任务，它将永远不会完成或失败。相反，您需要手动完成这样的任务。

## 手动完成

有几种方法可以安排现有的`CompletableFuture<T>`实例并启动附加的管道：

+   `boolean complete(T value)`

+   `boolean completeExceptionally(Throwable ex)`

两种方法如果调用成功将返回`true`，将阶段转换为期望状态。

Java 9 引入了额外的`complete`方法，用于正常完成的阶段，形式为`-Async`变体，以及基于超时的一个：

+   `CompletableFuture<T> completeAsync(Supplier<T> supplier)`

+   `CompletableFuture<T> completeAsync(Supplier<T> supplier, Executor executor)`

+   `CompletableFuture<T> completeOnTimeout(T value, long timeout, TimeUnit unit)`

`-Async`变体使用新的异步任务，以`supplier`的结果完成当前阶段。

`completeOnTimeout`这种方法在达到`timeout`之前，如果该阶段未能完成，就使用给定的`value`完成当前阶段。

而不是创建一个新实例然后手动完成它，您也可以使用以下这些`static`方便的工厂方法之一创建一个已经完成的实例：

+   `CompletableFuture<U> completedFuture(U value)`

+   `CompletableFuture<U> failedFuture(Throwable ex)`（Java 9+）

+   `CompletionStage<U> completedStage(U value)`（Java 9+）

+   `CompletionStage<U> failedStage(Throwable ex)`（Java 9+）

这些已经完成的未来对象可以用于任何组合操作中，或者作为 CompletableFutures 流水线的起点，正如我将在下一节中讨论的那样。

## 手动创建和完成实例的用例

实际上，CompletableFuture API 提供了一种简单的方法来创建具有多个步骤的异步任务流水线。通过手动创建和完成阶段，您可以对之后如何执行流水线进行精细控制。例如，如果已知结果，则可以避免启动任务。或者，您可以为常见任务创建部分流水线工厂。

让我们看看几个可能的用例。

### CompletableFuture 作为返回值

`CompletableFuture` 作为可能昂贵或长时间运行任务的出色返回值。

想象一个天气报告服务，调用 REST API 返回一个 `WeatherInfo` 对象。尽管天气随时间变化，但有意义的是在更新之前缓存某个地方的 `WeatherInfo` 一段时间。

REST 调用自然比简单的缓存查找更昂贵，并且需要更长时间，因此可能会阻塞当前线程太长时间，以至于无法接受。将其包装在 `CompletableFuture` 中提供了一种简单的方法，将任务从当前线程中卸载，导致以下通用的带有单一 `public` 方法的 `WeatherService`：

```java
public class WeatherService {

  public CompletableFuture<WeatherInfo> check(ZipCode zipCode) {
    return CompletableFuture.supplyAsync(
      () -> this.restAPI.getWeatherInfoFor(zipCode)
    );
  }
}
```

添加缓存需要两种方法，一种用于存储任何结果，另一种用于检索现有结果，如下所示：

```java
public class WeatherService {

  private Optional<WeatherInfo> cached(ZipCode zipCode) {
    // ...
  }

  private WeatherInfo storeInCache(WeatherInfo info) {
    // ...
  }

  // ...
}
```

使用 `Optional<WeatherInfo>` 为您提供了一个功能性的起点，以稍后连接每个部分。缓存机制的实际实现对示例的目的和意图并不重要。

实际的 API 调用也应进行重构，以创建更小的逻辑单元，从而导致一个公共方法和三个私有的独立操作。通过使用 `thenApply` 方法和 `storeInCache` 方法，可以将将结果存储在缓存中的逻辑添加为 `CompletableFuture` 操作：

```java
public class WeatherService {

  private Optional<WeatherInfo> cacheLookup(ZipCode zipCode) {
    // ...
  }

  private WeatherInfo storeInCache(WeatherInfo info) {
    // ...
  }

  private CompletableFuture<WeatherInfo> restCall(ZipCode zipCode) {

    Supplier<WeatherInfo> restCall = this.restAPI.getWeatherInfoFor(zipCode);

    return CompletableFuture.supplyAsync(restCall)
                            .thenApply(this::storeInCache);
  }

  public CompletableFuture<WeatherInfo> check(ZipCode zipCode) {
    // ...
  }
}
```

现在所有部分都可以组合起来完成提供缓存天气服务的任务，如示例 Example 13-8 所示。

##### 示例 13-8\. 使用 CompletableFutures 的缓存 WeatherService

```java
public class WeatherService {

  private Optional<WeatherInfo> cacheLookup(ZipCode zipCode) { ![1](img/1.png)
    // ...
  }

  private WeatherInfo storeInCache(WeatherInfo info) { ![1](img/1.png)
    // ...
  }

  private CompletableFuture<WeatherInfo> restCall(ZipCode zipCode) { ![2](img/2.png)

    Supplier<WeatherInfo> restCall = () -> this.restAPI.getWeatherInfoFor(zipCode);

    return CompletableFuture.supplyAsync(restCall)
                            .thenApply(this::storeInCache);
  }

  public CompletableFuture<WeatherInfo> check(ZipCode zipCode) { ![3](img/3.png)

    return cacheLookup(zipCode).map(CompletableFuture::completedFuture) ![4](img/4.png)
                               .orElseGet(() -> restCall(zipCode)); ![5](img/5.png)
  }
}
```

![1](img/#co_asynchronous_tasks_CO5-1)

缓存查找返回一个 `Optional<WeatherInfo>`，提供一个流畅和功能性的起点。 `storeInCache` 方法返回存储的 `WeatherInfo` 对象，可用作方法引用。

![2](img/#co_asynchronous_tasks_CO5-3)

`restCall` 方法将 REST 调用本身与成功完成后的结果存储在缓存中结合在一起。

![3](img/#co_asynchronous_tasks_CO5-4)

`check` 方法通过首先查找缓存来组合其他方法。

![4](img/#co_asynchronous_tasks_CO5-5)

如果找到 `WeatherInfo`，它会立即返回一个已完成的 `CompletableFuture<WeatherInfo>`。

![5](img/#co_asynchronous_tasks_CO5-6)

如果未找到 `WeahterInfo` 对象，则 Optional 的 `orElseGet` 懒执行 `reastCall` 方法。

将 CompletableFutures 与 Optionals 结合的优势在于，对于调用者来说，后台发生了什么并不重要，无论数据是通过 REST 加载还是直接来自缓存。每个 `private` 方法都以最高效的方式执行单一任务，而 `public` 方法只在绝对需要时将它们组合为异步任务管道执行昂贵的工作。

### 未决的 CompletableFuture 管道

未决的 CompletableFuture 实例永远不会自行完成任何状态。类似于流，直到连接终端操作，CompletableFuture 任务管道不会执行任何工作。因此，它作为更复杂任务管道的第一阶段提供了一个完美的起点，甚至是稍后按需执行的预定义任务的脚手架。

假设您想处理图像文件。涉及多个独立步骤可能失败。与直接处理文件不同，工厂提供未决的 `CompletedFuture` 实例，如 Example 13-9 所示。

##### Example 13-9\. 带有未决 CompletableFuture 的 ImageProcessor

```java
public class ImageProcessor {

  public record Task(CompletableFuture<Path> start, ![1](img/1.png)
                     CompletableFuture<InputStream> end) {
    // NO BODY
  }

  public Task createTask(int maxHeight,
                         int maxWidth,
                         boolean keepAspectRatio,
                         boolean trimWhitespace) {

    var start = new CompletableFuture<Path>(); ![2](img/2.png)

    var end = unsettled.thenApply(...) ![3](img/3.png)
                       .exceptionally(...)
                       .thenApply(...)
                       .handle(...);

    return new Task(start, end); ![4](img/4.png)
  }
}
```

![1](img/#co_asynchronous_tasks_CO6-1)

调用者需要访问未决的第一阶段来启动管道，但还需要阶段来访问最终结果。

![2](img/#co_asynchronous_tasks_CO6-2)

返回的 CompletableFuture 实例的通用类型必须与实际执行管道时调用者提供的类型匹配。在本例中，使用 `Path` 到图像文件。

![3](img/#co_asynchronous_tasks_CO6-3)

任务管道始于一个未决实例，以便可以懒加载添加所需的处理操作。

![4](img/#co_asynchronous_tasks_CO6-4)

`Task` 记录返回，以便轻松访问第一个和最后一个阶段。

运行任务管道是通过调用第一阶段 `start` 上的任意 `complete` 方法完成的。然后，最后一个阶段用于检索可能的结果，如下所示：

```java
// CREATING LAZY TASK
var task = this.imageProcessor.createTask(800, 600, false, true);

// RUNNING TASK
var path = Path.of("a-functional-approach-to-java/cover.png");
task.start().complete(path);

// ACCESSING THE RESULT
var processed = task.end().get();
```

就像没有终端操作的流管道会为多个项创建一个延迟处理管道一样，未决的 CompletableFuture 管道是一个延迟可用的单个项任务管道。

# 关于线程池和超时

并发编程的另外两个方面不容忽视：超时和线程池。

默认情况下，所有`-Async`的`CompletableFuture`操作使用 JDK 的公共`ForkJoinPool`。这是一个基于运行时设置具有合理默认值的高度优化的线程池¹。顾名思义，“common”池是一个共享池，也被 JDK 的其他部分如并行流使用。然而，与并行流不同，异步操作可以使用自定义的`Executor`。这使您可以使用适合您需求的线程池²，而不影响公共池。

# 守护线程

使用`ForkJoinPool`通过线程与通过`Executor`创建的用户线程之间的一个重要区别是它们能够比主线程生存更久。默认情况下，用户创建的线程是非守护线程，这意味着它们会比主线程更长时间地存活，并阻止 JVM 退出，即使主线程已完成所有工作。然而，通过`ForkJoinPool`使用线程可能会随着主线程的结束而被终止。有关该主题的更多详细信息，请参阅 Java 冠军 A N M Bazlur Rahman 的这篇[博文](https://bazlur.com/2021/07/be-sure-of-using-fork/join-common-pool-they-are-daemon-threads/)。

在最高效的线程上运行任务只是方程式的一半；考虑超时也是另一半。一个永不完成或超时的`CompletableFuture`将保持永远挂起，阻塞其线程。例如，如果尝试通过调用`get()`检索其值，则当前线程也会被阻塞。选择适当的超时可以防止永远阻塞的线程。然而，使用超时意味着现在也必须处理可能的`TimeoutException`。

提供多种操作，包括中间操作和终端操作，如表 13-4 所列。

表 13-4\. 与超时相关的操作

| 方法签名 | 用例 |
| --- | --- |
| `CompletableFuture<T> completeOnTimeout(T value, long timeout, TimeUnit unit)` | 在超时后使用提供的值正常完成阶段。（Java 9+） |
| `CompletableFuture<T> orTimeout(long timeout, TimeUnit unit)` | 在超时后异常地完成阶段。（Java 9+） |
| `T get(long timeout, TimeUnit unit)` | 阻塞当前线程直到计算结束。如果超时，则抛出`TimeoutException`异常。 |

中间操作`completeOnTimeout`和`orTimeout`提供了一种类似拦截器的操作，用于处理`CompletableFuture`管道中任何位置的超时情况。

超时的替代方案是通过调用`boolean cancel(boolean mayInterruptIfRunning)`取消正在运行的阶段。它取消一个未解决的阶段及其依赖项，因此可能需要一些协调和跟踪正在取消的正确阶段。

# 关于异步任务的最终思考

异步编程是并发编程的一个重要方面，以实现更好的性能和响应能力。然而，要理解异步代码执行的时间和在哪个线程上执行并不明显。

协调不同线程在 Java 中并不是什么新鲜事。如果你不习惯多线程编程，这可能是一件麻烦且难以高效完成的事情。这正是`CompletableFuture` API 的闪光之处。它将复杂的异步、可能是多步骤任务的创建与协调结合到一个广泛、一致且易于使用的 API 中。这使得你可以比以往更轻松地将异步编程纳入你的代码中。此外，你不需要通常与多线程编程相关的常见样板和“扶手”。

然而，就像所有编程技术一样，存在一个*最佳问题上下文*。如果不加区分地使用，异步任务可能会达不到预期的目标。

异步运行任务适用于以下任何情况：

+   许多任务需要同时进行，至少其中一个能够取得进展。

+   处理重 I/O、长时间运算、网络调用或任何类型的阻塞操作的任务。

+   任务大多数是独立的，不必等待另一个任务完成。

即使是像`CompletableFuture`这样的高级抽象，多线程代码也是以可能的效率为代价的简单交换。

像其他并发或并行高级 API 一样，比如我在第八章中讨论的并行流 API，协调多个线程涉及非明显的成本。应该有意选择这些 API 作为优化技术，而不是作为一种希望更有效地利用可用资源的一揽子解决方案。

如果你对如何安全地在多线程环境中导航的细节感兴趣，我推荐由 Oracle 的 Java 语言架构师 Brian Goetz³ 所著的 *Java Concurrency in Practice* 这本书。即使自 2006 年发布以来引入了所有新的并发功能，这本书仍然是该主题的事实参考手册。

# 要点

+   Java 5 引入了类型`Future<T>`作为异步任务的容器类型，具有最终结果。

+   CompletableFuture API 在提供了许多以前不可用的理想特性的基础上，改进了`Future<T>`类型。它是一个声明式的、反应式的、基于 Lambda 的协调 API，拥有 70 多个方法。

+   任务可以轻松地链式或合并成一个更复杂的管道，如果需要的话，每个任务都在新线程中运行。

+   异常是*头等公民*，你可以在函数流畅调用中恢复，不像 Streams API。

+   `CompletableFuture<T>`实例可以手动创建，无需任何线程或其他协调即可使用预先存在的值，或者作为挂起实例，以提供其附加操作的按需启动点。

+   由于`CompletableFuture` API 是一个并发工具，因此还需要考虑通常与并发相关的方面和问题，如超时和线程池。与并行流一样，异步运行任务应被视为一种优化技术，而不一定是首选选项。

¹ 通用的`ForkJoinPool`的默认设置及其如何更改的解释可在[其文档](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/ForkJoinPool.xhtml)中找到。

² 优秀的书籍*Java 并发实战*由 Josh Bloch 等人编写（ISBN 9780321349606），在*第二部分：第八章 应用线程池*中，提供了更好地理解线程池工作及其最佳应用的所有信息。

³ Goetz, Brian. 2006\. “Java Concurrency in Practice.” Addison-Wesley. ISBN 978-0321349606.
