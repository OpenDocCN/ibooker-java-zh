## 附录 A. 使用 Java 8 函数式特性

当 Java 8 发布时，Oracle 将其定位为向更函数式编程迈出的一步。在 Oracle 的“JDK 8 新特性”笔记中列出的函数式友好特性包括以下内容：

+   “Lambda 表达式，这是一种新的语言特性，已在本版本中引入。它们使您能够将功能作为方法参数处理，或将代码作为数据。Lambda 表达式允许您更紧凑地表达单方法接口（称为函数式接口）的实例。”这是函数式范式的一个重要方面。

+   “方法引用为已命名的现有方法提供了易于阅读的 lambda 表达式。”这句话的后半部分可能指的是“现有方法”，因为未命名的的方法是不存在的。

+   “类型注解提供了在任何使用类型的地方应用注解的能力，而不仅仅是声明上。”

+   “改进的类型推断。”

+   “新 `java.util.stream` 包中的类提供了 Stream API，以支持对元素流进行函数式操作。Stream API 集成到 Collections API 中，使得诸如顺序或并行 map-reduce 转换等操作成为可能。”

你可以在 Oracle 的原始“JDK 8 新特性”文档中阅读这些声明（以及许多与函数式编程无关的其他声明）([`mng.bz/27na`](http://mng.bz/27na))。

在这次演示中，Oracle 没有列出几个元素，并省略了一个重要的事实：

+   `Function` 包

+   `Optional` 类

+   `CompletableFuture` 类

+   事实是，大多数集合通过添加 `stream()` 方法进行了修改，这使得它们可以转换为 `Stream` 实例。

所有这些，包括 `Optional`、`Stream` 和 `CompletableFuture` 都是单子结构（有关这意味着什么，请参阅附录 B appendix B）的事实，清楚地表明 Oracle 的意图是使使用 Java 进行函数式编程更容易。

在这本书中，我大量使用了这些功能友好的特性，例如 lambda 表达式和函数式接口，并间接受益于更好的类型推断和扩展的类型注解。然而，我没有使用其他函数式元素，如 `Optional` 或 `Stream`。在本附录中，我将解释原因。

### A.1\. `Optional` 类

`Optional` 类类似于你在第六章 chapter 6 中开发的 `Option` 类。它旨在解决 `null` 引用的问题，但对函数式编程帮助不大。显然，某些地方出了问题。`Optional` 类有一个 `get` 方法，如果有“封装”的值，它将返回该值，否则返回 `null`。当然，调用此方法违背了原始目标。

如果你想要使用`Optional`类，你应该记住永远不要调用`get`。你可能会反对，因为`Option`类有`getOrThrow`方法，虽然它永远不会返回`null`，但如果没有数据可用，它将抛出异常。但这个方法是受保护的，并且不能从外部扩展这个类。这有很大的不同。这个方法与`List`中的`head`或`tail`方法等价：它们不应该从外部调用。

此外，`Optional`类与`Option`有相同的局限性：`Optional`可以用于真正的可选数据，但通常数据的缺失是由于错误。`Optional`，就像`Option`一样，不允许你携带错误原因，所以它只适用于真正的可选数据，这意味着当数据缺失的原因很明显时，例如从一个映射中返回一个值，或者字符串中字符的位置。如果映射的`get(key)`方法不返回值，无论是`null`还是空的`Optional`，都应该很明显，键没有找到。如果`indexOf(char)`方法不返回值或空的`Optional`，应该意味着字符不在字符串中。

但即使是这样也不一定正确。映射的`get(key)`方法可能会返回`null`，因为`null`值被存储在那个键下。或者它可能不返回任何值，因为键是`null`（假设`null`不是一个有效的键）。`indexOf(char)`方法也可能因为许多原因不返回值，例如负数参数。在这些情况下返回`Optional`并不能表明错误的性质。此外，这个`Optional`与可能产生错误的其它方法返回的值组合起来会很困难。

由于所有这些原因，`Optional`，就像我们的`Option`版本一样，是无用的。这就是我们开发`Return`类型的原因，你可以用它来表示可选数据的缺失以及错误。

### A.2\. 流

流是 Java 8 的另一个新元素，它混合了三个不同的概念：

+   惰性集合

+   单子集合

+   自动并行化

这三个概念是独立的，没有明显的理由说明为什么它们被混合在一起。不幸的是，就像许多其他旨在做几件不同事情的工具一样，它们在每个方面都不够理想。

单子数据结构对于函数式编程至关重要，而 Java 集合不是单子的。你可以通过简单地在新添加的`stream()`方法上调用这些集合来创建这样的结构。如果流有所有必要的函数处理方法，这可能是一个可接受的解决方案。但流被设计成能够自动从串行切换到并行处理。这个过程相当复杂，这可能是为什么一些重要的方法没有在流中实现。例如，Java 8 流没有`takeWhile`或`dropWhile`方法。

这可能是一个为了获得自动并行化访问的合理代价，但即使这个特性实际上也不是真正可用的。（这个问题在 Java 9 中得到了解决。）所有并行流都使用一个包含与计算机上物理线程数量相同但减去一个（主线程）的线程数量的单个 fork/join 池。任务被分配到池中每个工作线程的等待队列中。一旦一个线程耗尽了其任务队列，它就会从其他线程“窃取”工作。主线程本身也通过从工作线程“窃取”工作来参与。

整体结果并不理想，因为当然，计算机可能还有许多其他任务要同时执行。想想一个 Java EE 应用程序正在接收来自客户端的请求。这些请求已经并行处理，所以进一步并行化每个请求几乎没有好处。通常，在这种情况下，将没有任何好处。

更糟糕的是，因为所有并行流共享同一个 fork/join 池，如果一个流阻塞，它可能会阻塞所有其他流！你可以为每个流使用一个特定的池，但这有点复杂，并且只有在使用较小大小的池（即较少的线程）时才应该这样做。如果你对这样的技术感兴趣，请查看我在 DZone 上发布的以下文章：

+   “Java 8 中的问题，第三部分：流和并行流” ([`dzone.com/articles/whats-wrong-java-8-part-iii`](https://dzone.com/articles/whats-wrong-java-8-part-iii))

+   “Java 8 中的问题，第七部分：流再次出现” ([`dzone.com/articles/whats-wrong-java-8-part-vii`](https://dzone.com/articles/whats-wrong-java-8-part-vii))

Java 8 流最糟糕的可能就是它们只能使用一次。一旦对它们调用了一个终端方法，它们就再也不能使用了。任何进一步的访问都会产生异常。这有两个后果。

第一个问题是无法进行记忆化。你无法像第二次访问流那样，只能创建一个新的流。结果是，如果值是延迟评估的，它们将不得不再次评估。

第二个后果甚至更糟：Java 8 的流不能用于理解模式。想象一下，你想编写一个函数来验证毕达哥拉斯关系 a² + b² = c²，使用类似于下面的`Triple`类实现：

```
public class Triple {
  public final int a;
  public final int b;
  public final int c;
  Triple(int a, int b, int c) {
    this.a = a;
    this.b = b;
    this.c = c;
  }

  @Override
  public String toString() {
    return String.format("(%s,%s,%s)", a, b, c);
  }
}
```

在命令式 Java 中，`pyths`方法可以如下实现：

```
static List<Triple> pyths(int n) {
  List<Triple> result = new ArrayList<>();
  for (int a = 1; a <= n; a++) {
    for (int b = 1; b <= n; b++) {
      for (int c = 1; c <= n; c++) {
        if (a * a + b * b == c * c) {
          result.add(new Triple (a, b, c));
        }
      }
    }
  }
  return result;
}
```

“功能”版本，使用流，应该看起来像这样：

```
static Stream<Triple> pyths(int n) {
  Stream<Integer> stream = IntStream.rangeClosed(1, n).boxed();
  return stream.flatMap(a -> stream
      .flatMap(b -> stream
          .flatMap(c -> a * a + b * b == c * c
              ? Stream.of(new Triple (a, b, c))
              : Stream.empty())));
}
```

不幸的是，在 Java 8 中，这将产生以下异常：

```
java.lang.IllegalStateException: stream has already been operated upon or closed
```

相比之下，你可以使用你在第五章中开发的`List`类来编写这个示例。

Java 8 流的另一个限制是折叠是一个终端操作，这意味着折叠（在 Java 8 流中称为 `reduce`）将导致评估所有流元素。为了理解差异，回想一下你在第九章中开发的 `Stream.foldRight` 方法。使用此方法，你可以编写如下 `identity` 函数的实现：

```
public Stream<A> identity() {
  return foldRight(Stream::empty, a -> b -> cons(() -> a, b));
}
```

此方法完全惰性，允许你用它来实现 `map`、`flatMap` 以及许多其他方法。这在 Java 8 流中是完全不可能的。

这是否意味着你永远不应该使用 Java 8 流？绝对不是。Java 8 流在性能是最重要的标准时是一个很好的选择，尤其是在你需要处理原始数据时。然而，在生产环境中，通常应避免使用并行流。对于大多数函数式使用，真正的函数式流是一个更好的选择。

如果你想要（或需要）在函数式上下文中使用 Java 8 `Stream`，请注意，尽管 `Stream` 类型有一个 `reduce` 方法（实际上有三个版本的这个方法）用于折叠，但这并不是折叠流的最佳方式。折叠应该使用 `Collector` 实现来完成。`Collector` 是一个接口，它定义了五个方法：

```
@Override
public Supplier<A> supplier();

@Override
public BiConsumer<A, T> accumulator();

@Override
public BinaryOperator<A> combiner();

@Override
public Function<List<List<T>>, List<List<T>>> finisher();

@Override
public Set<Characteristics> characteristics();
```

`supplier` 方法返回一个 `Supplier<A>` 用于身份元素。`accumulator` 方法返回一个 `BiConsumer<A, T>`，这是折叠函数的非功能性替代品。相应的折叠函数将是 `BiFunction <A, T, A>`，它将一个元素与当前结果组合。而不是返回结果，消费者应该将其存储在某个地方（在 `Collector` 中）。换句话说，它是一个基于状态变异的折叠版本。`finisher` 是一个可选函数，它将在最终结果上应用。最后，`characteristics` 返回 `Collector` 使用的特征集，用于优化其工作。有三个可能的特征——`CONCURRENT`、`IDENTITY_FINISH` 和 `UNORDERED`：

+   `CONCURRENT` 表示 `accumulator` 函数支持并发，可以被多个线程使用。

+   `IDENTITY_FINISH` 表示 `finisher` 函数是身份函数，因此可以被忽略。

+   `UNORDERED` 表示流是无序的，这为并行化提供了更多自由。

这里是一个将 `Stream<String>` 折叠成 `List<List<String>>` 的 `Collector` 示例，模拟了给定最大长度行上的单词分组。首先，你定义一个泛型的 `GroupingCollector`：

```
import java.util.ArrayList;
import java.util.List;
import java.util.function.*;
import java.util.stream.Collector;

import static java.util.stream.Collector.Characteristics.IDENTITY_FINISH;

public class GroupingCollector<T> {

  private final BiPredicate<List<T>, T> p;

  public GroupingCollector(BiPredicate<List<T>, T> p) {
    this.p = p;
  }

  public void accumulator(List<List<T>> llt, T t) {
    if (! llt.isEmpty()) {
      List<T> last = llt.get(llt.size() - 1);
      if (p.test(last, t)) {
        llt.get(llt.size() - 1).add(t);
      } else {
        addNewList(llt, t);
      }
    } else {
      addNewList(llt, t);
    }
  }

  public List<List<T>> combiner(List<List<T>> list1, List<List<T>> list2) {
    List<List<T>> result = new ArrayList<>();
    result.addAll(list1);
    result.addAll(list2);
    return result;
  }

  public static <T> void addNewList(List<List<T>> llt, T t) {
    List<T> list = new ArrayList<>();
    list.add(t);
    llt.add(list);
  }

  public Collector<T, List<List<T>>, List<List<T>>> collector() {
    return Collector.of(ArrayList::new, this::accumulator, this::combiner, IDENTITY_FINISH);
  }
}
```

然后，你创建一个特定的字符串分组收集器：

```
import java.util.List;
import java.util.function.BiPredicate;
import java.util.stream.Collector;

public class StringGrouperCollector {

  private StringGrouperCollector() {
  }

  public static Collector<String, List<List<String>>, List<List<String>>> getInstance(int length) {
    BiPredicate<List<String>, String> p = (ls, s) -> length(ls) + s.length() <= length;
    return new GroupingCollector<>(p).collector();
  }

  public static int length(List<String> list) {
    int length = 0;
    for (String s : list) {
      length += s.length();
    }
    return length;
  }
}
```

最后，你可以创建客户端代码来测试收集器：

```
public class Client {

  public static void main(String...args) {

   List<String> words2 = Arrays.asList("Once", "upon", "a", "time", "there", "was", "a", "prince",
        "who", "lived", "in", "a", "magnificent", "castle");
     words2.stream().collect(StringGrouperCollector.getInstance(20))
         .forEach(System.out::println);
  }
}
```

该程序打印以下内容：

```
[Once, upon, a, time, there]
[was, a, prince, who, lived, in]
[a, magnificent, castle]
```

原则与折叠操作完全相同，抽象了流元素迭代的迭代过程。
