# 第七章\. 使用流处理数据

流利用了 Java 8 引入的许多函数特性，提供了一种声明性的方式来处理数据。流 API 涵盖了许多用例，但你需要了解不同的操作和可用的辅助类，以充分利用它们。

第六章 着重介绍了流的基础知识。本章将在此基础上构建，并教你创建和处理流以解决各种用例。

# 原始流

在 Java 中，泛型仅适用于基于对象的类型（参见¹）。这就是为什么`Stream<T>`不能用于像`int`这样的原始值序列的原因。使用原始类型与流的唯一两种选项是：

+   自动装箱

+   专门的流变体

Java 的自动装箱支持 — 自动将原始类型与像`int`和`Integer`之类的基于对象的对应物转换 — 可能看起来像是一个简单的解决方案，因为它自动地工作，如下所示：

```java
Stream<Long> longStream = Stream.of(5L, 23L, 42L);
```

尽管自动装箱引入了多个问题。例如，与直接使用原始类型相比，从原始值到对象的转换会导致开销。通常情况下，这种开销可以忽略不计。然而，在数据处理管道中，频繁创建包装类型的开销会累积，并可能降低整体性能。

原始包装类型的另一个非问题是可能存在`null`元素的情况。从原始类型直接转换为对象类型永远不会产生`null`，但是如果在流水线操作中需要处理包装类型而不是原始类型，则任何操作可能返回`null`。

为了减少这种情况，与 JDK 的其他函数特性一样，Stream API 为`int`、`long`和`double`等原始类型提供了专门的变体，而不依赖于自动装箱，如表 7-1 中所列。

表 7-1\. 原始流及其等价物

| 原始类型 | 原始流 | 装箱流 |
| --- | --- | --- |
| `int` | `IntStream` | `Stream<Integer>` |
| `long` | `LongStream` | `Stream<Long>` |
| `double` | `DoubleStream` | `Stream<Double>` |

原始流上的可用操作与它们的通用对应物相似，但使用原始功能接口。例如，`IntStream`提供了一个`map`操作来转换元素，就像`Stream<T>`一样。不过，不同于`Stream<T>`，进行此操作所需的高阶函数是专门的变体`IntUnaryOperator`，它接受并返回`int`，如下简化的接口声明所示：

```java
@FunctionalInterface
public interface IntUnaryOperator {

    int applyAsInt(int operand);

    // ...
}
```

在原始流上接受高阶函数的操作使用专门的功能接口，如`IntConsumer`或`IntPredicate`，以保持在原始流的限制范围内。与`Stream<T>`相比，这减少了可用操作的数量。不过，你可以通过将其映射到另一种类型或将原始流转换为其装箱变体来轻松地在原始流和`Stream<T>`之间切换：

+   `Stream<Integer> boxed()`

+   `Stream<U> mapToObj(IntFunction<? extends U> mapper)`

反过来，从`Stream<T>`到原始流也是支持的，`Stream<T>`上可用`mapTo...`和`flatMapTo...`操作：

+   `IntStream mapToInt(ToIntFunction<? super T> mapper)`

+   `IntStream flatMapToInt(Function<? super T, ? extends IntStream> mapper)`

除了常规的中间操作外，原始流还具有一组直观的算术终端操作用于常见任务：

+   `int sum()`

+   `OptionalInt min()`

+   `OptionalInt max()`

+   `OptionalDouble average()`

这些操作不需要任何参数，因为它们的行为对于数字是不可协商的。返回的类型是你从类似`Stream<T>`操作中期望的原始等价类型。

与一般的原始流一样，使用流进行算术运算有其用例，比如高度优化的并行处理大量数据。不过，对于更简单的用例，与现有的处理结构相比，切换到原始流通常不值得。

# 迭代流

流管道及其内部迭代通常处理现有元素序列或可轻松转换为元素序列的数据结构。与传统的循环结构相比，你必须放弃控制迭代过程，让流接管。然而，如果你需要更多控制，流 API 仍提供了在`Stream<T>`类型上可用的`static iterate`方法：

+   `<T> Stream<T> iterate(T seed, UnaryOperator<T> f)`

+   `IntStream iterate(int seed, IntUnaryOperator f)`

Java 9 添加了两种额外的方法，包括一个`Predicate`变体来设定结束条件：

+   `<T> Stream<T> iterate(T seed, Predicate<T> hasNext, UnaryOperator<T> next)`

+   `IntStream iterate(int seed, IntPredicate hasNext, IntUnaryOperator next)`

对于它们对应的流变体，`int`、`long`和`double`都有原始的`iterate`变体可用。

流的迭代方法产生通过将`UnaryOperator`应用于种子值而成为*有序*且可能无限的元素序列。换句话说，流元素将是`[seed, f(seed), f(f(seed)), ...]`等等。

如果这一般概念感觉熟悉，你是对的！它是`for`循环的流等价物：

```java
// FOR-LOOP
for (int idx = 1; ![1](img/1.png)
     idx < 5; ![2](img/2.png)
     idx++) { ![3](img/3.png)
  System.out.println(idx);
}

// EQUIVALENT STREAM (Java 8)
IntStream.iterate(1, ![1](img/1.png)
                  idx -> idx + 1) ![3](img/3.png)
         .limit(4) ![2](img/2.png)
         .forEachOrdered(System.out::println);

// EQUIVALENT STREAM (Java 9+)
IntStream.iterate(1, ![1](img/1.png)
                  idx -> idx < 5, ![2](img/2.png)
                  idx -> idx + 1) ![3](img/3.png)
         .forEachOrdered(System.out::println);
```

