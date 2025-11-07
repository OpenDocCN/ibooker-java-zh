# 附录 B. Java 8 中流的回顾

本附录是 Java 8 流和与之相关的基本函数式编程方面的复习。如果你不熟悉 Java 8 lambda 表达式的基本语法或其设计背后的哲学，你应该先阅读一本基础教材，熟悉这些概念，例如[*《现代 Java 实战：Lambda、Streams、函数式和响应式编程》，第 2 版*](https://livebook.manning.com/book/modern-java-in-action)，作者为 Raoul-Gabriel Urma、Mario Fusco 和 Alan Mycroft（Manning，2018）。Java 8 将 lambda 表达式作为*Project Lambda*的一部分引入，其总体目标可以总结如下：

+   允许开发者编写更干净、更简洁的代码。

+   为 Java Collections 库提供现代升级。

+   介绍一种抽象，它允许方便地使用基本功能习语。

在本附录中，我们讨论了 Collections 库的升级、默认方法和`Stream`抽象作为数据元素的函数式容器类型。

## B.1 向后兼容性

Java 平台最重要的概念之一是向后兼容性。指导哲学始终是，为早期版本的平台编写的或编译的代码必须继续与平台的后续版本一起工作。这一原则使开发者有更大的信心，即他们 Java 平台软件的升级不会影响当前正在运行的应用程序。

作为向后兼容性的结果，平台发展的方式存在限制，并且这些限制会影响开发者。

注意：为了保持向后兼容性，Java 平台可能不会在 JDK 中向现有接口添加额外的方法。

为了理解为什么是这样，考虑以下情况：如果某个接口`IFoo`的新版本在 Java 平台的版本 N 中添加了一个名为`newWithPlatformReleaseN()`的新方法，那么所有使用平台版本 N-1（或更早版本）编译的`IFoo`的先前实现将缺少这个新方法，这会导致在 Java 平台版本 N 下链接旧的`IFoo`实现失败。

这种限制对于 JDK 8 中 lambda 表达式的实现是一个严重的问题，因为其主要设计目标是能够将标准 JDK 数据结构升级以实现函数式编程学校的编码习语。意图是在 Java Collections 库的各个部分添加使用 lambda 表达式表达函数思想的新方法（如`map()`和`filter()`）。

## B.2 默认方法

为了解决这个问题，需要一个全新的机制。目标是允许通过添加*默认方法*来使用 Java 平台的新版本升级接口。

注意：从 Java 8 开始，任何接口都可以添加默认方法（有时称为可选方法）。这必须包括一个实现，称为*默认实现*，该实现直接在接口定义中编写。这一变化代表了接口定义的演变，并且不会破坏向后兼容性。

管理默认方法的规则如下：

+   接口实现*可能*（但不一定必须）实现默认方法。

+   如果实现类实现了默认方法，则使用类中的实现。

+   如果实现类没有实现默认方法，则使用接口定义中的默认实现。

让我们快速看一下一个例子。在 JDK 8 中添加到`List`的一个默认方法是`sort()`方法。其定义如下：

```
public default void sort(Comparator<? super E> c) {
    Collections.<E>sort(this, c);
}
```

这意味着任何`List`对象都有一个实例方法`sort()`，可以使用合适的`Comparator`就地排序列表。任何`List`的实现都可以提供自己的`sort()`行为覆盖，但如果它没有这样做，则默认实现（回退到`Collections`辅助类中提供的实现）将可用。

默认方法机制通过类加载工作。当一个接口的实现正在加载时，会检查类文件以查看是否所有可选方法都存在。如果存在，则类加载正常继续。如果不存在，则实现类的字节码会被修补以添加缺失方法的默认实现。

注意：默认方法代表了 Java 面向对象方法的基本变化。从 Java 8 开始，接口可以包含实现代码。许多开发者认为这放宽了 Java 严格单继承的一些规则。

开发者应该了解默认方法工作方式的一个细节：*默认实现冲突*的可能性，它有两个部分。首先，如果一个实现类已经有一个与新的默认方法同名和相同签名的现有方法，则始终会优先使用现有的实现，而不是默认实现。

其次，如果一个类实现了两个都包含同名和相同签名的默认方法的接口，该类必须实现该方法（并且可以选择将方法委托给接口的默认实现，或者完全执行其他操作）。这可能导致向接口添加默认方法会破坏客户端代码，因为如果客户端代码已经实现了另一个包含默认方法的接口，那么存在实现冲突的可能性。然而，在实践中，这种情况非常罕见，这种可能性被视为为了默认方法带来的其他好处而付出的微小代价。

## B.3 流

回想一下，Project Lambda 的目标之一是为 Java 语言提供轻松表达函数式编程技术的能力。例如，这意味着 Java 获得了编写`map()`和`filter()`惯用语的简单方法。

在 Java 8 的原始设计草图上，这些惯用语是通过直接将这些方法添加到经典 Java 集合接口中作为额外的默认方法来实现的。然而，这种方法有多个不满意的地方。

首先，因为`map()`和`filter()`是相对常见的名称，所以感觉现有实现的风险太高——许多用户编写的集合实现将会有不尊重新方法预期语义的现有方法。

相反，发明了一个新的抽象概念，称为`Stream`。`Stream`是一种容器类型，在某些方面与迭代器类似，用于更函数式的方法来处理集合和聚合数据。

`Stream`接口是放置所有新*函数式导向*方法的地方，例如`map()`、`filter()`、`reduce()`、`forEach()`和`flatMap()`。`Stream`上的方法广泛使用了函数式接口类型，例如 lambda 表达式作为参数。

将`Stream`视为一个可消费的元素序列。这意味着一旦从`Stream`中取出一个元素，它就不再可用，这与迭代器类似。

注意：由于`Stream`对象是可消费的，因此它们不应该被重用或存储在临时变量中。将`Stream`值分配给局部变量几乎总是代码的坏味道。

原始的集合类，如`List`和`Set`，被赋予了一个新的默认方法，称为`stream()`。这个方法为集合返回一个`Stream`对象，类似于在经典集合代码中使用`iterator()`的方式。

### B.3.1 示例

这段代码展示了我们如何使用`Stream`和 lambda 表达式来实现一个过滤惯用语：

```
List<String> myStrings = getSomeStrings();
String search = getSearchString();

System.out.println(myStrings.stream()
                            .filter(s -> s.equals(search))
                            .collect(Collectors.toList()));
```

注意，我们还需要调用`collect()`——这是因为`filter()`返回另一个`Stream`。在过滤操作之后，为了得到一个集合类型，我们需要做一些事情来主动将`Stream`转换为`Collection`。

整体方法看起来是这样的：

```
         stream()  filter()   map()   collect()
Collection -> Stream -> Stream -> Stream -> Collection
```

这个想法是让开发者构建一个需要应用到流中的“管道”操作。每个操作的实际上内容将使用每个操作的 lambda 表达式来表示。在管道的末端，结果需要被物化回一个集合中，因此使用了`collect()`方法。

让我们看看`Stream`接口定义的一部分（它定义了`map()`和`filter()`方法）：

```
public interface Stream<T> extends BaseStream<T, Stream<T>> {
    Stream<T> filter(Predicate<? super T> predicate);

    <R> Stream<R> map(Function<? super T, ? extends R> mapper);

    // ...
}
```

注意：不要担心那些看起来吓人的泛型定义。所有的“? super”和“? extends”子句意味着：当流中的对象有子类时，做正确的事情。

这些定义涉及两个新的接口：`Predicate`和`Function`。这两个接口都可以在`java.util.function`包中找到。这两个接口都只有一个方法，没有默认方法。因此，我们可以为它们编写 lambda 表达式，这些表达式将被自动转换为正确类型的实例。

注意：记住，当 Java 平台遇到 lambda 表达式时，总是将其转换为正确的*函数式接口*类型（通过类型推断）。

让我们看看一个代码示例。假设我们正在模拟海狸种群。有些是野生的，有些在野生动物园。我们想知道有多少圈养海狸是由培训动物园管理员照看的。使用 lambda 表达式和流，这很容易做到，如下所示：

```
Set<Otter> ots = getOtters();
System.out.println(ots.stream()
    .filter(o -> !o.isWild())
    .map(o -> o.getKeeper())
    .filter(k -> k.isTrainee())
    .collect(Collectors.toList())
    .size());
```

首先，我们过滤流，以便只处理圈养的海狸。然后，我们执行一个`map()`操作，以获取管理员流，而不是海狸流（注意，这个流的类型已从`Stream<Otter>`变为`Stream<Keeper>`）。然后，我们再次过滤，以选择仅培训管理员，然后使用静态方法`Collectors.toList()`将此转换为具体的集合实例。最后，我们使用熟悉的`size()`方法从具体的列表中返回计数。

在这个例子中，我们将我们的海狸转变成了负责它们的管理员。我们没有对任何海狸的状态进行变异以实现这一点——这有时被称为*无副作用*。

注意：在 Java 中，`map()`和`filter()`表达式中的代码应该是无副作用的。然而，Java 运行时并没有强制执行这个“规则”，所以请小心。你应该始终在自己的代码中遵循这个约定。

如果我们的用例意味着我们需要变异某些外部状态，我们可以使用两种方法之一，具体取决于我们想要实现什么。首先，如果我们想要构建聚合状态（例如，海狸年龄的运行总和），我们可以使用`reduce()`。或者，如果我们想要执行更通用的状态转换（例如，当旧管理员离开时将海狸转移到新管理员），则`forEach()`更合适。

让我们来看看如何使用下一代码片段中的`reduce()`方法计算海狸的平均年龄：

```
var kate = new Keeper();
var bob = new Keeper();
var splash = new Otter();
splash.incAge();
splash.setKeeper(kate);
Set<Otter> ots = Set.of(splash);

double aveAge = ((double) ots.stream()
    .map(o -> o.getAge())
    .reduce(0, (x, y) -> {return x + y;} )) / ots.size();
System.out.println("Average age: "+ aveAge);
```

首先，我们将海狸映射到它们的年龄。接下来，我们使用`reduce()`方法。它接受两个参数：初始值（通常称为*零*）和一个逐步应用的功能。在我们的例子中，这只是一个简单的加法，因为我们想要计算所有海狸的年龄总和。最后，我们将总年龄除以海狸的数量。

注意到`reduce()`的第二个参数是一个双参数 lambda。简单来说，这两个参数中的第一个是聚合操作的“运行总和”，第二个在遍历集合时实际上是循环变量。

最后，让我们转向我们想要改变状态的情况。为此，我们将使用`forEach()`操作。在我们的例子中，我们想要模拟 Keeper Kate 去度假的情况，因此现在应该将她的所有海獭交给 Bob。这可以轻松地完成如下：

```
ots.stream()
.filter(o -> !o.isWild())
.filter(o -> o.getKeeper().equals(kate))
.forEach(o -> o.setKeeper(bob));
```

注意，`reduce()`和`forEach()`都没有使用`collect()`。`reduce()`在遍历流的过程中收集状态，而`forEach()`只是将操作应用到流上的每个元素，所以在这两种情况下，都没有必要重新创建流。

## B.4 集合的局限性

Java 的集合类为该语言提供了极大的便利。然而，它们基于这样一个理念：集合中的所有元素都存在，并且它们在内存中的某个地方有表示。这意味着它们无法表示更通用的数据，例如无限集合。

例如，考虑所有质数的集合。我们不能将其建模为`Set<Integer>`，因为我们不知道所有的质数是什么，而且我们当然没有足够的堆空间来表示它们。在 Java 的早期版本中，这将是标准集合中一个非常难以解决的问题。

有可能构建一个主要使用迭代器并且将底层集合作为辅助角色的数据视图。然而，这需要纪律，并且不是 Java 集合的立即明显方法。在过去，如果开发者想要使用这种类型的方法，他们通常会依赖于提供更好功能支持的第三方库。

幸运的是，Java `Stream` 通过引入`Stream`接口作为更适合处理更通用数据结构的抽象来解决这个问题。这意味着`Stream`可以被视为比`Iterator`或`Collection`更通用。

注意：`Stream`不管理元素的存储，也不提供直接从流中访问单个元素的方法。

然而，`Stream` 并非真正的数据结构——相反，它是一种处理数据的抽象，尽管这两种情况之间的区别有些微妙。

## B.5 无限流

让我们更深入地探讨一下对无限序列数字建模的概念。以下是一些后果：

+   我们无法将整个流实体化为一个集合，因此像`collect()`这样的方法将不可行。

+   我们必须通过从流中拉取元素来操作。

+   我们需要一段代码，它能按需返回下一个元素。

这种方法还意味着表达式的值只有在需要时才会被计算。

在 Java 8 之前，表达式的值总是在它被绑定到变量或传递给函数时立即计算。这被称为*贪婪评估*，当然，这也是大多数主流编程语言中表达式评估的默认行为。

注意：从版本 8 开始，Java 引入了一种新的编程范式——`Stream`尽可能使用*延迟评估*。

这是一个非常强大的新功能，确实需要一些时间来适应。我们将在第十五章中更详细地讨论延迟评估。Java 中 lambda 表达式的目的是简化普通程序员的日常生活，即使这需要在平台上增加额外的复杂性。

## B.6 处理原始类型

我们之前一直忽略的一个 Stream API 的重要方面是如何处理原始类型。Java 的泛型不允许将原始类型用作类型参数，因此我们无法编写`Stream<int>`。幸运的是，`Streams`库提供了一些技巧来帮助我们解决这个问题。让我们看一个例子：

```
double totalAge = ((double) ots.stream()
                             .map(o -> o.getAge())
                             .reduce(0, (x, y) -> {return x + y;} ));

double aveAge = totalAge / ots.size();
System.out.println("Average age: "+ aveAge);
```

这实际上在大多数管道中使用了原始类型，所以让我们稍微展开一下，看看如何在像这样的代码中使用原始类型。

首先，不要被转换为`double`的操作弄混淆。这只是为了确保 Java 执行正确的平均计算，而不是执行整数除法。

`map()`的参数是一个 lambda 表达式，它接受一个`Otter`并返回一个`int`。如果我们能使用 Java 的泛型来编写它，lambda 表达式将被转换为实现`Function<Otter, int>`的对象。然而，由于 Java 的泛型不允许这样做，我们需要以不同的方式编码返回类型是`int`的事实——通过将其放入类型名称中，因此，实际上推断出的类型是`ToIntFunction<Otter>`。这种类型被称为函数类型的*原始特殊化*，用于避免在`int`和`Integer`之间的装箱和拆箱，从而节省不必要的对象生成，并允许我们使用特定于所使用原始类型的函数类型。

让我们更详细地分解平均计算。为了获取每只海狸的年龄，我们使用这个表达式：

```
ots.stream().map(o -> o.getAge())
```

让我们看看被调用的`map()`方法的定义，如下所示：

```
IntStream map(ToIntFunction<? super T> mapper);
```

从这里我们可以看到我们使用了特殊的函数类型`ToIntFunction`，我们还使用了一种特殊的`Stream`形式来表示整数的流。

之后，我们传递给`reduce()`，其定义如下：

```
int reduce(int identity, IntBinaryOperator op);
```

这也是一种专门的形式，它完全基于整数（ints）操作，并接受一个接受两个参数的 lambda 表达式（两个参数都是整数）来执行归约。

`reduce()`是一个收集操作（因此是急切的），所以管道在该点进行评估并返回一个单一值，然后将其转换为 double 并转换为整体平均值。

如果你错过了所有关于原始类型的细节，不要担心——这是类型推断的好处之一：大多数这些差异可以在大多数时候隐藏起来。

让我们通过讨论一个经常被开发者误解的主题来结束：对流上的并行操作的支持。

## B.7 并行操作？

在 Java 的旧版本（7 及之前），所有集合操作都是串行的。无论操作的集合有多大，都只会使用一个 CPU 核心来执行操作。随着数据集的增大，这可能会变得非常浪费，Project Lambda 的一个可能目标就是提升 Java 对集合的支持，以便高效地使用多核处理器。

注意：流式的惰性求值方法允许 lambda 表达式框架提供对并行操作的支持。

Stream API 的主要假设是创建流对象（无论是从集合还是通过其他方式）应该是低成本的，但管道中的某些操作可能很昂贵。这个假设使我们能够描述一个如下的并行管道：

```
s.stream()
    .parallel()
    // sequence of stream operations
    .collect( ... );
```

`parallel()` 方法将串行流转换为并行操作。其意图是允许普通开发者将 `parallel()` 作为透明并行化的入口点，并将提供并行支持的负担放在库编写者而不是最终用户身上。

理论上听起来不错，但在实践中，实现和其他细节最终会削弱 `parallel()` 机制的有用性。第十六章将更深入地讨论这一点。

由于这些限制，强烈建议您避免使用并行流，除非您能证明（使用第七章的方法）您的应用程序将从添加它们中受益。在实践中，作者们看到的情况中，实际有效的并行流案例不到六个。

符号

:clojure.spec.alpha/invalid: keyword 484

:shrunk key 491

[:smallest [(0)] key 491](../Text/14.htm#marker-1019874)

./gradlew command 377

./gradlew tasks meta-task 377

.gradle.kts extension 378

(.) macro 332

(agent) function 569

(alter) form 568

(apply) form 319

(cast) name 108

(concat) form 534

(conj <coll> <elt>) function 328

(conj) operation 557

(cons <elt> <coll>) function 328

(cons) operation 557

(constantly) function 330

(def <symbol> <value?>) special form 310

(def) form 310

(defmacro) special form 338

(defn) macro 305

(deref) form 311

(do <expr>*) special form 310

(do) form 309

(dosync) form 568

(every? <pred-fn> <coll>) function 328

(filter) call 535

(filter) form 323

(first <coll>) function 328

[(fn <name>? [<arg>*] <expr>*) special form 310](../Text/10.htm#marker-1033716)

(for) form 531

(future-done?) nonblocking call 564

(if <test> <then> <else>) special form 310

(if) special form 338

(import) form 332

(lazy-seq) form 534

(lazy-seq) macro 534

[(let [<binding>*] <expr>*) special form 310](../Text/10.htm#marker-1033731)

(let) form 317

(loop) form 320

(macroexpand-1) form 339

(macroexpand) form 339

(map) function 314

(new) form 332

(nth) function 313

(partial) form 535

(pcalls) function 564

(println) function 308

(proxy) form 334

(proxy) macro 333

(quote <form>) special form 310

(quote) form 311

(recur) form 320

(reduce) form 324

(ref) form 311, 565, 568

(rest <coll>) function 328

(send) call 570

(seq <coll>) function 328

(seq? <o>) function 328

(sort-by) new form 319

(take) form 534

(unless) form 336

(unless) function 337

(unless) macro 339

(var <symbol>) special form 310

(var) special form 311

(vec) form 313

(vector) form 313

(wait-and-log) call 570

@Container annotation 470

@file:JvmName(“AlternateClassName”) annotation 295

@FunctionalInterface annotation 146, 202

@JvmOverloads annotation 296

@Mojo annotation 373

@NotNull 注解 288

@Nullable 注解 288

@Test 注解 460

@Testcontainers 注解 469

&env 特殊变量 339

&form 特殊变量 339

#()形式 323 – 324

#define 预处理指令 336

#include 预处理指令 336

A

AbstractQueuedSynchronizer 类 173

访问控制 33 – 34, 588

访问器方法 236

Account 类 601

Account 接口 601

AccountManager 类 190

准确性 221 – 222

aconst_init 操作码 631

add 操作码 108

- -add-exports 开关 41, 43

- -add-host 标志 418

- -add-modules 开关 42

- -add-opens 命令行选项 45

Add-Opens JAR 清单属性 45

- -add-opens 开关 42

- -add-reads 开关 41

add()方法 186

额外的静态参数 590

代理 569 – 570

聚合模块 39

代数数据类型 73

*《算法设计手册》* (Skiena) 128

所有函数 279

分配器对象 617

aload_0 操作码 102

替代 JVM 语言

如何选择 262 – 265

使用该语言的开发者 264 – 265

学习语言难度 264

语言与 Java 良好交互 263 – 264

项目区域低风险 262 – 263

语言工具和测试支持 264

支持 JVM 265 – 268

编译器虚构 267 – 268

性能 266

非 Java 语言的运行时环境 266 – 267

语言动物学 252 – 256

动态类型与静态类型 253 – 254

命令式语言与函数式语言 254 – 255

解释性语言与编译性语言 252 – 253

重实现与原始版本 255 – 256

在 JVM 上进行多语言编程 256 – 262

竞争者 260 – 262

非 Java 语言，使用原因 257 – 259

新兴语言 259 – 260

Amdahl 定律 121 – 122

andThen() 方法 513

匿名类 607

任何函数 279

AOT（编译时）编译语言 234

API

Foreign Function 和 Memory API 612 – 618

在 Panama 处理本地内存 617 – 618

LibSPNG 示例 614 – 617

用支持的安全替换不安全 604 – 608

隐藏类 607 – 608

VarHandles 605 – 607

使用 BlockingQueue API 194 – 195

APM（应用程序性能监控） 432

AppClassLoader 类加载器 86

应用程序模块 38 – 39

应用程序插件 381

应用程序

使用 Docker 开发 Java 应用程序 411 – 424

使用 Gradle 构建镜像 412 – 414

在 Docker 中调试 421 – 423

使用 Docker Compose 进行本地开发 418 – 421

使用 Docker 记录日志 423 – 424

端口和主机 416 – 418

在 Docker 中运行构建 414 – 416

选择基础镜像 411 – 412

Gradle 构建 380 – 381

apply() 方法 513

为模块设计架构 46 – 52

Java 8 紧凑配置文件 47 – 49

多版本 JAR 文件 49 – 52

拆分包 47

算术

Clojure 315 – 316

操作码 107

函数的算术 329 – 330

AS Docker 命令 414

组装任务 388

asSequence() 函数 529

assertj 库 362

associateBy 函数 279

associateWith 函数 279

AST（抽象语法树） 308

astore 操作码 107

异步函数 556

async/await 方法 624

原子类 170 – 172

AtomicBoolean 类 170

AtomicInteger 类 170

AtomicLong 类 170

AtomicReference 类 170

Attach API 42

自动仪表化 213 – 214

自动模块 39 – 40

自动化

操作 346

静态分析 389 – 390

await() 方法 174

B

背压 189

向后兼容性 640 – 641

BadInit 类 90

裸机 402

屏障 174

基本插件 378

bean 风格 145

贝克，肯特 441, 443

BiFunction 接口 513

绑定 301

Bjarnason, Rúnar 516

块结构并发（Java 5 之前） 128 – 147

死锁 132 – 135

完全同步对象 131 – 132

免疫性 144 – 147

线程状态模型 130 – 131

同步

锁和 128 – 130

原因 135 – 137

线程状态和方法 137 – 143

废弃的线程方法 143

中断线程 140 – 142

处理异常和线程 142 – 143

volatile 关键字 137

BLOCKED 状态 131

BlockingQueue 接口 188, 194

BlockingQueue 模式 188 – 197

使用 BlockingQueue API 194 – 195

使用 WorkUnit 196 – 197

布尔参数 159

BootstrapClassLoader 类加载器 86

括号，在 Clojure 中 308 – 309

构建生命周期 350

<build><plugins> 部分 359

构建工具

Gradle 376 – 400

添加 Kotlin 387 – 388

自动化静态分析 389 – 390

构建 379 – 381

自定义 397 – 400

依赖关系 382 – 387

安装 376 – 377

超越 Java 8 390

脚本 378

任务 377 – 378

测试 388 – 389

使用插件 378 – 379

使用模块 391 – 397

避免工作 381 – 382

对开发者的重要性 345 – 350

自动化繁琐操作 346

确保开发者之间的一致性 349 – 350

管理依赖 346 – 349

Maven 350 – 376

添加另一种语言 355 – 357

编写 Maven 插件 372 – 376

构建生命周期 350 – 351

构建 353 – 354

控制清单 354

依赖管理 360 – 364

安装/POM 351 – 353

模块和 369 – 372

超越 Java 8 365 – 366

多版本 JAR 在 366 – 369

回顾 364 – 365

测试 357 – 360

- -build-cache 命令行标志 382

Builder 类 145

构建模块化应用 40 – 46

命令行开关 41 – 43

执行 43 – 45

反射和 45 – 46

buildscript 378

通过关键字 282, 526

通过记忆化调用 479

字节码 101 – 113

通过并发 149 – 168

死锁已解决，重新审视 162 – 166

重新审视死锁 160 – 162

丢失更新 151 – 153

字节码中的同步 153 – 157

同步方法 157 – 158

非同步读取 158 – 160

易失性访问 166 – 168

反汇编类 101 – 103

Kotlin 和 551 – 553

操作码（操作码）

算术 107

执行流程控制 108 – 109

调用 109 – 112

加载和存储 106 – 107

概述 105 – 106

平台操作 112

快捷形式 112 – 113

递归和 507 – 509, 522, 524

运行时环境 103 – 105

C

C 类 632

-c 开关 101

C1（客户端编译器） 235

C2（服务器编译器） 235

缓存未命中 222 – 224

缓存线程池 205

调用栈 104

调用目标 589

call() 方法 201, 268

可调用接口 201, 255, 584

CallSite 对象 589

取消函数 293

cancel() 方法 197

规范构造函数 63

容量 211

CAS（比较并交换）操作 171

Chiusano, Paul 516

类文件 82 – 86

字节码 101 – 113

反汇编类 101 – 103

操作码（操作码） 105 – 113

运行时环境 103 – 105

类加载和链接 83 – 85

初始化 85

准备 85

解析 85

验证 84 – 85

类对象 85 – 86

Classloader 类 86 – 95

自定义类加载 88 – 95

模块和类加载 95

检查 95 – 100

常量池 98 – 100

方法签名内部形式 96 – 98

javap 96

反射 113 – 118

结合类加载和 116 – 117

概述 114 – 116

问题 117 – 118

类关键字 281

类加载 82

类对象 83, 85 – 86, 129, 573

类类型 99

Class<?> 对象 573

Class<Integer> 类型 117

类，Kotlin 281 – 287

ClassFileParser::parseClassFile() 方法 92

ClassLoader 类 86 – 95

自定义类加载 88 – 95

依赖注入框架示例 93 – 94

异常，类加载 89 – 91

第一个自定义类加载器 91 – 93

类加载器示例 94 – 95

模块和类加载 95

clean lifecycle 351

-客户端切换 235

clj 命令 302

Clojure 260, 302

括号 308 – 309

并发 Clojure 557 – 570

代理 569 – 570

未来和 pcall 563 – 565

持久数据结构 557 – 563

软件事务内存 565 – 569

函数式编程和闭包 323 – 325

从 REPL 开始 303 – 305

Gradle 376 – 377

302 中的 Hello World – 303

在 Java 和 330 之间交互 – 335

从 Clojure 调用 Java 331 – 332

使用 REPL 进行探索性编程 334 – 335

Clojure 值的 Java 类型 332 – 333

Clojure 调用的本质 332

从 Java 使用 Clojure 335

使用 Clojure 代理 333 – 334

Kotlin 271

宏 335 – 341

犯错误 305 – 308

Maven 351 – 353

概述 300 – 302

使用 481 进行基于属性的测试 – 493

clojure.spec 483 – 486

clojure.spec 和 test.check 492 – 493

clojure.test 481 – 483

test.check 487 – 492

序列 325 – 330

语法和语义 309 – 323

算术、相等和其他操作 315 – 316

函数 316 – 319

列表、向量、映射和集合 312 – 315

循环 319 – 320

读取宏和调度 321 – 323

特殊形式 309 – 311

Testcontainers 467

Clojure 函数式编程 301, 325, 531 – 535

comprehensions 531 – 532

Clojure 中的柯里化 535

Kotlin 函数式编程 519 – 522

惰性序列 532 – 534

概述 498 – 499

*Clojure in Action* (Rathore) 299

clojure.lang 包 305

clojure.lang.ISeq 接口 325

clojure.lang.Ref 类型 568

clojure.lang.Var 类型 333

clojure.spec 483 – 486, 492 – 493

clojure.test 481 – 483

clojure.test 标准库 481

闭包 325

Clojure 323 – 325

Java 作为函数式编程语言的限制 509 – 511

Kotlin 函数式编程 517 – 518

概述 500 – 501

集群对象类型 425

CMD Docker 命令 408 – 410, 413 – 414

CMT (芯片多线程) 220

CNCF (云原生计算基金会) 433

Cognitect Labs 测试运行器测试运行器 483

科恩，迈克 438

collect() 方法 511, 643, 646

集合字面量 17

集合

Java 作为函数式编程语言的限制 514 – 515

Kotlin 277 – 279

限制 645 – 646

Collections 类 181

集合工厂（JEP 213） 17 – 18

集合辅助类 642

Collectors.toList() 静态方法 644

com.company.project 规范 37

com.oracle.graal.graal_enterprise 模块 38

命令行开关 41 – 43

通信开销 122

紧凑构造函数 67

紧凑配置文件（Java 8）47 – 49

紧凑记录构造函数 67 – 69

紧凑字符串 596 – 597

压缩 228

compareAndSwapInt() 方法 606

compareAndSwapLong() 方法 606

compareAndSwapObject() 方法 606

compareTo() 方法 539, 585

编译错误 288

编译日志 237 – 238

编译目标 355

编译阶段 350

compileClasspath Gradle 依赖配置 383 – 384

编译语言 252 – 253

compileOnly Gradle 依赖配置 384

编译器虚构 267 – 268

CompletableFuture 类 198 – 200, 499, 544 – 547

CompletableFuture.supplyAsync() 方法 200

CompletableFuture<T> 类型 199

组合函数 516

理解，Clojure 函数式编程 531 – 532

*《计算机体系结构：定量方法》（Hennessey 等著）220

并发

块结构并发（Java 5 之前）128 – 147

死锁 132 – 135

完全同步对象 131 – 132

免疫性 144 – 147

线程状态模型 130 – 131

同步和锁 128 – 130

同步，原因 135 – 137

线程状态和方法 137 – 143

volatile 关键字 137

设计力 124 – 128

冲突的力，原因 127

活系统 126

性能 126

可重用性 126 – 127

安全和并发类型安全 125 – 126

开销来源 127 – 128

Java 内存模型 (JMM) 147 – 149

Kotlin 291 – 294

理论入门 120 – 124

Amdahl 定律 121 – 122

硬件 121

Java 线程模型 123 – 124

经验教训 124

线程熟悉假设 120 – 121

通过字节码 149 – 168

死锁解决，重访 162 – 166

死锁重访 160 – 162

丢失更新 151 – 153

字节码中的同步 153 – 157

同步方法 157 – 158

未同步读取 158 – 160

volatile 访问 166 – 168

并发原语 170

并发编程

并发 Clojure 557 – 570

代理 569 – 570

futures 和 pcalls 563 – 565

持久数据结构 557 – 563

软件事务内存 565 – 569

F/J (Fork/Join) 框架 538 – 544

示例 539 – 542

并行化问题 542

工作窃取算法 543 – 544

FP (函数式编程) 和 544 – 549

CompletableFuture 类 544 – 547

并行流 547 – 549

Kotlin 协程 549 – 556

协程作用域和调度 554 – 556

协程如何工作 549 – 554

ConcurrentHashMap 类 176 – 185

并发 Dictionary 的方法 180 – 182

Dictionary 类的限制 179 – 180

简化版 HashMap 176 – 179

使用 182 – 185

并发类型安全 125

ConcurrentMap 接口 184

条件对象 173 – 174

配置参数，垃圾回收 232 – 233

conj 方法 557

cons 方法 557

共识屏障 174

一致性 349 – 350

常量池 98 – 100

CONSTANT_ 前缀 99

构造函数关键字 283

Consumer<> 接口 471

容器引擎 403

容器注册表 409

容器

Docker 406 – 411

构建 Docker 镜像 406 – 409

使用 Java 应用程序开发 411 – 424

运行 Docker 容器 409 – 411

收集容器日志 471

对于开发者的重要性 402 – 406

裸机 402

容器的好处 404 – 405

容器引擎 403

容器 403 – 404

容器缺点 405 – 406

宿主操作系统或 Type 1 虚拟机 402

Type 2 虚拟机 402 – 403

虚拟机 (VM) 403

Kubernetes 424 – 432

可观测性 432 – 435

性能 434 – 435

控制器 424

便利性和简洁性，Kotlin 271 – 281

集合 277 – 279

等价性 273

函数 273 – 277

if 表达式 279 – 281

从更少开始 271 – 272

变量 272 – 273

协作多任务处理 293

复制方法 519, 550

写时复制语义 185

CopyOnWriteArrayList 类 185 – 188

协程 291, 549

概述 549 – 554

作用域和调度 554 – 556

CoroutineScope 实例 555

成本，实现更高的性能 215 – 216

计数方法 455, 557

countDown() 方法 174

CountDownLatch 174 – 175

COWIterator 对象 186

create() 工厂方法 335

临界区 129

curried 函数 502

currying

在 Clojure 中 535

Java 作为函数式编程语言的限制 513

Kotlin FP 518

概述 502

自定义类加载 88 – 95

依赖注入框架示例 93 – 94

异常，类加载 89 – 91

第一个自定义类加载器 91 – 93

类加载器示例 94 – 95

D

DAG (有向无环图) 31

DAO (数据访问对象) 473

数据类构造 519

数据类，Kotlin 286 – 287

数据并行 547

*数据科学书坊* (Apeltsin) 212

数据类型 341

DaySupplier 类型 512

死锁 132 – 135, 160 – 166

在 Docker 中调试 421 – 423

解构模式 610

定义特殊形式 301

默认子句 60

默认实现 641

默认实现冲突 642

默认生命周期 350

默认行 59

默认方法 641 – 642

defineAnonymousClass() 方法 607

defineClass() 方法 92

defineClass1() 原生方法 92

defspec 函数 491

deftest 函数 482

退化 211

延迟函数 550

委托属性 479, 526

委托 180

去优化，即时编译 (JIT) 238

依赖 346 – 349

依赖冲突 348 – 349

在 Gradle 中 382 – 387

Maven 360 – 364

依赖配置 382

依赖冲突 348

依赖注入框架示例 93 – 94

依赖解析 361

部署阶段 351

部署对象类型 425

已弃用的线程方法 143

描述方法 480

设计力量，并发 124 – 128

冲突，原因 127

实时系统 126

性能 126

可重用性 126 – 127

安全和并发类型安全 125 – 126

开销来源 127 – 128

destroy() 方法 143

解构 610

开发者

构建工具 345 – 350

自动化繁琐操作 346

确保开发者之间的一致性 349 – 350

管理依赖 346 – 349

容器，其重要性 402 – 406

裸金属机器 402

容器的优点 404 – 405

容器引擎 403

容器 403 – 404

容器的缺点 405 – 406

宿主操作系统或 Type 1 虚拟机 402

Type 2 虚拟机 402 – 403

虚拟机 (VM) 403

使用语言 264 – 265

DI (依赖注入) 81

字典类 176

并发方法的途径 180 – 182

局限性 179 – 180

直接测量 213 – 214

不相交联合类型 73

Clojure 中的调度读取宏 321 – 323

调度，JVM 中的方法 572

调度，协程 554 – 556

可区分性 222

分布式跟踪可观察性支柱 433

div 操作码 108

分而治之模式 538

dload 操作码 106

do-while 循环 603

doc 函数 485

Docker 406 – 411

构建 Docker 镜像 406 – 409

使用 411 开发 Java 应用程序 – 424

使用 Gradle 构建镜像 412 – 414

在 Docker 中调试 421 – 423

使用 Docker Compose 进行本地开发 418 – 421

使用 Docker 记录日志 423 – 424

端口和主机 416 – 418

在 Docker 中运行构建 414 – 416

选择基础镜像 411 – 412

运行 Docker 容器 409 – 411

docker build 命令 406

docker 命令 423

Docker Compose 418 – 421

docker cp 命令 422

docker pull 命令 409

docker push 命令 409

docker run 命令 409

- -dry-run 标志 378

DSL (领域特定语言) 376, 378

鸭子类型 13

鸭子变量 13

哑对象 449 – 451

dup 操作码 103, 106

动态编译

单态调用和 236 – 237

原因 234

动态类型 253 – 254

E

-e 标志 410

贪婪评估 501, 646

简单情况是简单反模式 542

echoserver 容器 426

有效最终变量 510

效率 211

EJBs (企业 JavaBeans) 619

else 情况 281

else 条件 337

else 位置 320

涌现属性 208

端到端测试 440, 474 – 476

企业模块（JEP 320） 18 – 19

entry() 方法 18

entrySet() 方法 179

枚举关键字 73

环境 410, 510

临时端口 417

平等

Clojure 315 – 316

Kotlin 273

equals() 方法 64

人体工程学 435

逃逸分析 628

评估堆栈 104

EvilOrder 类 70

异常 142 – 143

execute() 方法 543

执行，模块化应用程序 43 – 45

执行控制操作码 108 – 109

执行步骤 367

executions 元素 374

执行器参数 546

执行器接口 202

Executors 类 202

Executors.newVirtualThreadPerTaskExecutor() 工厂方法 625

ExecutorService 接口 202

ExecutorService.invokeAll() 辅助方法 564

完备性 70

ExpectedException 规则 464

解释函数 485

探索性编程 303, 334 – 335

导出关键字 36 – 37

导出关键字 33

EXPOSE 端口 Docker 命令 417

表达式 59

扩展关键字 285

扩展对象 399

扩展属性 399

ExternalResource 类 461

F

F/J (Fork/Join) 框架 538 – 544

示例 539 – 542

针对 542 的并行化问题

工作窃取算法 543 – 544

工厂方法 144

编写失败的测试 443 – 445

安全故障插件 459

伪造对象 449, 454 – 456

false 值，Clojure 316

Feather, Michael 442

feed() 方法 115

字段对象 85

Fieldref 100

files 函数 392

filter 集合 516

filter 函数技术 531

filter() 方法 511, 641 – 644

filter() 管道 511

final 关键字 146, 171, 498

final 方法 578

final 引用 504

final var 组合 519

findClass() 方法 88

findConstructor() 方法 584

findVirtual() 方法 584

有限资源 402

fire-and-forget 虚拟线程 622

一等函数 254, 268, 275

first() 方法 335

固定线程池 204 – 205

 fixtures 478

flatMap() 方法 546, 643

飞行记录器 239 – 240

浮点数 100

for 循环 278, 319, 604

forEach() 方法 643

forEach() 操作 645

Foreign Function API 612 – 618

在 Panama 处理原生内存 617 – 618

LibSPNG 示例 614 – 617

ForkJoinPool 执行器服务 538

ForkJoinTask 类型 543

form 300

format() 方法 12

FP (函数式编程)

Clojure 323 – 325

Clojure FP 531 – 535

comprehensions 531 – 532

Clojure 中的 currying 535

惰性序列 532 – 534

概念 498 – 502

闭包 500 – 501

柯里化和部分应用 502

高阶函数 499 – 500

不可变性 498 – 499

懒惰 501

纯函数 498

递归 500

并发编程和 544 – 549

CompletableFuture 类 544 – 547

并行流 547 – 549

Java 作为 FP 语言的限制 502 – 515

闭包 509 – 511

柯里化和部分应用 513

高阶函数 505 – 506

Java 类型系统和集合 514 – 515

懒惰 511 – 512

可变性 503 – 505

纯函数 503

递归 506 – 509

Kotlin FP 515 – 530

闭包 517 – 518

柯里化和部分应用 518

不可变性 519 – 522

惰性求值 525 – 526

纯和高阶函数 516 – 517

序列 526 – 530

尾递归 522 – 525

FROM Docker 命令 409, 414

完整集合 228

完全同步对象 131 – 132

函数接口 644

函数对象 513

函数类型 506

函数接口类型 644

函数式语言 254 – 255

*Kotlin 中的函数式编程* (Vermeulen, Bjarnason, and Chiusano) 516

面向函数的方法 642

函数 498

Clojure 316 – 319

Kotlin 273 – 277

Fusco, Mario 640

Future 接口 197 – 200

futures，在 Clojure 中 563 – 565

FutureTask 类 200 – 201

模糊测试 489

FX (外汇货币交易) 56

G

G1 收集 229 – 231

GA (一般可用性) 434

垃圾回收 224 – 233

内存区域 227

基础 225

配置参数 232 – 233

完整集合 228

G1 229 – 231

标记和清除 225 – 227

并行收集器 231 – 232

safepoints 228

年轻集合 228

GAV (组，工件，版本) 坐标 352

gen 函数 493

生成函数 488

代际垃圾回收 226

生成器包 488

泛型特化 633

GenericContainer 对象 470

泛型 633

get() 方法 145, 178, 197, 199, 313, 512

getAndAddInt() 方法 602

getAndAddLong() 方法 606

getAndIncrement() 方法 171

getAndSetInt() 方法 606

getAndSetLong() 方法 606

getAndSetObject() 方法 606

getCallerClass() 607

getClass() 方法 334

getDeclaredMethod() 方法 117

getfield 指令码 106, 168

getInstance() 方法 237

getMethod() 方法 117

getstatic 指令码 106

getSuperclass() 方法 85

目标 351

Goetz, Brian 168

黄金锤技术 117

goto 指令码 108 – 109

Gough, James 246

GraalVM 261

Gradle 376 – 400

添加 Kotlin 387 – 388

自动化静态分析 389 – 390

构建 379 – 381

使用 412 构建图像 – 414

自定义 397 – 400

创建自定义插件 399 – 400

自定义任务 397 – 399

依赖关系 382 – 387

安装 376 – 377

超越 Java 8 390

脚本 378

任务 377 – 378

测试 388 – 389

使用插件 378 – 379

与模块一起使用 391 – 397

JLink 393 – 397

模块化应用 392 – 393

模块化库 391 – 392

避免工作 381 – 382

gradlew 包装器 377

gradlew.bat 命令 377

粒度 222

绿色线程 619

groom() 方法 575

Groovy

Kotlin 与 378

概述 260

分组构造 478

分组函数 478

分组 lambda 479

分组方法 480

守卫模式 77

H

Happens-Before 147

硬件 121

hash() 辅助方法 178

hashCode() 方法 60, 64

HashMap 111, 176 – 179

头部阻塞 19 – 20

头部阻塞，HTTP 21

堆 225

堆扁平化 628

Clojure 中的 Hello World 302 – 303

hi 符号 310

hi 变量 310

隐藏类 607 – 608

高阶函数

Java 作为函数式编程语言的限制 505 – 506

Kotlin 函数式编程 516 – 517

概述 499 – 500

同构语言 336

主机操作系统 402

主机，Docker 416 – 418

HotSpot 235 – 236

C1 (客户端编译器) 235

C2 (服务器编译器) 235

实时 Java 235 – 236

HTTP/2 19 – 24

头部阻塞 19 – 20

HTTP 头部性能 21

Java 11 22 – 24

其他考虑因素 22

受限连接 20 – 21

TLS 21 – 22

HttpClient 类型 22

HttpRequest 类型 22

HttpResponse.BodyHandler<T> 接口 23

巨无霸区域 230

I

IaaS (基础设施即服务) 405

Idc 指令 106

无身份引用 631

if 条件 508

if 表达式 279 – 281

if 指令 108

if 语句 279, 448

IFn 接口 307, 584

IFoo 接口 641

- -illegal-access 命令行开关 42, 45

ILP (指令级并行) 220

图片

构建 Docker 406 – 409

使用 Gradle 构建 412 – 414

选择基础 411 – 412

命令式语言 254 – 255

实现配置 383

导入语句 50

in 关键字 280

孵化特性 15 – 16

indexFor() 辅助方法 178

无限流 646

初始化块，Kotlin 283

<init> 方法 103

初始化阶段 85

内联类型 628

内联方法 234, 236

插入方法 455

实例变量 100

安装阶段 351

instanceof 关键字 74 – 75, 630

类加载器示例 94 – 95

Int 类型 288

Integer 类型 73, 100, 563, 626

集成测试 466 – 476

使用 Selenium 进行端到端测试示例 474 – 476

收集容器日志 471

安装 testcontainers 467

Postgres，示例 472 – 473

Redis，示例 467 – 471

接口方法 575 – 576

InterfaceMethodref 100

内部结构

封装 30

invokedynamic 589 – 593

方法句柄 583 – 589

查找 585 – 586

MethodHandle 583 – 584

MethodType 584 – 585

反射 vs. 代理 vs. 586 – 589

方法调用 571 – 578

最终方法 578

接口方法 575 – 576

特殊方法 576 – 577

虚拟方法 572 – 575

保护 32 – 33

反射内部 578 – 583

小的内部更改 593 – 600

紧凑字符串 596 – 597

嵌套成员 597 – 600

字符串连接 593 – 595

不安全

概述 600 – 604

使用支持的 API 替换 604 – 608

与 Java 的互操作性

Kotlin 294 – 297

语言 263 – 264

使用 Clojure 330 – 335

从 Clojure 调用 Java 331 – 332

使用 REPL 进行探索性编程 334 – 335

Clojure 值的 Java 类型 332 – 333

Clojure 调用的本质 332

从 Java 使用 Clojure 335

使用 Clojure 代理 333 – 334

解释性语言 252 – 253

中断线程 140 – 142

*算法导论* (Cormen 等人) 128

调用操作码 109 – 112

调用函数 275

调用方法 584

invoke 操作码 102, 105, 110, 158

调用()方法 115, 307, 543, 578

InvokeDynamic 100

invokedynamic 64, 589 – 593

invokedynamic 操作码 109, 267, 588

invokeExact() 方法 586

invokeinterface 操作码 109, 111

invokespecial 操作码 102 – 103, 109, 578

调用静态操作码 109, 507

invokeSuspend 方法 552

invokevirtual 操作码 103, 109, 572, 574

调用方法 571 – 578

final 方法 578

接口方法 575 – 576

特殊方法 576 – 577

虚拟方法 572 – 575

is 操作符 290

isDone() 方法 197, 199

ISeq 接口 328

isInterrupted() 方法 141

地峡 612

it 方法 480

iterator() 方法 186, 334

J

JAR 工具 51

JARs, 多版本 49 – 52

构建示例 49 – 52

在 Maven 中 366 – 369

Java

Java 11 中的变化 17 – 25

集合工厂 (JEP 213) 17 – 18

HTTP/2 (Java 11) 19 – 24

企业模块的移除 (JEP 320) 18 – 19

单文件源代码程序（JEP 330）24 – 25

Clojure 与 330 的互操作性 – 335

从 Clojure 调用 Java 331 – 332

使用 REPL 进行探索性编程 334 – 335

Clojure 值的 Java 类型 332 – 333

Clojure 调用的本质 332

从 Java 使用 Clojure 335

使用 Clojure 代理 333 – 334

使用 Docker 开发应用程序 411 – 424

使用 Gradle 构建镜像 412 – 414

在 Docker 中调试 421 – 423

使用 Docker Compose 进行本地开发 418 – 421

使用 Docker 进行日志记录 423 – 424

端口和主机 416 – 418

在 Docker 中运行构建 414 – 416

选择基础镜像 411 – 412

增强类型推断（var 关键字）9 – 13

免费使用 637 – 638

语言和平台功能 13 – 16

更改语言 14 – 15

定义 4 – 6

孵化功能和预览功能 15 – 16

JSRs 和 JEPs 15

语法糖 14

作为函数式编程语言的局限性 502 – 515

闭包 509 – 511

柯里化和部分应用 513

高阶函数 505 – 506

Java 类型系统和集合 514 – 515

惰性 511 – 512

可变性 503 – 505

纯函数 503

递归 506 – 509

新的 Java 发布模型 6 – 9

付费支持选项 639

项目琥珀 610 – 612

项目 loom 618 – 625

使用虚拟线程编程 623 – 625

发布 625

线程构建类 622 – 623

虚拟线程 621 – 622

项目巴拿马 612 – 618

项目瓦尔哈拉 626 – 633

更改语言模型 630

值对象的影响 630 – 633

泛型 633

严格的语法 258 – 259

线程模型 123 – 124

Java 8

向后兼容性 640 – 641

紧凑配置文件 47 – 49

默认方法 641 – 642

Gradle 和 390

处理原始数据 646 – 647

无限流 646

集合的限制 645 – 646

Maven 和 365 – 366

并行操作 648

流 642 – 645

Java 11 17 – 25

集合工厂（JEP 213）17 – 18

HTTP/2（Java 11）19 – 24

企业模块的移除（JEP 320）18 – 19

单文件源代码程序（JEP 330）24 – 25

Java 17

instanceof 运算符 74 – 75

模式匹配和预览功能 75 – 77

记录 60 – 69

紧凑记录构造函数 67 – 69

名义类型 66 – 67

密封类型 69 – 74

Switch 表达式 57 – 60

文本块 55 – 57

Java 18 633 – 634

Java 代理 94

Java 命令 354, 396

*Java 并发实践* (Goetz) 168

Java EE (Java 企业版) 18

Java 语言 4

*Java 模块系统，Java 模块系统* (Parlog) 116

Java 模块

为模块设计 46 – 52

Java 8 紧凑配置文件 47 – 49

多版本 JARs 49 – 52

分割包 47

基本语法 34 – 37

导出和需求 36 – 37

传递性 37

构建第一个模块化应用 40 – 46

命令行开关 41 – 43

执行 43 – 45

反射和 45 – 46

加载 37 – 40

应用程序模块 39

自动模块 39 – 40

平台模块 38 – 39

未命名模块 40

概述 27 – 34

模块图 31 – 32

新的访问控制语义 33 – 34

Project Jigsaw 28 – 31

保护内部结构 32 – 33

Java 性能调优 220 – 224

缓存未命中 222 – 224

时间在 221 中的作用 – 222

精度 221 – 222

粒度 222

测量 222

网络分布式定时 222

精度 221

java 进程 410

Java SE (Java 标准版) 18, 637 – 638

Java SE 8 638

Java SE 11 638

Java SE 17 (LTS) 638 – 639

Java Streams 地址 646

java-library 插件 380

java.base 模块 31, 581

java.io 包 39

java.lang 包 39

java.lang: IdentityObject 接口 630

java.lang.Enum 库类型 70

java.lang.management 包 49

java.lang.Record 类 64

java.lang.reflect 包 115

java.lang.reflect.Method 类 115, 578

java.lang.String 对象 310

java.lang.Thread 类 137, 200

java.lang.Thread 子类 622

java.lang.Void 类型 117

java.library.path 系统属性 614

java.net.http 模块 16, 41

java.se 模块 39

java.time API 499

java.util 集合接口 277

java.util 包 39, 334

java.util.concurrent 包 169

java.util.concurrent.atomic 包 170

java.util.concurrent.Flow 发布者 23

java.util.function 接口 511

javap 编译器 506

javac 源代码编译器 11

javap 95 – 96

jcmd 240

JCP (Java 社区进程) 15

JDK 并发库

原子类 170 – 172

BlockingQueue 模式 188 – 197

使用 BlockingQueue API 194 – 195

使用 WorkUnit 196 – 197

现代并发应用程序的构建块 169 – 170

ConcurrentHashMap 类 176 – 185

并发字典的方法 180 – 182

Dictionary 类的限制 179 – 180

简化 HashMap 176 – 179

使用 182 – 185

CopyOnWriteArrayList 类 185 – 188

CountDownLatch 174 – 175

Future 类型 197 – 200

锁类 172 – 174

任务和执行 200 – 206

缓存线程池 205

Executor 接口 202

固定线程池 204 – 205

任务建模 200 – 201

单线程执行器 203

STPE (ScheduledThreadPoolExecutor) 205 – 206

JDK Flight Recorder 238 – 246

Flight Recorder 239 – 240

Mission Control 240 – 246

jdk.attach 模块 43

jdk.incubator 命名空间 16

jdk.incubator.foreign 模块 613

jdk.internal.jvmstat 模块 43

jdk.unsupported 模块 600

JEPs (JDK Enhancement Proposal) 15

jextract 工具 613

JIMAGE 格式 28

jimage 工具 29

JIT (即时) 编译 233 – 238

去优化 238

动态编译

单态调用和 236 – 237

原因 234

HotSpot 235 – 236

C1 (客户端编译器) 235

C2 (服务器编译器) 235

实时 Java 235 – 236

内联方法 236

读取编译日志 237 – 238

jlink 工具 29, 52, 393 – 397

JLS (Java 语言规范) 4, 147

JMC (JDK 任务控制) 240, 423

JMM (Java 内存模型) 120, 147 – 149, 601

jmod 命令 36

JMOD 格式 28

JNI (Java 本地接口) 15, 612

join() 调用 139

Clojure 的乐趣 (Fogus 和 Houser) 299

JPMS (Java 平台模块) 26

JRE (Java 运行时库) 348

JRuby 语言 255

jshell 交互环境 12, 623

jsr 指令 (已弃用) 108

JSRs (Java 规范请求) 15

JUnit 4 和 5 459 – 464

JUnit in Action (Tudose) 443

junit-vintage-engine 依赖项 460

JVM (Java 的虚拟机) 30 – 31, 571

JVM_DefineClass() 本地方法 83

JVM_DefineClassWithSource() C 函数 92

JVMTI (JVM 工具接口) 213, 607

Jython 语言 255

K

K8s (Kubernetes) 424

key-fn 函数 318

keying 函数 318

keys 函数 485

keySet() 方法 179

klass 573

唐纳德·E·克努特 216

Kotlin 259 – 260

并发 291 – 294

便利性和简洁性 271 – 281

集合 277 – 279

等价性 273

函数 273 – 277

if 表达式 279 – 281

从更少开始 271 – 272

变量 272 – 273

协程 549 – 556

协程作用域和调度 554 – 556

协程的工作原理 549 – 554

对类和对象的另一种看法 281 – 287

Gradle 添加 387 – 388

Groovy 与 378

安装 271

Java 兼容性 294 – 297

使用原因 271

安全性 287 – 291

空安全 288 – 289

智能转换 289 – 291

使用 476 进行规范式测试 – 481

Kotlin 函数式编程 515 – 530

闭包 517 – 518

柯里化和部分应用 518

不可变性 519 – 522

惰性求值 525 – 526

纯函数和一阶函数 516 – 517

序列 526 – 530

尾递归 522 – 525

kotlin 交互式 shell 271

kotlin-coroutine-core 库 291

kotlin-stdlib 库 297, 388

kotlin.collections 接口 277

kotlin.collections 包 277

kotlin.collections.List 接口 520

kotlin.collections.Map 接口 520

kotlinc 命令行编译器 271

kotlinx.collections.immutable library 521

Kt class 294

kubectl command 425 – 427

Kubernetes 424 – 432

L

L-type descriptors 632

lambda expressions 275, 591 – 593

LambdaMetafactory.metafactory() method 592

language features, Java 13 – 16

changing language 14 – 15

defined 4 – 6

incubating and preview features 15 – 16

instanceof operator 74 – 75

JSRs and JEPs 15

Pattern Matching and preview features 75 – 77

Records 60 – 69

compact record constructors 67 – 69

nominal typing 66 – 67

sealed types 69 – 74

Switch Expressions 57 – 60

syntactic sugar 14

Text Blocks 55 – 57

latches 174

lateinit pattern 479

latency 210

launch method 292, 552

layers 408

laziness

Clojure FP 532 – 534

Java limitations as FP language 511 – 512

Kotlin FP 525 – 526

overview 501

lazy evaluation 501, 646

lazy() function 525

Lazy<T> interface 525 – 526

LazySeq lazy implementation 333

LazyThreadSafetyMode enumeration 526

ldc (load constant) opcode 594

learning languages 264

library module 39

LibSPNG 614 – 617

生命周期方法 202

LIMIT 实例 70

LinkageError 类 91

Lisp (大量令人烦恼的愚蠢括号) 308

列表推导式 531

列表接口 313, 641, 643

列表对象 641

- -list-modules 标志 38

list-modules 开关 41

listOf 只读辅助函数 520

列表，Clojure 312 – 315

实时系统，并发 126

加载操作码 106 – 107

loadClass() 方法 88

加载和链接，类 83 – 85

通过 214 自动仪器化

与反射结合 116 – 117

初始化 85

准备 85

解析 85

验证 84 – 85

加载 Java 模块 37 – 40

应用程序模块 39

自动模块 39 – 40

平台模块 38 – 39

未命名的模块 40

使用 Docker Compose 的本地开发 418 – 421

LocalDate 类 512

锁类 172 – 174

锁对象 172 – 173

锁条带化 183

log4j2 库 424

可记录性守卫 216

使用 Docker 记录 423 – 424

日志支柱 433

Long 100

lookupClass() 表达式 588

lookupswitch 操作码 108

循环，Clojure 319 – 320

丢失更新 151 – 153

低暂停收集器 229

低风险项目区域 262 – 263

lreturn 操作码 522

LVTI (局部变量类型推断) 10

M

M:1 线程 619

宏展开时间 336

宏，Clojure 304, 335 – 341

main 方法 87, 296, 354, 593

主线开发模型 6

清单，Maven 354

手动仪器 213

Map 集合 516

Map 接口 111, 184

Map.Entry 接口 177

map()方法 314, 511, 528, 641 – 643, 647

Map，Clojure 312 – 315

标记和清除 225 – 227

Maven 350 – 376

添加另一种语言 355 – 357

编写 Maven 插件 372 – 376

构建生命周期 350 – 351

构建 353 – 354

控制清单 354

依赖管理 360 – 364

安装和 POM (项目对象模型) 351 – 353

模块和 369 – 372

模块化应用程序 370 – 372

模块化库 369 – 370

超越 Java 8 365 – 366

多版本 JARs 在 366 – 369

审查 364 – 365

测试 357 – 360

Maven 目标 373

maven 编译器插件 367

maven-failsafe-plugin 359

maven-publish 插件 392

maven-surefire-plugin 359

测量，性能分析 212 – 217

实现更高性能的成本 215 – 216

过早优化的危险 216 – 217

如何进行 213 – 214

通过类加载自动仪器化 214

直接测量 213 – 214

性能目标 214 – 215

时间和 222

要测量什么 212 – 213

何时停止 215

megamorphic 582

memoized 479

内存

领域 227

在 Panama 中处理原生内存 617 – 618

延迟层次结构 219 – 220

软件事务内存 565 – 569

Memory API 612 – 618

在 Panama 中处理原生内存 617 – 618

LibSPNG 示例 614 – 617

MemoryAddress 类 613

MemorySegment 接口 613, 617

MergeSort 算法 539

Meszaros, Gerard 449

metafactories 592

Method 类 579

方法句柄 583 – 589

查找 585 – 586

MethodHandle 583 – 584

MethodType 584 – 585

反射与代理的比较 586 – 589

Method Handles API 583

方法调用 571 – 578

final 方法 578

接口方法 575 – 576

特殊方法 576 – 577

虚拟方法 572 – 575

方法对象 85, 115, 579

方法签名 96 – 98

MethodAccessor 对象 580

MethodHandle 100, 592

MethodHandles.Lookup 类型 589

MethodHandles.lookup() 方法 584

Methodref 100

方法 498

MethodType 100, 584 – 585, 589

度量支柱 433

minikube 命令 425

任务控制 240 – 246

错误，在 Clojure 中 305 – 308

混合集合 230

模拟对象 449, 456 – 457

mock() 方法 457

模拟 457 – 458

建模任务 200 – 201

Callable 接口 201

FutureTask 类 201

*《现代 Java 动作，第 2 版》（Urma, Fusco, Mycroft）640

模块化应用程序

Gradle 392 – 393

Maven 和 370 – 372

模块化 Java 运行时 28 – 29

模块化库

Gradle 391 – 392

Maven 和 369 – 370

模块图 31 – 32

模块关键字 35

模块名称 36

模块路径 38

模块解析 95

module-info.java 声明 394

module-info.java 模块描述符 614

- -module-path 开关 41

ModuleDeclaration 生成式 35

ModuleDescriptor 类型 116

ModuleDirective 生成式 35

模块 27

类加载和 95

Gradle

JLink 393 – 397

模块化应用 392 – 393

模块化库 391 – 392

Gradle 和 391 – 397

Maven 和 369 – 372

模块化应用 370 – 372

模块化库 369 – 370

monitorenter 操作码 112, 154, 157

monitorexit 操作码 112, 154, 157

单态调用 235 – 237

摩尔定律 217 – 219

mul 操作码 108

多方法 341

多版本 JAR 49 – 52

构建示例 49 – 52

在 Maven 中 366 – 369

多阶段构建 414

可变性 503 – 505

默认可变 301

可变状态 304

mvn dependency:tree 命令 362

Mycroft, Alan 640

N

- -name container-name 参数 421

NameAndType 100

命名元组 66

本地方法 612

自然数 531

负缓存 90

巢居者 597, 600

网络分布式定时 222

new 关键字 281

新名称 112

new 操作码 103

NEW 线程状态 130

newCondition() 方法 174

newFixedThreadPoolContext 函数 555

Newland, Chris 246

newWithPlatformReleaseN() 方法 641

next() 方法 335

节点 558

名义类型 66 – 67

非 JVM 语言 261 – 262

不可表示的类型 12

非确定性暂停 226 – 227

无函数 279

不可重复读 160

NOP (无操作) 219

NoSuchMethodError 异常 349

notify() 方法 630

NullPointerException (NPE) 288, 631

O

Object 类 114, 630

对象头 573

objectFieldOffset() 方法 602

对象

Kotlin 281 – 287

Kubernetes 424

可观察性，容器 432 – 435

ofEntries() 方法 18

offer() 方法 194

ofPlatform() 方法 622

ofVirtual() 方法 622

OO (面向对象) 语言 254

指令集（操作码）

算术 107

执行流程控制 108 – 109

调用 109 – 112

加载和存储 106 – 107

概述 105 – 106

平台操作 112

112 的快捷形式 – 113

开放关键字 35, 285

开放模块 45

OpenJDK 637 – 638

opens 关键字 35

OpenTelemetry 433 – 434

操作

自动化 346

Clojure 315 – 316

*Optimizing Java* (Evans, Gough, Newland) 246

Optional 类型 288

或函数 484

Oracle JDK 637 – 638

Oracle OpenJDK 构建 637 – 638

编排器 424

有序关闭 203

OrderPartition 类型 68

org.apache.catalina.Executor 类 206

org.beryx.jlink 插件 395

org.graalvm.js.scriptengine 模块 38

org.graalvm.sdk 模块 38

org.junit.jupiter.junit-jupiter-api 依赖 459

org.junit.jupiter.junit-jupiter-engine 依赖 459

org.testcontainers:testcontainers 库 472

原始语言，重新实现与 255 – 256

操作系统 (操作系统) 619

OSR (栈上替换) 238

overhead, concurrency 127 – 128

P

-P 开关 417

-p 开关 577

包阶段 350

包保护默认 282

付费支持选项 639

并行收集器 231 – 232

并行操作 648

并行流 547 – 549

parallel() 方法 648

父类加载器 88

Parlog, Nicolai 116

部分应用

Java 作为函数式编程语言的限制 513

Kotlin 函数式编程 518

概述 502

偏序 147

按值传递语言 510

编写测试，传递 445

模式匹配 74 – 77, 290

模式变量 75

暂停目标 229

pcalls 函数 563 – 565

待处理队列 193

性能

替代 JVM 语言 266

并发 126

目标 214 – 215

在容器中 434 – 435

性能分析

垃圾回收 224 – 233

内存区域 227

基础 225

配置参数 232 – 233

完整集合 228

G1 229 – 231

标记和清除 225 – 227

并行收集器 231 – 232

安全点 228

年轻代收集器 228

Java 性能调优 220 – 224

缓存未命中 222 – 224

时间在 221 – 222

JDK 飞行记录器 238 – 246

飞行记录器 239 – 240

任务控制 240 – 246

即时编译 (JIT) 233 – 238

去优化 238

动态编译 234, 236 – 237

HotSpot 235 – 236

内联方法 236

阅读编译日志 237 – 238

对 212 的实用方法 – 217

实现更高性能的成本 215 – 216

过早优化的危险 216 – 217

如何进行测量 213 – 214

性能目标 214 – 215

要测量什么 212 – 213

何时停止 215

术语 209 – 211

容量 211

退化 211

效率 211

延迟 210

可扩展性 211

吞吐量 210

利用率 210

为什么要关心 217 – 220

内存延迟层次 219 – 220

摩尔定律 217 – 219

PermGen (永久生成) 231

permgen Java 堆 572

允许 | 警告 | 拒绝设置 45

允许关键字 72

持久数据结构 521, 557 – 563

PersistentVector.Node 内部类 559

阶段 350

平台功能，Java 13 – 16

更改语言 14 – 15

定义 4 – 6

孵化功能和预览功能 15 – 16

JSRs 和 JEPs 15

语法糖 14

平台模块 38 – 39

平台操作指令 112

PlatformClassLoader 86

<插件> 元素 354

插件 351

编写 Maven 372 – 376

Gradle

创建自定义插件 399 – 400

概述 378 – 379

加法函数 555

Pod 对象类型 425

波兰表示法 308

poll() 方法 194

在 JVM 上进行多语言编程 256 – 262

竞争者 260 – 262

GraalVM 261

Groovy 260

非 JVM 语言 261 – 262

Scala 260 – 261

非 Java 语言，使用原因 257 – 259

新兴语言 259 – 260

Clojure 260

Kotlin 259 – 260

POM (项目对象模型) 351 – 353

端口，Docker 416 – 418

Postgres 472 – 473

Postgres 测试容器对象 473

精度 221

谓词接口 644

谓词 278

抢占式线程 624

过早优化 216 – 217

准备阶段 85

预处理器指令 336

预览功能 15 – 16, 75 – 77

Price 接口 451, 467

原始类 628

原始类型特殊化 647

原始值类型 632

原始类型 646 – 647

ProcessHandle 类 50

项目 Amber 609 – 612

<项目> 元素 354, 358

项目 Jigsaw 28 – 31

封装内部 30

JVM 现在是模块化的 30 – 31

模块化 Java 运行时 28 – 29

项目 Lambda 640

项目 Loom 618 – 625

使用虚拟线程编程 623 – 625

发布 625

线程构建类 622 – 623

虚拟线程 621 – 622

项目巴拿马

Foreign Function and Memory API 612 – 618

在巴拿马处理原生内存 617 – 618

LibSPNG 示例 614 – 617

概述 612

项目 Valhalla 626 – 633

改变语言模型 630

值对象的影响 630 – 633

泛型 633

属性 487

<属性> 元素 354

属性测试 263

基于属性的测试 263, 481 – 493

clojure.spec 483 – 486

clojure.spec 和 test.check 492 – 493

clojure.test 481 – 483

test.check 487 – 492

协议 341

provides 关键字 35

代理

Clojure 333 – 334

反射与 586 – 589

public static final 变量 70

public 可见性 284

纯函数

Java 作为函数式编程语言的限制 503

Kotlin 函数式编程 516 – 517

概述 498

put() 方法 111, 178

putAll() 方法 179

putfield 操作码 106, 152, 159, 168, 630

putIfAbsent() 方法 184

putstatic 操作码 102, 106

pwd 命令 410

Q

Q 类型描述符 632

限定导出 39

查询变量 56

队列接口 188

快速检查函数 490

R

RAII（资源获取即初始化）模式 617

范围 529

reader 宏 321 – 323

实时 Java 235 – 236

rebinding 304

接收器对象 115

记录 60 – 69

紧凑记录构造函数 67 – 69

名义类型 66 – 67

记录类 63

记录组件 63

记录命名风格 145

递归关系 534

递归

Java 作为函数式编程语言的限制 506 – 509

Kotlin 522 – 526

概述 500

Redis 419, 467 – 471

redis 键 419

redis 对象 471

reduce() 方法 643, 645

可重入锁实现 172

可重入读写锁实现 172

重构测试 445 – 446

引用透明性 498

反射 45 – 46, 113 – 118

组合类加载 116 – 117

内部结构 578 – 583

概述 114 – 116

问题 117 – 118

代理与 586 – 589

重实现语言 255 – 256

- -release 标志 51

remove (int index) 列表变异方法 514

remove() 方法 184

REPL (读取-评估-打印循环) 302

使用 334 – 335

入门 303 – 305

replace() 方法 184

requires 关键字 35 – 37

解析阶段 85

ResourceScope 类 617

响应对象 23

受限连接 20 – 21

受限关键字 35

ret 操作码 (已弃用) 108

可重用性 126 – 127

Rhino 语言 255

根模块 40

RUN Docker 命令 407

运行测试函数 482

run() 方法 102, 138, 200

Runnable 接口 255

RUNNABLE 线程状态 130, 138

非 Java 语言的运行时环境 266 – 267

运行时布线 93

运行时管理的并发 121

runtimeClasspath Gradle 依赖配置 383 – 384

runtimeOnly Gradle 依赖配置 384

rust-bindgen 工具 618

S

安全点 228

安全性

并发类型安全 125 – 126

Kotlin 287 – 291

空安全 288 – 289

智能转换 289 – 291

SAM (单抽象方法) 146

Scala 260 – 261

可伸缩性 211

标量化 628

scheduleAtFixedRate()方法 205

ScheduledThreadPoolExecutor (STPE) 205 – 206

scheduleWithFixedDelay()方法 205

Schwartzian 转换 318

<作用域>元素 382

作用域 550

作用域，协程 554 – 556

脚本，Gradle 378

密封接口 72

密封关键字 73

密封类型 69, 71 – 74

次构造函数 284

SEDA (分阶段事件驱动架构)方法 621

SegmentAllocator 接口 613, 617

Selenium 474 – 476

语义版本控制 349

语义。*见*语法和语义

发送方法 23

sendAsync 方法 23

seq (序列) 325

seq 方法 557

sequence lambda 530

Sequence<T>接口 526

sequenceOf()函数 528

序列 527

Clojure 325 – 330

Kotlin 函数式编程 526 – 530

服务器 JRE 28

-server 开关 235

服务抽象 430

服务 Bean 94

服务 Kubernetes 对象类型 425

Set 接口 179, 643

setAccessible() 方法 45，117

集合，Clojure 312 – 315

浅不可变性 147

共享可变状态 544

快捷形式 112 – 113

收缩 491

关闭标志 167

shutdown() 方法 195

无副作用 498，644

单文件源代码程序（JEP 330）24 – 25

单线程执行器 203

站点生命周期 351

size() 方法 644

斯凯纳，史蒂文 128

小内部更改 593 – 600

紧凑字符串 596 – 597

巢居者 597 – 600

字符串连接 593 – 595

智能转换 289

软件事务内存 565 – 569

sort() 方法 641

- -source 标志 24

特殊形式，Clojure 309 – 311

特殊方法 576 – 577

规范式测试 476 – 481

规范 476

Spek 476 – 481

拆分包 47

SREs（软件可靠性工程师）433

SSD（固态硬盘）219

栈 225

start() 方法 138，622

startVirtualThread() 静态方法 622

状态机 550

状态相关更新 137

静态分析 389 – 390

静态方法 287

静态同步方法 129

静态类型 253 – 254

STDERR 输出流 424

STDOUT 输出流 424

斯蒂尔，盖·L. 226

stop() 方法 166, 293

store 操作码 106

Stream 抽象 640, 642

Stream 接口 511, 642 – 643

Stream 对象 643

stream() 方法 511, 643

流 642 – 646

String 构造函数 595

字符串表示 597

字符串类型 73, 273, 288, 589, 593

StringBuilder 对象 216, 594

StringConcatFactory 类 595

字符串

紧凑型 596 – 597

连接 593 – 595

StringSeq 类 335

结构数组 629

结构化共享 557

结构化类型 13, 66

stub 对象 449, 451 – 453

STW (Stop-the-World) 226

子操作码 108

submit() 方法 543

*敏捷成功* (Cohn) 438

sudo ip addr show 命令 418

求和类型 73

sun.misc.Unsafe 类 170, 600

sun.net 包 33

supplyAsync() 工厂方法 546

surefire 插件 459

suspend 函数 293, 530, 550

suspend 关键字 293, 550

suspend() 方法 143

Sutter, Herb 218

switch 表达式 57 – 60

switch 语句 57

Symbol 类 305

同步

字节码中 153 – 157

锁和 128 – 130

原因 135 – 137

基于同步的并发 119

synchronized 关键字 120, 157

synchronized 关键字 150, 157 – 158

synchronizedMap()方法 181

同步关系 147

语法糖 14

语法和语义

Clojure 309 – 323

算术、相等和其他操作 315 – 316

函数 316 – 319

列表、向量、映射和集合 312 – 315

循环 319 – 320

读取宏和调度 321 – 323

特殊形式 309 – 311

Java 模块 34 – 37

导出和需求 36 – 37

传递性 37

合成访问方法 599

System.currentTimeMillis()方法 110

System.getenv()方法 411

T

tableswitch 操作码 108

尾位置 508

尾递归 507, 522 – 525

tailrec 关键字 524

尾递归非尾递归函数 525

take()方法 194, 529

任务管理器对象 167

任务和执行 200 – 206

缓存线程池 205

执行器接口 202

固定线程池 204 – 205

Gradle 377 – 378

建模任务 200 – 201

Callable 接口 201

FutureTask 类 201

单线程执行器 203

STPE (ScheduledThreadPoolExecutor) 205 – 206

tasks.jar 配置 381

TDD (测试驱动开发) 441 – 448

多个用例示例 446 – 448

概述 442 – 443

单一用例示例 443 – 448

重构测试 445 – 446

编写失败的测试（红色） 443 – 445

编写通过测试（绿色） 445

终端方法 511

已终止线程状态 131

测试调用 478

测试双倍 449 – 458

哑对象 449 – 451

伪造对象 454 – 456

模拟对象 456 – 457

模拟的问题 457 – 458

存根对象 451 – 453

测试函数 478

测试 lambda 479

测试方法 480

测试阶段 350

测试支持，替代 JVM 语言 264

测试任务 477

测试编译阶段 353

*测试驱动开发：通过示例*（Beck） 441, 443

test.check 487 – 493

test.check-compatible 生成器 492

测试编译 Classpath Gradle 依赖配置 384

测试编译 Only Gradle 依赖配置 384

Testcontainers 466 – 476

使用 Selenium 进行端到端测试的示例 474 – 476

收集容器日志 471

安装 467

Postgres，示例 472 – 473

Redis，示例 467 – 471

testImplementation Gradle 依赖配置 384, 389

测试

从 JUnit 4 到 5 459 – 464

Gradle 388 – 389

如何 438 – 441

使用 Testcontainers 进行集成测试 466 – 476

使用 Selenium 进行端到端测试的示例 474 – 476

收集容器日志 471

安装 testcontainers 467

Postgres，示例 472 – 473

Redis，示例 467 – 471

Maven 357 – 360

使用 Clojure 进行基于属性的测试 481 – 493

clojure.spec 483 – 486

clojure.spec 和 test.check 492 – 493

clojure.test 481 – 483

test.check 487 – 492

原因 438

使用 Spek 和 Kotlin 进行规范式测试 476 – 481

TDD（测试驱动开发） 441 – 448

多用途案例示例 446 – 448

概述 442 – 443

单用途案例示例 443 – 448

测试替身 449 – 458

哑对象 449 – 451

伪造对象 454 – 456

模拟对象 456 – 457

模拟的问题 457 – 458

存根对象 451 – 453

testRuntimeClasspath Gradle 依赖配置 384

testRuntimeOnly Gradle 依赖配置 384

文本块 55 – 57

然后形成 338

thenApply() 方法 546

理论入门，并发 120 – 124

Amdahl 的定律 121 – 122

硬件 121

Java 线程模型 123 – 124

经验教训 124

Thread 熟人假设 120 – 121

线程

熟人假设 120 – 121

线程构建类 622 – 623

Thread.Builder 类 622

Thread.interrupted() 方法 141

Thread.State 枚举 130

Thread.stop() 方法 143

ThreadDeath 异常 143

ThreadFactory 实例 623

线程模型 123 – 124

ThreadPoolExecutor 类 203

线程

状态模型 130 – 131

状态和方法 137 – 143

已弃用的线程方法 143

中断线程 140 – 142

处理异常和线程 142 – 143

吞吐量 210

分层编译 236

时间和性能调整 221 – 222

精度 221 – 222

粒度 222

测量 222

网络分布式计时 222

精度 221

时间变量 110

TIMED_WAITING 线程状态 131

时间片 624

TLS, HTTP/2 (Java 11) 21 – 22

转换为关键字 35

ToIntFunction 特殊函数类型 647

工具，替代 JVM 语言 264

toString() 方法 64, 585

touchEveryItem() 函数 223

传递性闭包 85

传递性依赖 347

传递性关键字 35

传递性 37

传输级更新 19

真实值，Clojure 316

try-with-resources 617

Tudose, C*ătă*lin 443

图灵完备性 500

类型 1 虚拟机 402

类型 2 虚拟机 402 – 403

类型描述符 96

类型擦除 154

类型提示 272

类型推断（var 关键字） 9 – 13

类型模式 75

类型系统 514 – 515

U

无界执行器 625

联合类型 73

单元测试 439

未命名的模块 40

Unsafe

概述 600 – 604

替换为支持的 API 604 – 608

隐藏类 607 – 608

VarHandles 605 – 607

UnsupportedClassVersionError 错误 90

非同步读取 158 – 160

URLCanonicalizer 类 32 – 33

Urma, Raoul-Gabriel 640

用户命名空间 310, 483

userid 整数 9

uses 关键字 35

Utf8 99 – 100

利用率 210

V

val 声明 519

valid? 函数 483

验证阶段 350

值类 628, 630 – 633

值记录 629

ValueObject 接口 630

values 510

values() 方法 179

Var 类，Clojure 305

var 关键字 253, 272

var 对象，Clojure 302

var 属性，Kotlin 290

VarHandles 605 – 607

变量，Kotlin 272 – 273

可变参数函数 329 – 330

变体代码 49

向量，Clojure 312 – 315

验证阶段 84 – 85

Verify 阶段 351

验证行为 457

Vermeulen, Marco 516

虚拟方法 572 – 575

虚拟线程

概述 621 – 622

使用编程 623 – 625

VirtualMachineDescriptor 类 43

VMs (虚拟机) 403

VMSpec (JVM 规范) 4

void 方法 117, 332

volatile 关键字 120, 137

Volatile Shutdown 模式 166 – 168

W

等待序列懒序列 565

wait() 方法 630

when 关键字 280

when() 方法 457

while 循环 140, 193, 319

with 关键字 35

withers 499

withfield 操作码 631

避免工作，Gradle 381 – 382

工作窃取算法 543 – 544

*Working Effectively with Legacy Code* (Feather) 442

WorkUnit 196 – 197

包装器 377

X

*xUnit Test Patterns* (Meszaros) 449

-XX:+UnlockCommercialFeatures 选项 240

Y

yield 关键字 58, 554

年轻集合 228

Z

僵尸，编译日志和 238