![1](img/#co_working_with_streams_CO1-1)

种子，或初始迭代值。

![2](img/#co_working_with_streams_CO1-2)

终止条件。

![3](img/#co_working_with_streams_CO1-3)

迭代值的增加。`for`循环需要赋值，而流需要返回值。

循环和流的变体为循环体/后续流操作产生相同的元素。Java 9 引入了带有限制`Predicate`的`iterate`变体，因此不需要额外的操作来限制总体元素。

迭代流相比`for`循环的最大优势在于，您仍然可以使用类似循环的迭代，但获得惰性函数流管道的好处。

不必在流创建时定义结束条件。相反，稍后的中间流操作，如`limit`，或终端条件，如`anyMatch`，可以提供它。

迭代流的特性为`ORDERED`、`IMMUTABLE`，对于原始流而言还包括`NONNULL`。如果迭代基于数字且范围已知，可以通过`IntStream`和`LongStream`上的静态`range…​`方法获得更多流的优化，如短路。

+   `IntStream range(int startInclusive, int endExclusive)`

+   `IntStream rangeClosed(int startInclusive, int endInclusive)`

+   `LongStream range(long startInclusive, +long endExclusive)`

+   `LongStream rangeClosed(long startInclusive, long endInclusive)`

即使可以通过`iterate`获得相同的结果，主要区别在于底层的`Spliterator`。返回的流的特性为`ORDERED`、`SIZED`、`SUBSIZED`、`IMMUTABLE`、`NONNULL`、`DISTINCT`和`SORTED`。

选择迭代或范围流的创建取决于您的需求。迭代方法为迭代过程提供更多自由度，但失去了流特性，尤其是在并行流中提供最优化可能性。

# 无限流

流的惰性特性允许元素作为它们被*按需*处理，而不是*一次性*处理。

JDK 中所有可用的流接口 — `Stream<T>`及其原始衍生物`IntStream`、`LongStream`和`DoubleStream` — 都具有用于基于迭代方法或无序生成方法创建无限流的静态便捷方法。

虽然前面的`iterate`方法从一个*种子*开始，并依赖于在当前迭代值上应用它们的`UnaryOperator`，但`static generate`方法只依赖于`Supplier`生成它们的下一个流元素：

+   `<T> Stream<T> generate(Supplier<T> s)`

+   `IntStream generate(IntSupplier s)`

+   `LongStream generate(LongSupplier s)`

+   `DoubleStream generate(DoubleSupplier s)`

缺少起始种子值会影响流的特性，使其变为`UNORDERED`，这对并行使用很有益。通过`Supplier`创建的无序流对于常量、非相互依赖的元素序列（例如随机值）非常有帮助。例如，创建一个`UUID`流工厂非常简单：

```java
Stream<UUID> createStream(int count) {
  return Stream.generate(UUID::randomUUID)
               .limit(count);
}
```

无序流的缺点在于，它们不能保证`limit`操作会在并行环境中选择前`n`个元素。这可能导致对元素生成`Supplier`的调用次数比实际结果流所需的次数多。

参考以下示例：

```java
Stream.generate(new AtomicInteger()::incrementAndGet)
      .parallel()
      .limit(1_000)
      .mapToInt(Integer::valueOf)
      .max()
      .ifPresent(System.out::println);
```

流管道的预期输出是`1000`。但输出很可能大于`1000`。

这种行为在并行执行环境中的无序流中是预期的。在大多数情况下，这并不重要，但它突显了选择具有良好特性的正确流类型以获得最大性能和尽可能少的调用的必要性。

## 随机数

流 API 对生成无限随机数流有特殊考虑。虽然可以使用`Stream.generate`来创建这样的流，例如使用`Random#next()`，但也有更简单的方法可用。

有三种不同的随机数生成类型能够创建流：

+   `java.util.Random`

+   `java.util.concurrent.ThreadLocalRandom`

+   `java.util.SplittableRandom`

所有这三种类型都提供了多种方法来创建随机元素的流：

```java
IntStream ints()
IntStream ints(long streamSize)

IntStream ints(int randomNumberOrigin,
               int randomNumberBound)

IntStream ints(long streamSize,
               int randomNumberOrigin,
               int randomNumberBound)

LongStream longs()

LongStream longs(long streamSize)

LongStream longs(long randomNumberOrigin,
                 long randomNumberBound)

LongStream longs(long streamSize,
                 long randomNumberOrigin,
                 long randomNumberBound)

DoubleStream doubles()

DoubleStream doubles(long streamSize)

DoubleStream doubles(double randomNumberOrigin,
                     double randomNumberBound)

DoubleStream doubles(long streamSize,
                     double randomNumberOrigin,
                     double randomNumberBound)
```

技术上，这些流只是*有效地无限*，如它们文档中所述²。如果未提供`streamSize`，结果流将包含`Long.MAX_VALUE`个元素。上限和下限由`randomNumberOrigin`（包括）和`randomNumberBound`（不包括）设置。

将在 “示例：随机数” 中讨论一般使用和性能特征。

## 记忆并非无限

当使用无限流时最重要的一点是，你的内存是有限的。限制无限流不仅重要，而且是绝对必要的！忘记放置限制性的中间或终端操作将不可避免地使用完 JVM 可用的所有内存，并最终抛出`OutOfMemoryError`。

列出了用于限制任何流的可用操作，见表 7-2。

表 7-2\. 流限制操作

| 操作类型 | 操作 | 描述 |
| --- | --- | --- |
| 中间操作 | `limit(long maxSize)` | 将流限制为`maxSize`个元素 |
| `takeWhile(Predicate<T> predicate)` | 取元素直到`predicate`评估为`false`（Java 9+） |
| 终端操作（保证） | `Optional<T> findFirst()` | 返回流的第一个元素 |
| `Optional<T> findAny()` | 返回单个、非确定性的流元素 |
| 终端操作（不保证） | `boolean anyMatch(Predicate<T> predicate)` | 返回是否有*任何*流元素与`predicate`匹配 |
| `boolean allMatch(Predicate<T> predicate)` | 返回是否*所有*流元素与`predicate`匹配 |
| `boolean noneMatch(Predicate<T> predicate)` | 返回是否*没有*流元素与`predicate`匹配 |

最简单的选择是`limit`。像`takeWhile`这样使用`Predicate<T>`的选择操作必须谨慎制定，否则可能仍会导致流消耗比需要更多的内存。对于终端操作，只有`find…​`操作能保证终止流。

`…​Match`操作与`takeWhile`存在相同的问题。如果谓词不符合它们的目的，流管道将处理*无限*数量的元素，并因此消耗所有可用内存。

如“操作的成本”中讨论的，流中限制操作的位置也会影响通过的元素数量。即使最终结果可能相同，尽早限制流元素的流动将节省更多内存和 CPU 周期。

# 从数组到流，再从流到数组

数组是一种特殊类型的对象。它们是一种类似集合的结构，保存其*基本类型*的元素，并且仅提供一种通过索引访问特定元素以及数组的总长度的方法，除了从`java.lang.Object`继承的*通常*方法。它们也是未来直到*Valhalla 项目*变得可用之前，拥有原始类型集合的唯一方式³。

然而，数组具有两个特征使其非常适合基于流的处理。首先，它们在创建时就确定了长度并且不会改变。其次，它们是有序序列。这就是为什么`java.util.Arrays`上有多个便利方法来为不同的基本类型创建适当流的原因。使用适当的终端操作可以从流创建数组。

## 对象类型数组

创建典型的`Stream<T>`由`java.util.Arrays`上的两个`static`便利方法支持：

+   `<T> Stream<T> stream(T[] array)`

+   `<T> Stream<T> stream(T[] array, int startInclusive, int endExclusive)`

如你所见，从数组创建`Stream<T>`非常简单明了。

另一种方式，从`Stream<T>`到`T[]`是通过使用以下两种终端操作之一来完成的：

+   `Object[] toArray()`

+   `<A> A[] toArray(IntFunction<A[]> generator)`

第一种变体无论流的实际元素类型如何，都只能返回一个`Object[]`数组，这是由 JVM 创建数组的方式决定的。如果需要流元素类型的数组，必须为流提供一种创建适当数组的方法，这就是第二种变体的作用所在。

第二种变体需要一个`IntFunction`来创建指定大小的数组。最简单的方法是使用方法引用：

```java
String[] fruits = new String[] {
    "Banana",
    "Melon",
    "Orange"
};

String[] result = Arrays.stream(fruits)
                        .filter(fruit -> fruit.contains("a"))
                        .toArray(String[]::new);
```

###### 警告

在`toArray`中使用创建的数组时没有静态类型检查。类型在元素存储在分配的数组中时在运行时检查，如果类型不兼容则抛出`ArrayStoreException`。

## 原始数组

三种原始流专门化类型`IntStream`、`LongStream`和`DoubleStream`都有专用的`static`方法`Arrays.stream`变体：

+   `IntStream stream(int[] array)`

+   `IntStream stream(int[] array, int startInclusive, int endExclusive)`

`LongStream`和`DoubleStream`变体只在`array`类型和返回的原始流中有所不同。

因为原始流中的元素类型是固定的，它们只有一个不需要`IntFunction`的`toArray`方法：

```java
int[] fibonacci = new int[] {
    0, 1, 1, 2, 3, 5, 8, 13, 21, 34
};

int[] evenNumbers = Arrays.stream(fibonacci)
                          .filter(value -> value % 2 == 0)
                          .toArray();
```

# 低级流创建

到目前为止，我讨论的所有流创建方法都非常高级，从另一个数据源、迭代、生成或任意对象创建流。它们直接在各自的类型上可用，尽可能少地需要参数。辅助类型`java.util.stream.StreamSupport`还有几个低级`static`方便方法可用于直接从 Spliterator 创建流。这样，你可以为自己的自定义数据结构创建流表示。

以下两种方法接受`Spliterator`以创建新的流：

`Stream<T> stream(Spliterator<T> spliterator, boolean parallel)`

从任何可以由`Spliterator<T>`表示的源创建顺序或并行流的最简单方式。

`Stream<T> stream(Supplier<? extends Spliterator<T>> supplier, int characteristics, boolean parallel)`

不要立即使用 Spliterator，而是在 Stream 管道的终端操作调用之后调用 Supplier。这样可以将对源数据结构的任何可能干扰传递到更小的时间范围，使得对非`IMMUTABLE`或非`CONCURRENT`急切绑定流更安全。

强烈建议用于创建`Stream<T>`的 Spliterators 要么是`IMMUTABLE`，要么是`CONCURRENT`，以最小化可能的干扰或对底层数据源的更改。

另一个好的选择是使用*延迟绑定* Spliterator，意味着元素在 Spliterator 创建时不是固定的。相反，它们在首次使用时绑定，即在调用终端操作后 Stream 管道开始处理其元素时。

###### 注意

原始 Spliterator 变体也存在低级流创建方法。

如果你没有`Spliterator<T>`而是一个`Iterator<T>`，那么 JDK 已经为你准备好了。类型`java.util.Spliterators`有多个方便的方法用于创建 Spliterators，其中两种方法专门用于`Iterator<T>`：

```java
Spliterator<T> spliterator(Iterator<? extends T> iterator,
                                                 long size,
                                                 int characteristics)

Spliterator<T> spliteratorUnknownSize(Iterator<? extends T> iterator,
                                      int characteristics)
```

您可以使用先前讨论的 `Stream<T> stream(Spliterator<T> spliterator, boolean parallel)` 方法中创建的创建的 `Spliterator<T>` 实例来最终创建一个 `Stream<T>`。

# 处理文件 I/O

流不仅用于基于集合的遍历。它们还提供了一个与 `java.nio.file.Files` 类一起遍历文件系统的绝佳方法。

本节将讨论文件 I/O 和 Streams 的几个用例。与其他 Streams 不同，I/O 相关的 Streams 必须在使用完毕后通过调用 `Stream#close()` 显式关闭。 `Stream<T>` 符合 `java.lang.AutoCloseable` 接口，因此示例将使用 `try-with-resources` 块进行处理，这将在 “文件 I/O 流的注意事项” 中详细解释。

本节中的所有示例都使用书籍的 [代码存储库](https://github.com/benweidig/a-functional-approach-to-java) 中的文件作为它们的来源。以下文件系统树表示示例中使用的文件的整体结构：

```java
├── README.md
├── assets
│   └── a-functional-approach-to-java.png
├── part-1
│   ├── 01-an-introduction-to-functional-programming
│   │   └── README.md
│   ├── 02-functional-java
│   │   ├── README.md
│   │   ├── java
|   |   └─ ...
└── part-2
    ├── 04-immutability
    │   ├── ...
    │   └── jshell
    │       ├── immutable-copy.java
    │       ├── immutable-math.java
    │       ├── unmodifiable-list-exception.java
    │       └── unmodifiable-list-modify-original.java
    ├─ ...
```

## 读取目录内容

通过调用方法 `Files.list` 来列出目录内容，以创建所提供 `Path` 的惰性填充的 `Stream<Path>`：

```java
static Stream<Path> list(Path dir) throws IOException
```

其参数必须是目录，否则将抛出 `NotDirectoryException`。示例 7-1 展示了如何列出一个目录。

##### 示例 7-1\. 列出一个目录

```java
var dir = Paths.get("./part-2/04-immutability/jshell");

try (var stream = Files.list(dir)) {
  stream.map(Path::getFileName)
        .forEach(System.out::println);
} catch (IOException e) {
  // ...
}
```

输出列出了章节 4 的目录 `jshell` 中的文件：

```java
unmodifiable-list-exception.java
unmodifiable-list-modify-original.java
immutable-copy.java
immutable-math.java
```

检索的内容顺序不能保证，我将在 “文件 I/O 流的注意事项” 中详细介绍。

## 深度优先目录遍历

两个 `walk` 方法如其名称所示，从特定起始点“遍历”整个文件树。惰性填充的 `Stream<Path>` 遵循*深度优先*，这意味着如果一个元素是目录，则会首先进入并遍历该目录，然后再遍历当前目录中的下一个元素。

`java.nio.file.Files` 中两个 `walk` 变体的区别在于它们将遍历的最大目录深度：

```java
static Stream<Path> walk(Path start, ![1](img/1.png)
                         int maxDepth, ![2](img/2.png)
                         FileVisitOption... options) ![3](img/3.png)
                         throws IOException

static Stream<Path> walk(Path start, ![1](img/1.png)
                         FileVisitOption... options) ![3](img/3.png)
                         throws IOException
```

![1](img/#co_working_with_streams_CO2-1)

遍历的起始点。

![2](img/#co_working_with_streams_CO2-2)

要遍历的最大目录级别。 `0`（零）将流限制为起始级别。第二个没有 `maxDepth` 的变体没有深度限制。

![3](img/#co_working_with_streams_CO2-3)

零个或多个选项用于遍历文件系统。到目前为止，只有 `FOLLOW_LINKS` 存在。请注意，通过跟随链接，可能会发生可能的循环遍历。如果 JDK 检测到这一点，它会抛出 `FileSystemLoopException`。

可以按照 示例 7-2 所示的方式遍历文件系统。

##### 示例 7-2\. 遍历文件系统

```java
var start = Paths.get("./part-1");

try (var stream = Files.walk(start)) {
  stream.map(Path::toFile)
        .filter(Predicate.not(File::isFile))
        .sorted()
        .forEach(System.out::println);
} catch (IOException e) {
  // ...
}
```

遍历生成以下输出：

```java
./part-1
./part-1/01-an-introduction-to-functional-programming
./part-1/02-functional-java
./part-1/02-functional-java/java
./part-1/02-functional-java/jshell
./part-1/02-functional-java/other
./part-1/03-functional-jdk
./part-1/03-functional-jdk/java
./part-1/03-functional-jdk/jshell
```

流至少包含一个元素，即起始点。如果不可访问，则会抛出`IOException`。与`list`类似，流元素的遇见顺序不被保证，我将在“文件 I/O 流的注意事项”中详细讨论这一点。

## 搜索文件系统

尽管可以使用`walk`搜索特定的`Path`，但也可以使用`find`方法。它将一个`BiPredicate`与当前元素的`BasicFileAttribute`直接集成到流创建中，使流更专注于您任务的需求：

```java
static Stream<Path> find(Path start, ![1](img/1.png)
                         int maxDepth, ![2](img/2.png)
                         BiPredicate<Path, BasicFileAttributes> matcher, ![3](img/3.png)
                         FileVisitOption... options) ![4](img/4.png)
                         throws IOException
```

![1](img/#co_working_with_streams_CO3-1)

搜索的起始点。

![2](img/#co_working_with_streams_CO3-2)

遍历的最大目录级别。`0`（零）限制到起始级别。与`Files.walk`不同，不存在无需`maxDepth`的方法变体。

![3](img/#co_working_with_streams_CO3-3)

包含在流中的`Path`的条件。

![4](img/#co_working_with_streams_CO3-4)

文件系统遍历的零个或多个选项。目前只有`FOLLOW_LINKS`存在。请注意，通过跟踪链接可能会发生可能的循环遍历。如果 JDK 检测到这一点，它会抛出`FileSystemLoopException`。

使用它，可以实现示例 7-2 而无需将`Path`映射到`File`，如示例 7-3 所示。

##### 示例 7-3\. 查找文件

```java
var start = Paths.get("./part-1");

BiPredicate<Path, BasicFileAttributes> matcher =
  (path, attr) -> attr.isDirectory();

try (var stream = Files.find(start,
                             Integer.MAX_VALUE,
                             matcher)) {

    stream.sorted()
          .forEach(System.out::println);
} catch (IOException e) {
  // ...
}
```

输出与使用`walk`相当，同样的假设——*深度优先*和非保证遇见顺序——也适用于`find`。真正的区别在于对当前元素的`BasicFileAttributes`的访问，这可能影响性能。如果需要根据文件属性进行过滤或匹配，使用`find`将节省从`Path`元素显式读取文件属性的操作，这可能会略微提高性能。然而，如果只需要`Path`元素而不需要访问其文件属性，则`walk`方法同样是一个很好的选择。

## 逐行读取文件

使用 Streams 轻松处理文件逐行读取的常见任务，该任务提供了`lines`方法。根据文件的`Charset`，有两种变体：

```java
static Stream<String> lines(Path path, ![1](img/1.png)
                            Charset cs) ![2](img/2.png)
                            throws IOException

static Stream<String> lines(Path path) ![1](img/1.png)
                      throws IOException
```

![1](img/#co_working_with_streams_CO4-1)

指向要读取的文件的`Path`。

![2](img/#co_working_with_streams_CO4-2)

文件的字符集。第二个变体默认为`StandardCharsets.UTF_8`。

###### 小贴士

尽管你可以使用任何想要的`Charset`，但在并行处理中会有性能差异。`lines`方法经过优化，适用于`UTF_8`、`US_ASCII`和`ISO_8859_1`。

让我们看一个简单的例子，统计托尔斯泰《战争与和平》中的单词数，如示例 7-4 所示。

##### 示例 7-4\. 统计《战争与和平》中的单词数

```java
var location = Paths.get("war-and-peace.txt"); ![1](img/1.png)

// CLEANUP PATTERNS ![2](img/2.png)
var punctuation = Pattern.compile("\\p{Punct}");
var whitespace  = Pattern.compile("\\s+");
var words       = Pattern.compile("\\w+");

try (Stream<String> stream = Files.lines(location)) { ![3](img/3.png)

  Map<String, Integer> wordCount =

           // CLEAN CONTENT ![4](img/4.png)
    stream.map(punctuation::matcher)
          .map(matcher -> matcher.replaceAll(""))
          // SPLIT TO WORDS ![5](img/5.png)
          .map(whitespace::split)
          .flatMap(Arrays::stream)
          // ADDITIONAL CLEANUP ![6](img/6.png)
          .filter(word -> words.matcher(word).matches())
          // NORMALIZE ![7](img/7.png)
          .map(String::toLowerCase)
          // COUNTING ![8](img/8.png)
          .collect(Collectors.toMap(Function.identity(),
                                    word -> 1,
                                    Integer::sum));
} catch (IOException e) {
  // ...
}
```

![1](img/#co_working_with_streams_CO5-1)

使用 Project Gutenberg 的*《战争与和平》*普通文本版本⁴，因此没有格式可能会影响单词计数。

![2](img/#co_working_with_streams_CO5-2)

正则表达式预编译以防止为每个元素重新编译。这种优化非常重要，因为为每个元素和`map`操作创建`Pattern`会迅速累积并影响整体性能。

![3](img/#co_working_with_streams_CO5-3)

`lines`调用返回文件的行作为元素的`Stream<String>`。`try-with-resources`块是必需的，因为 I/O 操作必须显式关闭，您将在“文件 I/O 流的注意事项”中了解更多。

![4](img/#co_working_with_streams_CO5-4)

需要移除标点符号，否则直接跟在标点符号旁边的相同单词将被计为不同的单词。

![5](img/#co_working_with_streams_CO5-5)

现在清理后的行根据空白字符分割，从而创建了一个`Stream<String[]>`。为了实际计数单词，`flatMap`操作将流扁平化为`Stream<String>`。

![6](img/#co_working_with_streams_CO5-6)

“word”匹配器是一个额外的清理和选择步骤，仅计数实际的单词。

![7](img/#co_working_with_streams_CO5-7)

将元素映射为小写确保不同大小写的单词被视为一个单词。

![8](img/#co_working_with_streams_CO5-8)

终端操作创建了一个`Map<String, Integer>`，以单词作为键，出现次数作为值。

流管道完成了既定任务，接管文件读取任务并逐行提供内容，使您可以集中精力处理代码的处理步骤。

我们将在第八章中重新讨论此特定示例，再次查看如何通过使用并行流显著改进这样一个常见任务。

## 文件 I/O 流的注意事项

使用流和文件 I/O 非常简单。然而，正如我之前提到的，有三个不寻常的方面。它们并不重要，不会减少使用基于流的文件 I/O 的可用性或有用性，尽管您需要注意它们：

+   需要关闭流

+   目录内容是弱一致的

+   元素顺序不保证

这些方面源于一般的 I/O 处理，并且在大多数与 I/O 相关的代码中都能找到，不仅限于流管道。

### 明确关闭流

在 Java 中处理资源，如文件 I/O，通常需要在使用后关闭它们。未关闭的资源可能会*泄漏*，这意味着垃圾收集器在资源不再需要或使用后无法回收其内存。处理流的 I/O 也是如此。这就是为什么你需要显式关闭基于 I/O 的流，至少与非 I/O 流相比如此。

`Stream<T>`类型通过`BaseStream`扩展了`java.io.AutoClosable`，因此关闭它的最简单方法是使用`try-with-resources`块，正如在“使用文件 I/O”部分和下面的代码中所见：

```java
try (Stream<String> stream = Files.lines(location)) {
  stream.map(...)
        ...
}
```

所有与`java.nio.file.Files`上的流相关的方法根据它们的签名都会抛出`IOException`，因此你需要以某种形式处理该异常。结合合适的`try-with-resources`块和适当的`catch`块可以一举解决这两个要求。

### 弱一致性的目录内容

`java.nio.file.Files`上的`list`、`walk`和`find`方法是*弱一致性*并且是*惰性*填充的。这意味着在流创建时并不会一次性扫描实际的目录内容以获得遍历期间的固定快照。在创建或遍历`Stream<Path>`后，文件系统的任何更新可能会或可能不会反映出来。

这种约束背后的推理很可能是出于性能和优化考虑。流管道应该是惰性顺序管道，没有区分它们的元素。文件树的固定快照将要求在流创建时收集所有可能的元素，而不是在由终端操作触发的实际流处理时进行惰性处理。

### 非保证元素顺序

流的惰性特性带来了文件 I/O 流的另一个方面，这可能是你不会预料到的。文件 I/O 流的遭遇顺序不能保证按照自然顺序（例如字母顺序）——因此，你可能需要额外的`sorted`中间操作来确保一致的元素顺序。这是因为流是由文件系统填充的，而文件系统不保证以有序的方式返回其文件和目录。

# 处理日期和时间

处理日期始终是具有许多边缘情况的挑战。幸运的是，Java 8 引入了一个*日期和时间 API*⁠⁵。其不可变性很好地适合于任何函数式代码，并且也提供了一些与流相关的方法。

## 查询时间类型

新的日期和时间 API 为任意属性提供了灵活和功能强大的查询接口。像大多数流操作一样，通过其参数将实际需要的逻辑注入到方法中，使方法本身成为具有更大通用性的更一般的支架：

```java
<R> R query(TemporalQuery<R> query);
```

通用签名允许查询任何类型，使其非常灵活：

```java
// TemporalQuery<Boolean> == Predicate<TemporalAccessor>

boolean isItTeaTime = LocalDateTime.now()
                                   .query(temporal -> {
                                     var time = LocalTime.from(temporal);
                                     return time.getHour() >= 16;
                                   });

// TemporalQuery<LocalTime> == Function<TemporalAccessor,Localtime>
LocalTime time = LocalDateTime.now().query(LocalTime::from);
```

实用类`java.time.temporal.TemporalQueries`提供了预定义查询，显示在 表 7-3 中，以消除自己创建常见查询的需求。

表 7-3\. `java.time.temporal.TemporalQueries` 中预定义的 `TemporalQuery<T>`

| `static` 方法 | 返回类型 |
| --- | --- |
| `chronology()` | `Chronology` |
| `offset()` | `ZoneOffset` |
| `localDate()` | `LocalDate` |
| `localTime()` | `LocalTime` |
| `precision()` | `TemporalUnit` |
| `zoneId()` | `ZoneId` |
| `zone()` | `ZoneId` |

显然，并非所有的时间 API 类型都支持每一种查询类型。例如，你无法从`Local…​`类型中获取`ZoneId`/`ZoneOffset`。每种方法都有详细的文档说明⁶，说明了它们支持的类型和预期的使用情况。

## LocalDate-Range Streams

Java 9 引入了 Stream 的能力，用于单个 JSR 310 类型 `java.time.LocalDate`，以创建 `LocalDate` 元素的连续范围。你不必担心不同日历系统的所有复杂性和边缘情况，以及日期计算实际上是如何执行的。日期和时间 API 将为你处理它们，通过提供一致且易于使用的抽象。

两个 `LocalDate` 实例方法创建了一个有序和连续的 Stream:

+   `Stream<LocalDate> datesUntil(LocalDate endExclusive)`

+   `Stream<LocalDate> datesUntil(LocalDate endExclusive, Period step)`

第一个变体相当于使用 `Period.ofDays(1)`。它们的实现不会溢出，这意味着任何元素加上 `step` *必须*在 `endExclusive` 之前。日期的方向也不是*仅限未来*。如果 `endExclusive` 在过去，你必须提供一个负的 `step` 来创建一个朝过去的 Stream。

# 使用 JMH 测量 Stream 性能

在整本书中，我都提到了 Java 的函数式技术和工具，比如 Streams，与*传统*方法相比会产生一定的开销，你必须考虑到这一点。这就是为什么使用基准测试来衡量 Stream 流水线的性能至关重要的原因。Streams 不是一个容易测试的目标，因为它们是多个操作的复杂流水线，背后有许多优化，这些优化取决于它们的数据和操作。

JVM 及其*即时*编译器可能很难进行基准测试和确定实际性能。这就是*Java 微基准测试工具包*的用武之地。

[JMH](https://openjdk.java.net/projects/code-tools/jmh/) 负责 JVM 预热、迭代和代码优化，这可能会淡化结果，使其更可靠，因此更适合用作评估的基线。它是基准测试的*事实标准*，并在 JDK 版本 12 中被包含在内⁷。

可用于 IDE 和构建系统的插件，例如 [Gradle](https://github.com/melix/jmh-gradle-plugin)、[IntelliJ](https://github.com/artyushov/idea-jmh-plugin)、[Jenkins](https://github.com/brianfromoregon/jmh-plugin) 或 [TeamCity](https://github.com/presidentio/teamcity-plugin-jmh)。

[JMH GitHub 仓库的示例目录](https://github.com/openjdk/jmh/blob/master/jmh-samples/src/main/java/org/openjdk/jmh/samples/)有很多文档完整的基准测试，解释了其使用的复杂性。

我不会进一步讨论如何在一般情况下对流或 lambda 进行基准测试，因为这超出了本章的范围，而且很容易占用整本书的空间。事实上，我建议你查看**《Java 优化》（Optimizing Java）**，作者是**本杰明·J·埃文斯**（Benjamin J Evans）、**詹姆斯·高夫**（James Gough）、**克里斯·纽兰德**（Chris Newland）⁸，以及**《Java 性能》（Java Performance）**，作者是**斯科特·奥克斯**（Scott Oaks）⁹，以了解更多关于基准测试和如何在 Java 中测量性能的信息。

# 更多关于收集器的信息

第六章介绍了收集器和相应的终端操作 `collect`，作为将流水线的元素聚合到新数据结构中的强大工具。实用类型 `java.util.stream.Collectors` 有大量的 `static` 工厂方法可以为几乎任何任务创建收集器，从简单的聚合到新的 `Collection` 类型，甚至更复杂的多步聚合流水线。这种更复杂的收集器是通过*下游收集器*的概念完成的。

收集器的一般思想很简单：将元素收集到一个新的数据结构中。如果你想要一个基于集合的类型，比如 `List<T>` 或 `Set<T>`，这是一个非常直接的操作。然而，在 `Map<K, V>` 的情况下，通常需要复杂的逻辑来获取一个正确形成的数据结构，以满足你的目标。

将一系列元素收集到基于键值的数据结构（如 `Map<K, V>`）可以通过多种方式完成，每种方式都有其挑战。例如，即使是简单的键值映射，每个键只有一个值，也存在处理键冲突的问题。但是，如果你想进一步转换 Map 的值部分，比如分组、缩减或分区，你需要一种方法来操作已收集的值。这就是下游收集器发挥作用的地方。

## 下游收集器

`java.util.stream.Collectors` 工厂方法提供的一些预定义收集器可以接受一个额外的收集器来操作*下游*元素。基本上，这意味着在主要收集器完成其工作之后，下游收集器对收集的值进行进一步的更改。这几乎就像是在之前收集的元素上工作的二级流水线。

下游收集器的典型任务包括：

+   转换

+   缩减

+   扁平化

+   过滤

+   组合收集器操作

本节的所有示例将使用以下 `User` 记录和 `users` 数据源：

```java
record User(UUID id,
            String group,
            LocalDateTime lastLogin,
            List<String> logEntries) { }

List<User> users = ...;
```

### 转换元素

使用 `Collectors.groupingBy` 方法将流元素分组到简单的键值映射中非常容易。然而，键值映射的值部分可能不是您需要的形式，可能需要额外的转换。

例如，通过其 `group` 对 `Stream<User>` 进行分组将创建一个 `Map<String, List<User>>`：

```java
Map<String, List<User>> lookup =
  users.stream()
       .collect(Collectors.groupingBy(User::group));
```

简单明了。

如果您不希望整个 `User` 而只是其 `id` 处于其中位置，该怎么办？您不能使用中间 `map` 操作在收集之前转换元素，因为您不再能真正访问 `User` 来实际分组它们。而是可以使用下游收集器来转换已收集的元素。这就是为什么有多个可用的 `groupingBy` 方法的原因，就像我们将在本节中使用的方法一样：

```java
Collector<T, ?, Map<K, D>> groupingBy(Function<? super T, ? extends K> classifier,
                                      Collector<? super T, A, D> downstream)
```

尽管此方法签名中的不同通用类型可能看起来令人生畏，但不要担心！让我们分解签名以更好地理解发生的情况。

列出了涉及的四种类型在 Table 7-4。

表 7-4\. `groupingBy` 的通用类型

| 通用类型 | 用于 |
| --- | --- |
| `T` | 收集之前流的元素类型。 |
| `K` | `Map` 结果的键类型。 |
| `D` | 由下游收集器创建的结果 `Map` 值部分的类型。 |
| `A` | 下游收集器的累加器类型。 |

如您所见，每种方法签名的类型表示整个过程的一部分。`classifier` 创建键，将类型为 `T` 的元素映射到键类型 `K`。下游收集器将类型为 `T` 的元素聚合到新的结果类型 `D` 中。因此，总体结果将是一个 `Map<K, D>`。

###### 提示

Java 的类型推断通常会为您匹配正确的类型，因此如果您只想使用这些复杂的通用方法而不是自己编写它们，您不必太在意实际的通用签名。如果发生类型不匹配并且编译器无法自动推导类型，请尝试使用 IDE 的帮助将操作逻辑重构为专用变量，以查看推断的类型。与一次性调整整个流水线相比，调整较小的代码块要容易得多。

本质上，每个接受额外下游收集器的收集器都包含原始逻辑 — 在本例中是键映射器 — 和下游收集器，影响映射到键的值。您可以将下游收集过程视为另一个被收集的流。不过，它只遇到主收集器关联的键的值。

让我们回到查找`User`组的查找映射。目标是创建一个`Map<String, Set<UUID>>`，将`User`组映射到一组不同的`id`实例。创建下游收集器的最佳方法是考虑实现目标所需的具体步骤，以及`java.util.stream.Collectors`的哪些工厂方法可以实现这些步骤。

首先，您需要`User`元素的`id`，这是一个映射操作。方法`Collector<T, ?, R> mapping(Function<? super T, ? extends U> mapper, Collector<? super U, A, R> downstream)`创建一个收集器，将收集的元素映射后传递给另一个收集器。需要另一个下游收集器的原因很简单；映射收集器的唯一目的是，您可能已经猜到的，*映射*元素。映射后的元素的实际收集超出了它的范围，因此委托给下游收集器。

其次，您希望将映射后的元素收集到一个`Set`中，可以通过`Collectors.toSet()`完成。

将收集器单独编写，可以使它们的意图和层次结构更加可见：

```java
// COLLECT ELEMENTS TO SET
Collector<UUID, ?, Set<UUID>> collectToSet = Collectors.toSet();

// MAP FROM USER TO UUID
Collector<User, ?, Set<UUID>> mapToId =
  Collectors.mapping(User::id,
                     collectToSet);

// GROUPING BY GROUP
Collector<User, ?, Map<String, Set<UUID>>> groupingBy =
  Collectors.groupingBy(User::group, mapToId);
```

正如我之前所说，通常可以让编译器推断类型并直接使用`Collectors`工厂方法。如果静态导入该类，甚至可以省略重复的`Collectors.`前缀。将所有收集器组合并在流管道中使用会导致简单直接的收集管道：

```java
import static java.util.stream.Collectors.*;

Map<String, Set<UUID>> lookup =
  users.stream()
       .collect(groupingBy(User::group,
                           mapping(User::id, toSet())));
```

结果类型也可以由编译器推断。尽管如此，我更喜欢显式声明，以便更好地传达由流管道返回的类型。

另一种方法是将主要的下游收集器保持为变量，以使`collect`调用更简单。这样做的缺点是，在使用 lambda 表达式而不是方法引用的情况下，必须帮助编译器推断出正确的类型。

```java
var collectIdsToSet = Collectors.mapping(User::id, ![1](img/1.png)
                                         Collectors.toSet());

// LAMBDA ALTERNATIVE

var collectIdsToSetLambda = Collectors.mapping((User user) -> user.id(), ![2](img/2.png)
                                               Collectors.toSet());

Map<String, Set<UUID>> lookup =
  users.stream()
       .collect(Collectors.groupingBy(User::group,
                                      collectIdsToSet)); ![3](img/3.png)
```

![1](img/#co_working_with_streams_CO6-1)

方法引用告诉编译器流的元素类型，因此下游收集器也知道它。

![2](img/#co_working_with_streams_CO6-2)

`mapper`的 lambda 变体需要知道要处理的类型。您可以为 lambda 参数提供显式类型，也可以将`var`替换为更复杂的泛型`Collector<T, A , R>`签名。

![3](img/#co_working_with_streams_CO6-3)

由于变量名的存在，`collect`调用仍然很表达力。如果某些聚合操作经常使用，应考虑将它们重构为带有工厂方法的辅助类型，类似于`java.util.stream.Collectors`。

### 减少元素

有时需要减少操作而不是聚合操作。设计减少的下游收集器的一般方法与前一节相同：定义您的总体目标，分解成必要的步骤，最后创建下游收集器。

对于这个示例，不要为`group`按`id`创建查找 Map，让我们统计每个`User`的`logEntries`。

总体目标是计算每个`User`元素的日志条目数。所需步骤是获取`User`的日志计数并将它们求和到最终的总数。

您可以使用`Collectors.mapping`工厂方法与另一个下游 Collector 来实现这个目标：

```java
var summingUp = Collectors.reducing(0, Integer::sum);

var downstream =
  Collectors.mapping((User user) -> user.logEntries().size(),
                     summingUp);

Map<UUID, Integer> logCountPerUserId =
  users.stream()
       .collect(Collectors.groupingBy(User::id, downstream));
```

与要求映射和减少下游 Collector 齐头并进不同，您可以使用其他`Collector.reduce`变体之一，其中包括一个`mapper`：

```java
Collector<T, ?, U> reducing(U identity,
                            Function<? super T, ? extends U> mapper,
                            BinaryOperator<U> op)
```

这种`reduce`变体除了一个种子值（`identity`）和归约操作（`op`）外，还需要一个`mapper`来将`User`元素转换为所需的值：

```java
var downstream =
  Collectors.reducing(0,                                       // identity
                      (User user) -> user.logEntries().size(), // mapper
                      Integer::sum);                           // op

Map<UUID, Integer> logCountPerUserId =
  users.stream()
       .collect(Collectors.groupingBy(User::id, downstream));
```

像`reduce`中间操作一样，使用一个减少的 Collector 用于下游操作是一种非常灵活的工具，能够将多个步骤合并为单个操作。选择方法，是使用多个下游 Collectors 还是单一的归约，取决于个人偏好和收集过程的整体复杂性。然而，如果你只需要对数字求和，`java.util.stream.Collectors`类型还提供了更专门的变体：

```java
var downstream =
  Collectors.summingInt((User user) -> user.logEntries().size());

Map<UUID, Integer> logCountPerUserId =
  users.stream()
       .collect(Collectors.groupingBy(User::id, downstream));
```

`summing` Collector 可用于通常的原始类型（`int`、`long`、`float`）。除了对数字求和外，您还可以计算平均值（前缀为`averaging`）或仅计数元素使用`Collectors.counting()`。

### 展开集合

在流中处理基于集合的元素通常需要一个`flatMap`中间操作将集合“展平”，以便将其返回到离散元素中以进一步处理管道，否则您将得到像`List<List<String>>`这样的嵌套集合。对流的收集过程也是如此。

通过它们的`group`将所有`logEntries`分组将导致一个`Map<String, List<List<String>>>`，这很可能不是你想要的。Java 9 添加了一个具有内置展平功能的新预定义 Collector：

```java
static Collector<T, ?, R> flatMapping(Function<T, Stream<U>> mapper,
                                      Collector<U, A, R> downstream)
```

就像其他添加的 Collector，`Collectors.filtering(…​)`，我在“过滤元素”中讨论过，如果作为唯一的 Collector 使用，与显式的`flatMap`中间操作没有任何优势。但是，用于多级减少，如`groupingBy`或`partitionBy`，它让您可以访问原始流元素*并*允许展平收集的元素：

```java
var downstream =
  Collectors.flatMapping((User user) -> user.logEntries().stream(),
                         Collectors.toList());

Map<String, List<String>> result =
  users.stream()
       .collect(Collectors.groupingBy(User::group, downstream));
```

与转换和减少 Collectors 一样，当需要使用展平下游 Collector 时，您很快就会掌握它。如果流管道的结果类型不符合您的预期，您很可能需要一个下游 Collector 来解决问题，可以通过使用`Collectors.mapping`或`Collectors.flatMapping`。

### 过滤元素

过滤流元素是几乎任何流管道的重要部分，借助中间的`filter`操作完成。Java 9 增加了一个新的预定义收集器，具有内置的过滤能力，将元素过滤步骤直接移动到积累过程之前：

```java
static <T, A, R> Collector<T,?,R> filtering(Predicate<T> predicate,
                                            Collector<T, A, R> downstream)
```

单独使用时，它与中间的`filter`操作没有区别。作为下游收集器，它的行为与`filter`大不相同，特别是在分组元素时很容易看出：

```java
import static java.util.stream.Collectors.*;

var startOfDay = LocalDate.now().atStartOfDay();

Predicate<User> loggedInToday =
  Predicate.not(user -> user.lastLogin().isBefore(startOfDay));

// WITH INTERMEDIATE FILTER

Map<String, Set<UUID>> todaysLoginsByGroupWithFilterOp =
  users.stream()
       .filter(loggedInToday)
       .collect(groupingBy(User::group,
                           mapping(User::id, toSet())));

// WITH COLLECT FILTER

Map<String, Set<UUID>> todaysLoginsByGroupWithFilteringCollector =
  users.stream()
       .collect(groupingBy(User::group,
                           filtering(loggedInToday,
                                     mapping(User::id, toSet()))));
```

你可能期望有相同的结果，但操作顺序会导致不同的结果：

首先中间过滤，其次分组

使用中间的`filter`操作会在任何收集发生之前移除任何不需要的元素。因此，在生成的`Map`中不包括今天未登录的用户组，如图 7-1 所示。

![使用“首先过滤，然后分组”来分组元素](img/afaj_0701.png)

###### 图 7-1\. 使用“首先过滤，然后分组”来分组元素

首先分组，然后向下过滤

如果没有中间的`filter`操作，`groupingBy`收集器将会遇到所有`User`元素，而不管它们的最后登录日期。下游收集器 — `Collectors.filtering` — 负责过滤元素，因此返回的`Map`仍包含所有用户组，而不考虑最后的登录情况。元素的流动如图 7-2 所示。

![使用“首先分组，然后向下过滤”来分组元素](img/afaj_0702.png)

###### 图 7-2\. 使用“首先分组，然后向下过滤”来分组元素

哪种方法更可取取决于您的需求。首先进行过滤可以返回可能的最少键值对，但首先进行分组可以让您访问所有`Map`键及其（可能）为空的值。

### 组合收集器

我想讨论的最后一个收集器是`Collectors.teeing`，在 Java 12 中添加，与其他不同之处在于它一次接受两个下游收集器，并将两者的结果合并为一个。

###### 注意

*teeing* 的名称来源于最常见的管道配件之一 — T 型管配件 — 其形状类似大写字母 T。

流的元素首先通过两个下游收集器，因此`BiFunction`可以将两个结果合并为第二步，如图 7-3 所示。

![Teeing 收集器的元素流动](img/afaj_0703.png)

###### 图 7-3\. Teeing 收集器的元素流动

想象一下，您想知道有多少用户及其中多少从未登录。如果没有 `teeing` 操作，您将不得不遍历元素两次：一次用于总体计数，另一次用于计算从未登录的用户。这两个计数任务可以由专用的 Collectors `counting` 和 `filtering` 表示，因此您只需遍历元素一次，并让 `teeing` 在管道末端执行这两个计数任务。然后，使用 `BiFunction<Long, Long>` 将结果合并到新的数据结构 `UserStats` 中。示例 7-5 展示了如何实现它。

##### 示例 7-5\. 查找最小和最大登录日期

```java
record UserStats(long total, long neverLoggedIn) { ![1](img/1.png)
  // NO BODY
}

UserStats result =
  users.stream()
       .collect(Collectors.teeing(Collectors.counting(), ![2](img/2.png)
                Collectors.filtering(user -> user.lastLogin() == null, ![3](img/3.png)
                                     Collectors.counting()),
                UserStats::new)); ![4](img/4.png)
```

![1](img/#co_working_with_streams_CO7-1)

由于 Java 缺乏动态元组，本地 Record 类型被用作结果类型。

![2](img/#co_working_with_streams_CO7-2)

第一个下游 Collector 用于计算所有元素的数量。

![3](img/#co_working_with_streams_CO7-3)

第二个下游 Collector 首先进行过滤，并使用附加的下游 Collector 计算剩余元素的数量。

![4](img/#co_working_with_streams_CO7-4)

`UserStats` 构造函数的方法引用用作两个下游 Collector 结果的合并函数。

与许多功能性添加类似，如果您主要来自面向对象的背景，`teeing` Collector 最初可能会显得有些奇怪。单独使用 `for` 循环以两个离散变量计数可以达到相同的结果。不同之处在于 `teeing` Collector 如何从 Stream 管道及其整体优势和功能可能性中受益，而不仅仅是终端操作本身。

## 创建您自己的 Collector

辅助类型 `java.util.stream.Collectors` 在撰写本书时的最新 LTS Java 版本 17 中提供了超过 44 个预定义的工厂方法。这些方法涵盖了大多数常见用例，特别是在联合使用时。有时您可能需要一个定制的、更具上下文特定性的 Collector，比预定义的更易于使用。这样一来，您还可以在类似 `Collectors` 的自定义辅助类中共享这些特定的 Collectors。

回顾第六章中提到，Collectors 借助四种方法聚合元素：

+   `Supplier<A> supplier()`

+   `BiConsumer<A, T> accumulator()`

+   `BinaryOperator<A> combiner()`

+   `Function<A, R> finisher()`

我之前未提及的 `Collector` 接口的一种方法是 `Set<Characteristics> characteristics()`。与 Streams 类似，Collectors 具有一组特性，允许不同的优化技术。当前提供的三个选项列在表 7-5 中。

表 7-5\. 可用的 java.util.Collector.Characteristics

| 特征 | 描述 |
| --- | --- |
| `CONCURRENT` | 支持并行处理 |
| `IDENTITY_FINISH` | 完成器是恒等函数，返回累加器本身。在这种情况下，只需要进行类型转换，而不是调用完成器本身。 |
| `UNORDERED` | 表示流元素的顺序不一定会保留。 |

为了更好地理解这些部分是如何结合在一起的，我们将重现一个现有的收集器，`Collectors.joining(CharSequence delimiter)`，它将 `CharSequence` 元素连接起来，以 `delimiter` 参数分隔。示例 7-6 显示了如何使用 `java.util.StringJoiner` 实现 `Collector<T, A, R>` 接口，以实现所需的功能。

##### 示例 7-6。用于连接字符串元素的自定义 Collector

```java
public class Joinector implements Collector<CharSequence, // T
                                            StringJoiner, // A
                                            String> {     // R

  private final CharSequence delimiter;

  public Joinector(CharSequence delimiter) {
    this.delimiter = delimiter;
  }

  @Override
  public Supplier<StringJoiner> supplier() {
    return () -> new StringJoiner(this.delimiter); ![1](img/1.png)
  }

  @Override
  public BiConsumer<StringJoiner, CharSequence> accumulator() {
    return StringJoiner::add; ![2](img/2.png)
  }

  @Override
  public BinaryOperator<StringJoiner> combiner() {
    return StringJoiner::merge; ![3](img/3.png)
  }

  @Override
  public Function<StringJoiner, String> finisher() {
    return StringJoiner::toString; ![4](img/4.png)
  }

  @Override
  public Set<Characteristics> characteristics() {
    return Collections.emptySet(); ![5](img/5.png)
  }
}
```

![1](img/#co_working_with_streams_CO8-1)

`StringJoiner` 类型是完美的可变结果容器，因为它具有公共 API 和分隔符支持。

![2](img/#co_working_with_streams_CO8-2)

向容器中添加新元素的累加逻辑就像使用适当的方法引用一样简单。

![3](img/#co_working_with_streams_CO8-3)

通过方法引用也可以实现多个容器的组合逻辑。

![4](img/#co_working_with_streams_CO8-4)

最后一步，将结果容器转换为实际结果，是通过容器的 `toString` 方法完成的。

![5](img/#co_working_with_streams_CO8-5)

`Joinector` 不具有任何可用的 Collector 特性，因此返回一个空的 `Set`。

足够简单，但这仍然是大量的代码，功能非常少，主要是返回方法引用。幸运的是，`Collector` 上有方便的工厂方法 `of`，可以简化代码：

```java
Collector<CharSequence, StringJoiner, String> joinector =
  Collector.of(() -> new StringJoiner(delimiter), // supplier
               StringJoiner::add,                 // accumulator
               StringJoiner::merge,               // combiner
               StringJoiner::toString);           // finisher
```

这个简短版本相当于之前接口的完整实现。

###### 注意

`Collector.of(…​)` 方法的最后一个参数并不总是可见，如果未设置，它是 Collector 特性的一组可变参数。

创建自己的收集器应保留用于自定义结果数据结构或简化特定领域任务。即便如此，你应该首先尝试使用可用的收集器和下游收集器的组合来实现结果。Java 团队投入了大量时间和知识，为你提供了安全且易于使用的通用解决方案，这些解决方案可以组合成相当复杂且强大的解决方案。然后，如果你有一个可用的收集器，你仍然可以将其重构为辅助类，使其可重用且易于阅读。

# 关于（顺序）流的最终想法

在我看来，Java Streams API 是一个绝对的游戏改变者，因此了解可用操作和使用流处理不同任务的方式非常重要。流提供了一种流畅、简洁且直接的数据处理方法，如果需要，还可以并行处理，正如你将在第八章中了解到的那样。然而，它们并不是设计来替代现有的循环结构，而是作为其补充。

作为 Java 开发者，你应该掌握的关于 Streams 最重要的技能是在使用足够的流管道来提高代码可读性和合理性之间找到平衡，同时不要通过忽略传统的循环结构来牺牲性能。

并非每个循环都需要成为流。然而，并非每个流都最好成为循环。当您习惯使用流进行数据处理时，您将更容易找到两种数据处理方法之间的健康平衡。

# 要点

+   Stream API 提供了多种可能性来创建流，从类似传统循环结构的迭代方法到特定类型的专门变体，如文件 I/O 或新的日期和时间 API。

+   与功能接口类似，大多数流及其操作通过专门类型支持原始类型，以减少自动装箱的数量。这些专门的变体在需要时可以为您提供性能优势，但会限制可用的操作。但是，您可以在管道中在原始和非原始流之间随时切换，以获得两个世界的优势。

+   下游收集器可以通过多种方式影响集合过程，例如转换或过滤，以将结果操作为任务所需的表示形式。

+   如果一组下游收集器的组合无法满足您的任务需求，您可以退而自行创建收集器。

¹ 正如在“Project Valhalla 和专用泛型”中讨论的那样，*Project Valhalla*将允许像原始类型这样的值类型用作泛型类型边界。然而，截至本书撰写之时，尚不清楚其具体可用日期。

² 例如，[`Random#ints()`的文档](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Random.xhtml#ints())指出该方法被实现为`Random.ints(Long.MAX_VALUE)`的等效方法。

³ 有关*Project Valhalla*的更多信息，请参见侧边栏“Project Valhalla 和专用泛型”。

⁴ Project Gutenberg 提供多个免费版本的[《战争与和平》](https://www.gutenberg.org/ebooks/2600)。

⁵ [Java *日期与时间 API*（JSR310）](https://openjdk.java.net/projects/threeten) 的目标是用一套全面的类型替换 `java.util.Date`，以一种不可变的方式提供一致和完整的日期和时间处理方法。

⁶ [官方文档 `java.time.temporal.TemporalQueries`](https://docs.oracle.com/javase/8/docs/api/java/time/temporal/TemporalQueries.xhtml) 详细列出了每个预定义 `TemporalQuery` 支持的类型。

⁷ JMH 也支持 Java 12 之前的版本，但您需要手动包含其两个依赖项：[JMH Core](https://mvnrepository.com/artifact/org.openjdk.jmh/jmh-core) 和 [JMH Generators/Annotation Processors](https://mvnrepository.com/artifact/org.openjdk.jmh/jmh-generator-annprocess)。

⁸ Evans, Benjamin J., Gough, James, Newland, Chris. 2018\. “优化 Java。” O’Reilly Media. 978-1-492-02579-5

⁹ Oaks, Scott. 2020\. “Java 性能，第二版。” O’Reilly Media. ISBN 978-1-492-05611-9.
