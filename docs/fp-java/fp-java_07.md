## 第八章. 高级列表处理

***本章涵盖的内容***

+   使用记忆化加速列表处理

+   组合 `List` 和 `Result`

+   在列表上实现索引访问

+   展开列表

+   自动并行列表处理

在 第五章 中，你创建了你的第一个数据结构，单链表。在那个时刻，你没有掌握所有使它成为数据处理完整工具所需的技巧。你特别缺少的一个有用工具是表示产生可选数据或可能产生错误的操作的方式。在 第六章 和 第七章 中，你学习了如何表示可选数据和错误。在本章中，你将学习如何将产生可选数据或错误的操作与列表组合。 

你还开发了一些远非最优的函数，例如 `length`，我说你最终会学到这些操作的更有效的方法。在本章中，你将学习如何实现这些技术。你还将学习如何自动并行化一些列表操作，以利用当今计算机的多核架构。

### 8.1. 长度问题

折叠列表涉及从一个值开始，并依次将其与列表的每个元素组合。这显然需要与列表长度成比例的时间。有没有办法使这个操作更快？或者，至少有没有办法让它看起来更快？

作为折叠应用的一个例子，你在练习 5.9 中在 `List` 中创建了一个 `length` 方法，其实现如下：

```
public int length() {
  return foldRight(this, 0, x -> y -> y + 1);
}
```

在这个实现中，列表是通过一个操作折叠的，该操作由将 1 加到结果中组成。起始值是 `0`，列表中每个元素的值被简单地忽略。这就是为什么你可以为所有列表使用相同的定义。因为列表元素被忽略，所以列表元素的类型无关紧要。

你可以将前面的操作与计算整数列表和的操作进行比较：

```
public static Integer sum(List<Integer> list) {
  return list.foldRight(0, x -> y -> x + y);
}
```

这里的主要区别在于 `sum` 方法只能与整数一起工作，而 `length` 方法适用于任何类型。请注意，`foldRight` 只是一种抽象递归的方式。列表的长度可以定义为空列表的 0，非空列表的长度加 1。同样，整数列表的和可以递归地定义为空列表的 0，非空列表的头元素加上尾部的和。

有其他操作可以以这种方式应用于列表，其中一些操作与列表元素的类型无关：

+   列表的哈希码可以通过简单地将其元素的哈希码相加来计算。因为哈希码是一个整数（至少对于 Java 对象来说是这样），这个操作不依赖于对象的类型。

+   列表的字符串表示，由`toString`方法返回，可以通过组合列表元素的`toString`表示来计算。再次强调，元素的实际类型是不相关的。

一些操作可能依赖于元素类型的某些特性，而不是具体的类型本身。例如，一个返回列表最大元素的`max`方法只需要类型是`Comparable`或`Comparator`。

#### 8.1.1\. 性能问题

所有这些方法都可以使用折叠来实现，但这样的实现有一个主要的缺点：计算结果所需的时间与列表的长度成比例。想象一下，你有一个大约一百万个元素的列表，你想要检查它的长度。计数元素似乎是唯一的方法（这就是基于折叠的`length`方法所做的事情）。但如果你在向列表中添加元素直到它达到一百万个元素，你当然不会在添加每个元素后计数。

在这种情况下，你会在某个地方保留元素的计数，并且每次向列表中添加元素时，都会将这个计数加一。如果你从一个非空列表开始，可能只需要计数一次，但这就足够了。这种技术就是你在第四章中学到的：记忆化。问题是，你可以在哪里存储记忆化的值？答案是显而易见的：在列表本身。

#### 8.1.2\. 记忆化的好处

维护列表中元素数量的计数将花费一些时间，因此向列表中添加元素会比不保持计数时稍微慢一些。这看起来像是你在时间与时间之间进行交易。如果你构建一个包含 1,000,000 个元素的列表，你将失去 1,000,000 倍于添加一个元素到计数所需的时间。然而，作为补偿，获取列表长度所需的时间将接近 0（并且显然是恒定的）。也许在增加计数时损失的总时间将与调用`length`时的收益相等。但一旦你多次调用`length`，这种收益就绝对明显了。

#### 8.1.3\. 记忆化的缺点

记忆化可以将一个在 O(*n*)时间（与元素数量成比例的时间）内工作的函数转换为 O(1)时间（恒定时间）。这是一个巨大的好处，尽管它有一个时间成本，因为它使得元素的插入稍微慢一些。但减慢插入通常不是一个大问题。

一个更加重要的问题是内存空间的增加。实现原地修改的数据结构没有这个问题。在可变列表中，没有什么能阻止你将列表长度记忆化为一个可变整数，它只需要 32 位。但是，对于不可变列表，你必须在每个元素中记忆化长度。很难知道确切的尺寸增加，但如果单链表的每个节点大小约为 40 字节（对于节点本身），加上头和尾的两个 32 位引用（在 32 位 JVM 上），这将导致每个元素约为 100 字节。在这种情况下，添加长度会导致增加略超过 30%。如果记忆化的值是引用，比如记忆化`Comparable`对象列表的`max`或`min`，结果也会相同。在 64 位 JVM 上，由于一些优化，计算甚至更加困难，但你可以理解这个概念。

| |
| --- |

##### 对象引用的大小

关于 Java 7 和 Java 8 中对象引用大小的更多信息，请参阅 Oracle 关于压缩 Oops 的文档([`mng.bz/TjY9`](http://mng.bz/TjY9))和 JVM 性能增强([`mng.bz/8X0o`](http://mng.bz/8X0o))。

| |
| --- |

是否在数据结构中使用记忆化（memoization）取决于你。对于经常被调用且不为其结果创建新对象的函数来说，这可能是一个有效的选项。例如，`length`和`hashCode`函数返回整数，而`max`和`min`函数返回对已存在对象的引用，因此它们可能是很好的候选者。另一方面，`toString`函数创建新的字符串，这些字符串需要被记忆化，这可能会造成巨大的内存空间浪费。另一个需要考虑的因素是函数的使用频率。`length`函数可能比`hashCode`函数使用得更频繁，因为使用列表作为映射键不是一种常见的做法。

#### 练习 8.1

创建`length`方法的记忆化版本。在`List`类中的签名将是

```
public abstract int lengthMemoized();
```

#### 解决方案 8.1

在`Nil`类中的实现与未记忆化的`length`方法完全相同：

```
public int lengthMemoized() {
  return 0;
}
```

要实现`Cons`版本，你必须首先将记忆化字段添加到类中，并在构造函数中初始化它：

```
private final int length;
private Cons(A head, List<A> tail) {
  this.head = head;
  this.tail = tail;
  this.length = tail.length() + 1;
}
```

然后你可以实现`lengthMemoized`方法，简单地返回长度：

```
public int lengthMemoized() {
  return length;
}
```

这个版本将比原始版本快得多。一个有趣的现象是`length`和`isEmpty`方法之间的关系。你可能倾向于认为`isEmpty`等价于`length == 0`，但从逻辑角度来看，尽管这是真的，但在实现和性能上可能会有很大的差异。

注意，以相同的方式（尽管是静态方法）缓存`Comparable`列表中的最大值或最小值是可能的，但在你想从列表中移除最大或最小值的情况下，这并没有帮助。最小或最大元素通常通过优先级来检索元素。在这种情况下，元素的`compareTo`方法会比较它们的优先级。缓存优先级会让你立即知道哪个元素具有最高优先级，但这帮助不大，因为你通常需要移除相应的元素。对于这样的用例，你需要一个不同的数据结构，你将在第十一章（chapter 11）中学习如何创建它。

#### 8.1.4\. 实际性能

正如我所说，决定是否应该缓存`List`类的一些函数取决于你。一些实验应该能帮助你做出决定。在创建一个包含 1,000,000 个整数的列表前后测量可用内存大小，使用缓存时内存增加非常小。尽管这种测量方法不是很精确，但在两种情况下（有或没有缓存），可用内存的平均减少量约为 22 MB，介于 20 MB 和 25 MB 之间。这表明理论上的 4 MB（1,000,000 x 4 字节）的增加并不像你预期的那么显著。另一方面，性能的提升是巨大的。在没有缓存的情况下，请求长度十次可能需要超过 200 毫秒。有了缓存，时间几乎为 0（短到无法用毫秒来测量）。

注意，尽管添加一个元素会增加成本（将长度加一并存储结果），但移除一个元素没有成本，因为尾部长度已经被缓存。

如果不希望使用缓存，另一种方法是优化`length`方法。而不是使用折叠，你可以求助于命令式风格，使用循环和局部可变变量。以下是从 Scala `List`类借用的`length`实现：

```
public int length() {
  List<A> these = this;
  int len = 0;
  while (!these.isEmpty()) {
    len += 1;
    these = these.tail();
  }
  return len;
}
```

虽然它在风格上看起来不太像函数式编程，但这种实现与函数式编程的定义完全兼容。它是一个纯函数，没有任何外部世界可观察的效果。主要问题是它只比基于折叠的实现快五倍，而对于非常大的列表，缓存实现可以快数百万倍。

### 8.2\. 列表与结果的组合

在上一章中，你看到了`Result`和`List`是非常相似的数据结构，主要区别在于它们的基数，但它们共享一些最重要的方法，例如`map`、`flatMap`，甚至`foldLeft`和`foldRight`。

你看到了如何用列表来组合列表，以及结果与结果的组合。现在，你将看到结果如何与列表组合。

#### 8.2.1\. 列表方法返回结果

到目前为止，你已经注意到我试图避免直接访问结果和列表的元素。如果列表是`Nil`，访问列表的头部或尾部将抛出异常，而在函数式编程中抛出异常是可能发生的最糟糕的事情之一。但是你看到，通过提供一个用于失败或空结果的情况的默认值，你可以安全地访问`Result`中的值。你能否在访问列表的头部时做同样的事情？不是完全一样，但你可以返回一个`Result`。

#### 练习 8.2

在`List<A>`中实现一个`headOption`方法，该方法将返回一个`Result<A>`。

##### 提示

在`List`中使用以下抽象方法声明，并在每个子类中实现它：

```
public abstract Result<A> headOption();
```

注意，方法被命名为`headOption`是为了表明一个值是可选的，尽管你将使用`Result`作为类型。

#### 解决方案 8.2

`Nil`类的实现返回`Empty`：

```
public Result<A> headOption() {
  return Result.empty();
}
```

`Cons`实现返回一个包含头值的`Success`：

```
public Result<A> headOption() {
  return Result.success(head);
}
```

#### 练习 8.3

创建一个返回列表中最后一个元素的`Result`的`lastOption`方法。

##### 提示

不要使用显式递归，而是尝试构建你在第五章中开发的方法。你应该能够在`List`类中定义一个单一的方法。

#### 解决方案 8.3

一个简单的解决方案是使用显式递归：

```
public Result<A> lastOption() {
  return isEmpty()
      ? Result.empty()
      : tail().isEmpty()
          ? Result.success(head())
          : tail().lastOption();
}
```

这种解决方案有几个问题。它是基于栈的递归，所以你应该将其转换为基于堆的，另外你还需要处理空列表的情况，其中`tail().lastOption()`会抛出 NPE。

但你可以简单地使用折叠，它为你抽象了递归！你需要做的只是创建正确的折叠函数。你需要始终保留存在的最后一个值。这可能就是你要使用的函数：

```
Function<Result<A>, Function<A, Result<A>>> f =
                                   x -> y -> Result.success(y);
```

或者使用方法引用：

```
Function<Result<A>, Function<A, Result<A>>> f =
                                   x -> Result::success;
```

然后你只需要使用`Result.Empty`作为恒等元`foldLeft`列表：

```
public Result<A> lastOption() {
  return foldLeft(Result.empty(), x -> Result::success);
}
```

#### 练习 8.4

你能否在`List`类中用单个实现替换`headOption`方法？这种实现的好处和缺点是什么？

#### 解决方案 8.4

有可能创建这样的实现：

```
public Result<A> headOption() {
  return foldRight(Result.empty(), x -> y -> Result.success(x));
}
```

唯一的好处是如果你喜欢这种方式，它会更有趣。在设计`last-Option`实现时，你知道你必须遍历列表以找到最后一个元素。要找到第一个元素，你不需要遍历列表。在这里使用`fold-Right`与反转列表然后遍历结果以找到最后一个元素（这是原始列表的第一个元素）完全相同。这并不高效！顺便说一句，这正是`lastOption`方法找到最后一个元素的方式：反转列表并取结果的第一个元素。所以除了有趣之外，实际上没有理由使用这种实现。

#### 8.2.2. 将 List<Result>转换为 Result<List>

当一个列表包含一些计算的结果时，它通常是一个 `List<Result>`。例如，将函数从 `T` 映射到 `Result<U>` 应用到一个 `T` 的列表上，将产生一个 `Result<U>` 的列表。这些值通常需要与接受 `List<T>` 作为参数的函数组合。这意味着你需要一种方法将结果 `List<Result<U>>` 转换成一个 `List<U>`，这与 `flatMap` 方法中的扁平化类似，但有一个巨大的区别：涉及两种不同的数据类型：`List` 和 `Result`。你可以应用几种策略来完成这个转换：

+   丢弃所有失败或空结果，并从剩余的成功列表中生成一个 `U` 的列表。如果列表中没有成功，结果可以简单地包含一个空的 `List`。

+   丢弃所有失败或空结果，并从剩余的成功列表中生成一个 `U` 的列表。如果列表中没有成功，结果将是一个 `Failure`。

+   决定所有元素都必须是成功，整个操作才能成功。如果所有元素都是成功，则使用值构造一个 `U` 的列表，并作为 `Success<List<U>>` 返回，否则返回 `Failure<List<U>>`。

第一个解决方案对应于所有结果都是可选的结果列表。第二个解决方案意味着列表中至少有一个成功，结果才是一个成功。第三个解决方案对应于所有结果都是必需的情况。

#### 练习 8.5

编写一个名为 `flattenResult` 的方法，它接受一个 `List<Result<A>>` 作为其参数，并返回一个包含原始列表中所有成功值的 `List<A>`，忽略失败和空值。这将是 `List` 中的一个静态方法，其签名如下：

```
public static <A> List<A> flattenResult(List<Result<A>> list)
```

尽量不要使用显式递归，而是组合 `List` 和 `Result` 类的方法。

##### 提示

为该方法选择的名字是你要做的指示。

#### 解决方案 8.5

要解决这个练习，你可以使用 `foldRight` 方法将列表折叠成一个由函数生成的列表列表。每个 `Success` 将被转换成一个包含单个元素的列表，其中包含值，而每个 `Failure` 或 `Empty` 将被转换成一个空列表。以下是该函数：

```
Function<Result<A>, Function<List<List<A>>, List<List<A>>>> f =
                    x -> y -> y.cons(x.map(List::list).getOrElse(list()));
```

一旦你有了这个函数，你可以用它来将列表向右折叠，生成一个包含值的列表列表，其中一些元素是空列表：

```
list.foldRight(list(), f)
```

剩下的工作就是将结果 `flatten`。完整的方法如下：

```
public static <A> List<A> flattenResult(List<Result<A>> list) {
  return flatten(list.foldRight(list(), x -> y ->
                y.cons(x.map(List::list).getOrElse(list()))));
  }
```

请注意，这不是最有效的方法。这主要是一个练习。

#### 练习 8.6

编写一个 `sequence` 函数，它将 `List<Result<T>>` 合并成一个 `Result<List<T>>`。如果原始列表中的所有值都是 `Success` 实例，则它将是一个 `Success<List<T>>`，否则是一个 `Failure<List<T>>`。以下是它的签名：

```
public static <A> Result<List<A>> sequence(List<Result<A>> list)
```

##### 提示

再次使用 `foldRight` 方法而不是显式递归。你还需要在 `Result` 类中定义的 `map2` 方法。

#### 解决方案 8.6

这里是使用 `foldRight` 和 `map2` 的实现方法：

```
public static <A> Result<List<A>> sequence(List<Result<A>> list) {
  return list.foldRight(Result.success(List.list()),
                  x -> y -> Result.map2(x, y, a -> b -> b.cons(a)));
}
```

注意，这个实现将空的 `Result` 处理成 `Failure`，并返回它遇到的第一个失败案例，这可能是一个 `Failure` 或一个 `Empty`。这可能是你需要的，也可能不是。为了坚持 `Empty` 表示可选数据的想法，你需要首先过滤列表以移除 `Empty` 元素：

```
public static <A> Result<List<A>> sequence2(List<Result<A>> list) {
  return list.filter(a -> a.isSuccess() || a.isFailure())
      .foldRight(Result.success(List.list()),
                 x -> y -> Result.map2(x, y, a -> b -> b.cons(a)));
}
```

最终，你应该将移除空元素的操作抽象成一个单独的方法，在 `List` 类中。但在本书的其余部分，我们仍将 `Empty` 视为 `sequence` 方法上下文中的 `Failure`。

#### 练习 8.7

定义一个更通用的 `traverse` 方法，它遍历一个 `A` 的列表，同时应用一个从 `A` 到 `Result<B>` 的函数，并生成一个 `Result<List<B>>`。以下是它的签名：

```
public static <A, B> Result<List<B>> traverse(List<A> list,
                                           Function<A, Result<B>> f)
```

然后用 `traverse` 方法定义 `sequence` 的新版本。

##### 提示

不要使用递归。优先使用 `foldRight` 方法，它为你抽象了递归。

#### 解决方案 8.7

首先定义 `traverse` 方法：

```
public static <A, B> Result<List<B>> traverse(List<A> list,
                                              Function<A, Result<B>> f) {
  return list.foldRight(Result.success(List.list()),
      x -> y -> Result.map2(f.apply(x), y, a -> b -> b.cons(a)));
}
```

然后你可以用 `traverse` 方法来重新定义 `sequence` 方法：

```
public static <A> Result<List<A>> sequence(List<Result<A>> list) {
  return traverse(list, x -> x);
}
```

### 8.3. 抽象化常见的列表使用场景

许多 `List` 数据类型的常见使用场景值得抽象化，这样你就不必一次又一次地重复相同的代码。你经常会发现自己发现新的使用场景，这些场景可以通过组合基本函数来实现。你永远不应该犹豫将这些使用场景作为 `List` 类中的新函数来包含。以下练习展示了几个最常见的使用场景。

#### 8.3.1. 压缩和解压缩列表

压缩（Zipping）是将两个列表组合成一个列表的过程，通过合并相同索引的元素。解压缩（Unzipping）是相反的过程，通过“解构”元素来从单个列表中生成两个列表，例如从一个点列表中生成 `x` 和 `y` 坐标的两个列表。

#### 练习 8.8

编写一个 `zipWith` 方法，它结合两个不同类型的列表的元素，并使用一个函数参数生成一个新的列表。以下是它的签名：

```
public static <A, B, C> List<C> zipWith(List<A> list1, List<B> list2,
                                        Function<A, Function<B, C>> f)
```

此方法接受一个 `List<A>` 和一个 `List<B>`，通过一个从 `A` 到 `B` 到 `C` 的函数，生成一个 `List<C>`。

##### 提示

压缩应该限制在最短列表的长度内。

#### 解决方案 8.8

对于这个练习，你必须使用显式递归，因为递归必须在两个列表上同时进行。你没有任何抽象可以使用。以下是解决方案：

```
public static <A, B, C> List<C> zipWith(List<A> list1, List<B> list2,
                                        Function<A, Function<B, C>> f) {
  return zipWith_(list(), list1, list2, f).eval().reverse();
}
private static <A, B, C> TailCall<List<C>> zipWith_(List<C> acc,
          List<A> list1, List<B> list2, Function<A, Function<B, C>> f) {
  return list1.isEmpty() || list2.isEmpty()
      ? ret(acc)
      : sus(() -> zipWith_(
          new Cons<>(f.apply(list1.head()).apply(list2.head()), acc),
          list1.tail(), list2.tail(), f));
}
```

`zipWith_` 辅助方法使用空列表作为起始累加器被调用。如果两个参数列表中的任意一个为空，递归停止，并返回当前的累加器。否则，通过将函数应用于两个列表的头部值来计算一个新的值，并递归地使用两个参数列表的尾部调用辅助函数。

#### 练习 8.9

前一个练习是通过对两个列表的元素按索引匹配来创建一个列表。编写一个`product`方法，该方法将生成从两个列表中取出的所有可能元素组合的列表。换句话说，给定两个列表`list("a", "b", "c")`和`list("d", "e", "f")`以及字符串连接，两个列表的乘积应该是`List("ad", "ae", "af", "bd", "be", "bf", "cd", "ce", "cf")`。

##### 提示

对于这个练习，你不需要使用显式递归。

#### 解决方案 8.9

解决方案与你在第七章中用来组合`Result`的推导模式类似。唯一的区别是它产生的组合数量与列表中元素数量的乘积一样多，而对于组合`Result`，组合的数量总是限制为 1。

```
public static <A, B, C> List<C> product(List<A> list1, List<B> list2,
                                        Function<A, Function<B, C>> f) {
  return list1.flatMap(a -> list2.map(b -> f.apply(a).apply(b)));
}
```

注意，这样可以通过组合超过两个列表。唯一的问题是组合的数量将以指数级增长。

`product`和`zipWith`的常见用例之一是使用组合函数的构造函数。以下是一个使用`Tuple`构造函数的示例：

```
List.product(List.list(1, 2, 3), List.list(4, 5, 6),
                                    x -> y -> new Tuple<>(x, y));
List.zipWith(List.list(1, 2, 3), List.list(4, 5, 6),
                                    x -> y -> new Tuple<>(x, y));
```

第一行将产生由两个列表的元素构建的所有可能元组的列表：

```
[(1,4), (1,5), (1,6), (2,4), (2,5), (2,6), (3,4), (3,5), (3,6), NIL]
```

第二行将只产生由具有相同索引的元素构建的元组列表：

```
[(1,4), (2,5), (3,6), NIL]
```

当然，你可以使用任何类的任何构造函数。（Java 对象实际上是具有特殊名称的元组。）

#### 练习 8.10

编写一个`unzip`静态方法，将元组列表转换为元组列表。以下是它的签名：

```
<A, B> Tuple<List<A>, List<B>> unzip(List<Tuple<A, B>> list)
```

##### 提示

不要使用显式递归。一个简单的`foldRight`调用就可以完成工作。

#### 解决方案 8.10

你需要使用一个包含两个空列表的元组作为恒等元素来`foldRight`列表：

```
public static <A,B> Tuple<List<A>, List<B>> unzip(List<Tuple<A, B>> list) {
  return list.foldRight(new Tuple<>(list(), list()),
               t -> tl -> new Tuple<>(tl._1.cons(t._1), tl._2.cons(t._2)));
}
```

#### 练习 8.11

将`unzip`函数泛化，使其可以将任何类型的列表转换为元组列表，给定一个接受列表类型对象作为其参数并产生元组的函数。例如，给定一个`Payment`实例的列表，你应该能够生成一个包含用于支付的信用的列表和一个包含支付金额的列表的元组。将此方法实现为`List`的实例方法，其签名如下：

```
<A1, A2> Tuple<List<A1>, List<A2>> unzip(Function<A, Tuple<A1, A2>> f)
```

##### 提示

解决方案与练习 8.10 的解决方案几乎相同。

#### 解决方案 8.11

重要的是，函数的结果将被使用两次。为了不将函数应用两次，你必须使用多行 lambda：

```
public <A1, A2> Tuple<List<A1>, List<A2>> unzip(Function<A,
                                                Tuple<A1, A2>> f) {
  return this.foldRight(new Tuple<>(list(), list()), a -> tl -> {
    Tuple<A1, A2> t = f.apply(a);
    return new Tuple<>(tl._1.cons(t._1), tl._2.cons(t._2));
  });
}
```

#### 8.3.2. 通过索引访问元素

单链表不是访问其元素的索引的最佳结构，但有时有必要使用索引访问。像往常一样，你应该将此类过程抽象为`List`方法。

#### 练习 8.12

编写一个`getAt`方法，该方法接受一个索引作为其参数，并返回相应的元素。该方法在索引超出范围的情况下不应抛出异常。

##### 提示

这次，从一个显式的递归版本开始。然后尝试回答以下问题：

+   是否可以使用折叠操作完成？是左折叠还是右折叠？

+   为什么显式的递归版本更好？

+   你能找到解决问题的方法吗？

#### 解答 8.12

显式的递归解决方案很简单：

```
public Result<A> getAt(int index) {
  return index < 0 || index >= length()
      ? Result.failure("Index out of bound")
      : getAt_(this, index).eval();
}

private static <A> TailCall<Result<A>> getAt_(List<A> list, int index) {
    return index == 0
              ? TailCall.ret(Result.success(list.head()))
              : TailCall.sus(() -> getAt_(list.tail(), index - 1));
}
```

首先，你可以检查索引是否为正且小于列表长度。如果不是，就返回一个`Failure`。否则，调用辅助方法递归地处理列表。此方法检查索引是否为 0。如果是，则返回列表的头部。否则，它将递归地调用自身在列表的尾部，并使用递减的索引。

这看起来是最好的可能递归解决方案。是否可以使用折叠操作？是的，可以，而且应该是一个左折叠。但解决方案很棘手：

```
public Result<A> getAt(int index) {
  Tuple<Result<A>, Integer> identity =
               new Tuple<>(Result.failure("Index out of bound"), index);

  Tuple<Result<A>, Integer> rt = index < 0 || index >= length()
      ? identity
      : foldLeft(identity, ta -> a -> ta._2 < 0
            ? ta
            : new Tuple<>(Result.success(a), ta._2 - 1));
  return rt._1;
}
```

首先，你必须定义恒等值。因为这个值必须同时持有结果和索引，所以它将是一个包含`Failure`情况的`Tuple`。然后你可以检查索引的有效性。如果发现无效，将临时结果（`rt`）设置为`identity`。否则，使用返回已计算结果（`ta`）的函数向左折叠，如果索引小于 0，或者如果否则返回一个新的`Success`。

这个解决方案可能看起来更聪明，但实际上并非如此，原因有三：

+   它的可读性远低。这可能具有主观性，所以这取决于你决定。

+   你必须使用一个中间结果（`rt`），因为 Java 无法推断正确的类型。如果你不相信我，尝试在最后一行将`rt`替换为其值。

+   它效率较低，因为它会在找到搜索到的值之后继续折叠整个列表。

#### 练习 8.13（困难且可选）

找到一个解决方案，使得基于折叠的版本在找到结果后立即终止。

##### 提示

你需要为这个特殊版本提供`foldLeft`，以及一个特殊的`Tuple`版本。

#### 解答 8.13

首先，你需要一个特殊的`foldLeft`版本，其中可以在找到折叠操作的吸收元素（或“零”元素）时退出折叠。想象一下你想要通过乘法折叠的整数列表。乘法的吸收元素是 0。以下是`List`类中短路（或退出）版本的`foldLeft`声明：

```
public abstract <B> B foldLeft(B identity, B zero,
                                           Function<B, Function<A, B>> f);
```

| |
| --- |

##### 零元素

通过类比，任何操作的吸收元素有时被称为“零”，但请记住，它并不总是等于 0。0 值只是乘法的吸收元素。对于正整数的加法，它将是无穷大。

| |
| --- |

这里是`Cons`的实现：

```
@Override
public <B> B foldLeft(B identity, B zero, Function<B, Function<A, B>> f) {
  return foldLeft(identity, zero, this, f).eval();
}

private <B> TailCall<B> foldLeft(B acc, B zero, List<A> list,
                                 Function<B, Function<A, B>> f) {
  return list.isEmpty() || acc.equals(zero)
      ? ret(acc)
      : sus(() -> foldLeft(f.apply(acc).apply(list.head()),
                                                zero, list.tail(), f));
}
```

如你所见，唯一的区别是，如果累加器的值被发现在是“零”，递归就会停止，并返回累加器。

现在你需要一个零值用于 fold。零值是一个 `Tuple<Result<A, Integer>`，其中 `Integer` 的值为 `-1`（第一个小于 0 的值）。你能使用标准的 `Tuple` 吗？不，你不能，因为它必须有一个特殊的等于方法，当整数值相等时返回 `true`，无论 `Result<A>` 是什么。完整的方法如下：

```
public Result<A> getAt(int index) {

  class Tuple<T, U> {

    public final T _1;
    public final U _2;

    public Tuple(T t, U u) {
      this._1 = Objects.requireNonNull(t);
      this._2 = Objects.requireNonNull(u);
    }

    @Override
    public boolean equals(Object o) {
      if (!(o.getClass() == this.getClass()))
        return false;
      else {
        @SuppressWarnings("rawtypes")
        Tuple that = (Tuple) o;
        return _2.equals(that._2);
      }
    }
  }

  Tuple<Result<A>, Integer> zero =
              new Tuple<>(Result.failure("Index out of bound"), -1);
  Tuple<Result<A>, Integer> identity =
              new Tuple<>(Result.failure("Index out of bound"), index);
  Tuple<Result<A>, Integer> rt = index < 0 || index >= length()
      ? identity
      : foldLeft(identity, zero, ta -> a -> ta._2 < 0
                    ? ta
                    : new Tuple<>(Result.success(a), ta._2 - 1));
  return rt._1;
}
```

注意，我已经省略了 `hashCode` 和 `toString` 方法，以使代码更短。

现在 fold 将在找到搜索到的元素后自动停止。当然，你可以使用新的 `foldLeft` 方法来逃逸任何带有零元素的计算。（记住：零，不是 0。）

#### 8.3.3\. 分割列表

有时候你需要将列表在特定位置分割成两部分。尽管单链表对于这种操作来说远非理想，但实现起来相对简单。分割列表有几个有用的应用，其中之一是使用多个线程并行处理其部分。

#### 练习 8.14

编写一个 `splitAt` 方法，该方法接受一个 `int` 作为参数，并在给定位置分割列表，返回两个列表。不应该有任何 `IndexOutOfBound-Exception`。相反，一个小于 `0` 的索引应该被视为 `0`，一个大于最大值的索引应该被视为索引的最大值。

##### 提示

使方法显式递归。

#### 解答

设计一个显式递归的解法很简单：

```
public Tuple<List<A>, List<A>> splitAt(int index) {
  return index < 0
      ? splitAt(0)
      : index > length()
          ? splitAt(length())
          : splitAt(list(), this.reverse(), this.length() - index).eval();
}

private TailCall<Tuple<List<A>, List<A>>> splitAt(List<A> acc,
                                                  List<A> list, int i) {
  return i == 0 || list.isEmpty()
      ? ret(new Tuple<>(list.reverse(), acc))
      : sus(() -> splitAt(acc.cons(list.head()), list.tail(), i - 1));
}
```

注意，第一个方法使用递归来调整索引的值。然而，不需要使用 `TailCall`，因为这个方法最多只会递归一次。第二个方法与 `getAt` 方法非常相似，不同之处在于列表首先被反转。该方法累积元素直到达到索引位置，因此累积的列表是正确的顺序，但剩余的列表需要反转回来。

#### 练习 8.15（如果你已经完成了练习 8.13，那么这个练习就不会那么难）

你能想到一个使用 fold 而不是显式递归的实现吗？

##### 提示

一个遍历整个列表的实现很简单。一个只遍历到找到索引为止的列表实现则要复杂得多，并且需要一个带有逃逸的新特殊版本的 `foldLeft`，返回逃逸值和列表的其余部分。

#### 解答 8.15

一个遍历整个列表的解决方案可能如下：

```
public Tuple<List<A>, List<A>> splitAt(int index) {
  int ii = index < 0 ? 0 : index >= length() ? length() : index;
  Tuple3<List<A>, List<A>, Integer> identity =
                         new Tuple3<>(List.list(), List.list(), ii);
  Tuple3<List<A>, List<A>, Integer> rt =
         foldLeft(identity, ta -> a -> ta._3 == 0
               ? new Tuple3<>(ta._1, ta._2.cons(a), ta._3)
               : new Tuple3<>(ta._1.cons(a), ta._2, ta._3 - 1));
  return new Tuple<>(rt._1.reverse(), rt._2.reverse());
}
```

fold 的结果累积在第一个列表累加器中，直到达到索引（在调整索引值以避免索引越界之后）。一旦找到索引，列表遍历继续，但剩余的值累积在第二个列表累加器中。

这个实现的一个问题是，通过在第二个列表累加器中累积剩余值，你反转了列表的这一部分。不仅不应该需要遍历列表的剩余部分，而且在这里它被做了两次：一次是为了以相反的顺序累积，一次是为了最终反转结果。为了避免这种情况，你应该修改特殊的“逃逸”版本的 `foldLeft`，使其不仅返回逃逸结果（吸收元素或零元素），还返回未受影响的列表的其余部分。为了实现这一点，你必须更改签名以返回一个 `Tuple`：

```
public abstract <B> Tuple<B, List<A>> foldLeft(B identity, B zero,
                                          Function<B, Function<A, B>> f);
```

然后你需要更改 `Nil` 类的实现：

```
@Override
public <B> Tuple<B, List<A>> foldLeft(B identity, B zero,
                                      Function<B, Function<A, B>> f) {
  return new Tuple<>(identity, list());
}
```

最后，你必须将 `Cons` 实现改为返回列表的剩余部分：

```
@Override
public <B> Tuple<B, List<A>> foldLeft(B identity, B zero,
                                      Function<B, Function<A, B>> f) {
  return foldLeft(identity, zero, this, f).eval();
}

private <B> TailCall<Tuple<B, List<A>>> foldLeft(B acc, B zero,
                         List<A> list, Function<B, Function<A, B>> f) {
  return list.isEmpty() || acc.equals(zero)
      ? ret(new Tuple<>(acc, list))
      : sus(() -> foldLeft(f.apply(acc).apply(list.head()),
                                               zero, list.tail(), f));
}
```

现在，你可以使用这个特殊的 `foldLeft` 方法重写 `splitAt` 方法：

```
public Tuple<List<A>, List<A>> splitAt(int index) {

  class Tuple3<T, U, V> {
    public final T _1;
    public final U _2;
    public final V _3;

    public Tuple3(T t, U u, V v) {
      this._1 = Objects.requireNonNull(t);
      this._2 = Objects.requireNonNull(u);
      this._3 = Objects.requireNonNull(v);
    }

    @Override
    public boolean equals(Object o) {
      if (!(o.getClass() == this.getClass()))
        return false;
      else {
        @SuppressWarnings("rawtypes")
        Tuple3 that = (Tuple3) o;
        return _3.equals(that._3);
      }
    }
  }

  Tuple3<List<A>, List<A>, Integer> zero =
                                      new Tuple3<>(list(), list(), 0);
  Tuple3<List<A>, List<A>, Integer> identity =
                                      new Tuple3<>(list(), list(), index);
  Tuple<Tuple3<List<A>, List<A>, Integer>, List<A>> rt = index <= 0
        ? new Tuple<>(identity, this)
        : foldLeft(identity, zero, ta -> a -> ta._3 < 0
                ? ta
                : new Tuple3<>(ta._1.cons(a), ta._2, ta._3 - 1));
  return new Tuple<>(rt._1._1.reverse(), rt._2);
}
```

这里，你同样需要一个特定的 `Tuple3` 类，它有一个特殊的 `equals` 方法，当第三个元素相等时返回 `true`，而不考虑前两个元素。请注意，第二个结果列表不需要反转。


**何时不使用 fold**

只因为可以使用 fold 并不意味着你应该这样做。前面的练习只是练习而已。作为一个函数式库的设计者，你需要选择最有效的实现方式。

一个函数式库必须有一个函数式接口，并且必须遵守函数式编程的要求，这意味着所有函数都必须是真正的函数，没有副作用，并且都必须遵守引用透明性。库内部发生的事情无关紧要。在像 Java 这样的命令式语言中，函数式库可以与函数式语言的编译器相提并论。编译后的代码总是命令式的，因为这是计算机能理解的。函数式库提供了更多的选择。一些函数可能以函数式风格实现，而另一些则以命令式风格实现；这无关紧要。当以命令式方式实现时，分割单链表或通过索引查找元素比以函数式方式实现要容易得多，也快得多，因为单链表并不适合这种操作。

最函数式的方法可能不是基于 fold 实现这些函数，而是根本不实现它们。如果你需要具有这些函数的函数式实现的结构，最好的做法是创建特定的结构，正如你将在第十章中看到的。第十章。


#### 8.3.4\. 搜索子列表

列表的一个常见用途是搜索以确定一个列表是否包含在另一个（更长的）列表中。换句话说，你想要知道一个列表是否是另一个列表的子列表。

#### 练习 8.16

实现一个 `hasSubList` 方法来检查一个列表是否是另一个列表的子列表。例如，列表 (3, 4, 5) 是列表 (1, 2, 3, 4, 5) 的子列表，但不是列表 (1, 2, 4, 5, 6) 的子列表。将其实现为一个具有以下签名的静态方法：

```
public static <A> boolean hasSubsequence(List<A> list, List<A> sub)
```

##### 提示

你首先必须实现一个`startsWith`方法来确定列表是否以子列表开头。一旦完成，你将递归地测试这个方法，从列表的每个元素开始。

#### 解决方案 8.16

可以显式地实现一个`startsWith`方法，如下所示：

```
public static <A> Boolean startsWith(List<A> list, List<A> sub) {
  return sub.isEmpty()
      ? true
      : list.isEmpty()
          ? false
          : list.head().equals(sub.head())
              ? startsWith(list.tail(), sub.tail())
              : false;
}
```

这是一个基于栈的版本，可以使用`TailCall`转换为基于堆的版本：

```
public static <A> Boolean startsWith(List<A> list, List<A> sub) {
  return startsWith_(list, sub).eval();
}

public static <A> TailCall<Boolean> startsWith_(List<A> list,
                                                List<A> sub) {
  return sub.isEmpty()
      ? ret(Boolean.TRUE)
      : list.isEmpty()
          ? ret(Boolean.FALSE)
          : list.head().equals(sub.head())
              ? sus(() -> startsWith_(list.tail(), sub.tail()))
              : ret(Boolean.FALSE);
}
```

从那里，实现`hasSubList`是直接的：

```
public static <A> boolean hasSubList(List<A> list, List<A> sub) {
  return hasSubList_(list, sub).eval();
}

public static <A> TailCall<Boolean> hasSubList_(List<A> list, List<A> sub){
  return list.isEmpty()
      ? ret(sub.isEmpty())
      : startsWith(list, sub)
          ? ret(true)
          : sus(() -> hasSubList_(list.tail(), sub));
}
```

#### 8.3.5. 用于处理列表的杂项函数

可以开发出许多其他有用的函数来处理列表。以下练习将为你提供在这个领域的一些实践。请注意，提出的解决方案当然不是唯一的。你可以自由地发明自己的。

#### 练习 8.17

创建一个`groupBy`方法，该方法接受一个从`A`到`B`的函数作为参数，并返回一个`Map`，其中键是函数应用于列表中每个元素的输出，值是与每个键对应的元素列表。换句话说，给定如下`Payment`列表，

```
public class Payment {

  public final String name;
  public final int amount;

  public Payment(String name, int amount) {
    this.name = name;
    this.amount = amount;
  }
}
```

以下代码应该创建一个包含（键/值）对的`Map`，其中每个键是一个名称，相应的值是相应人员做出的`Payment`列表：

```
Map<String, List<Payment>> map = list.groupBy(x -> x.name);
```

##### 提示

使用前几章中的函数式`Map`包装器。这次，先尝试创建一个命令式版本，然后基于折叠创建一个函数式版本。你更喜欢哪一个？

#### 解决方案 8.17

这里是一个命令式版本的示例。对此没有太多可说的，因为它只是传统的命令式代码，带有局部可变状态：

```
public <B> Map<B, List<A>> groupByImperative(Function<A, B> f) {
  List<A> workList = this;
  Map<B, List<A>> m = Map.empty();
  while (!workList.isEmpty()) {
    final B k = f.apply(workList.head());
    List<A> rt = m.get(k).getOrElse(list()).cons(workList.head());
    m = m.put(k, rt);
    workList = workList.tail();
  }
  return m;
}
```

注意，这个实现是完全功能性的，因为从方法外部看不到任何状态变化。但风格相当命令式，使用了一个`while`循环和局部变量。

这里是一个更函数式风格的版本，使用折叠操作：

```
public <B> Map<B, List<A>> groupBy(Function<A, B> f) {
  return foldRight(Map.empty(), t -> mt -> {
    final B k = f.apply(t);
    return mt.put(k, mt.get(k).getOrElse(list()).cons(t));
  });
}
```

选择你喜欢的风格由你决定。显然，第二个版本更紧凑。但主要优势是它更好地表达了意图。`groupBy`是一个折叠操作。选择命令式风格是重新实现折叠，而选择函数式风格是重用抽象。

#### 练习 8.18

编写一个`unfold`方法，该方法接受一个起始元素`S`和一个从`S`到`Result<Tuple<A, S>>`的函数`f`，并通过连续应用`f`到`S`值（只要结果是`Success`）来生成一个`List<A>`。换句话说，以下代码应该生成从 0 到 9 的整数列表：

```
List.unfold(0, i -> i < 10
    ? Result.success(new Tuple<>(i, i + 1))
    : Result.empty());
```

#### 解决方案 8.18

一个简单的非栈安全递归版本很容易实现：

```
public static <A, S> List<A> unfold_(S z,
                                     Function<S, Result<Tuple<A, S>>> f) {
    return f.apply(z).map(x ->
                   unfold_(x._2, f).cons(x._1)).getOrElse(list());
}
```

不幸的是，尽管这个解决方案很聪明，但它会在超过 1,000 次递归步骤后耗尽栈空间。为了解决这个问题，你可以创建一个尾递归版本，并使用`TailCall`类在堆上执行递归：

```
public static <A, S> List<A> unfold(S z,
                                    Function<S, Result<Tuple<A, S>>> f) {
  return unfold(list(), z, f).eval().reverse();
}

private static <A, S> TailCall<List<A>> unfold(List<A> acc, S z,
                                      Function<S, Result<Tuple<A, S>>> f) {
    Result<Tuple<A, S>> r = f.apply(z);
    Result<TailCall<List<A>>> result =
               r.map(rt -> sus(() -> unfold(acc.cons(rt._1), rt._2, f)));
    return result.getOrElse(ret(acc));
}
```

注意，然而，这将反转列表。这可能在小型列表中不是大问题，但对于大型列表可能是。在这种情况下，回到命令式风格可能是一个选择。

#### 练习 8.19

编写一个 `range` 方法，它接受两个整数作为参数，并生成一个所有大于或等于第一个整数且小于第二个整数的整数列表。

##### 提示

当然，你应该使用你已经定义的方法。

#### 解答 8.19

如果你重用练习 8.18 中的方法，这会非常简单：

```
public static List<Integer> range(int start, int end) {
  return List.unfold(start, i -> i < end
      ? Result.success(new Tuple<>(i, i + 1))
      : Result.empty());
}
```

#### 练习 8.20

创建一个 `exists` 方法，它接受一个从 `A` 到 `Boolean` 的函数，表示一个条件，如果列表中至少有一个元素满足这个条件，则返回 `true`。不要使用显式递归，但尝试基于你已定义的方法构建。

##### 提示

没有必要评估列表中所有元素的条件。该方法应在找到满足条件的第一个元素时立即返回。

#### 解答 8.20

递归解决方案可以定义为以下：

```
public boolean exists(Function<A, Boolean> p) {
  return p.apply(head()) || tail().exists(p);
}
```

因为 `||` 运算符会惰性评估其第二个参数，所以一旦找到满足谓词 `p` 表达的条件的一个元素，递归过程就会停止。但这是一个非尾递归的基于栈的方法，如果列表很长且在前 1,000 或 2,000 个元素中没有找到满足条件的元素，它将会耗尽栈空间。顺便提一下，如果列表为空，它也会抛出异常，因此你必须在 `List` 类中定义一个抽象方法，并为 `Nil` 子类提供一个特定的实现。

一个更好的解决方案是重用 `foldLeft` 方法，并使用零参数：

```
public boolean exists(Function<A, Boolean> p) {
  return foldLeft(false, true, x -> y -> x || p.apply(y))._1;
}
```

#### 练习 8.21

创建一个 `forAll` 方法，它接受一个从 `A` 到 `Boolean` 的函数，表示一个条件，如果列表中的所有元素都满足这个条件，则返回 `true`。

##### 提示

不要使用显式递归。而且，你并不总是需要评估列表中所有元素的条件。`forAll` 方法将与 `exists` 方法非常相似。

#### 解答 8.21

这个解决方案与 `exists` 方法非常相似，有两个不同之处：恒等值和零值被反转，布尔运算符是 `&&` 而不是 `||`：

```
public boolean forAll(Function<A, Boolean> p) {
  return foldLeft(true, false, x -> y -> x && p.apply(y))._1;
}
```

注意，另一种可能性是重用 `exists` 方法：

```
public boolean forAll(Function<A, Boolean> p) {
  return !exists(x -> !p.apply(x));
}
```

这个方法检查是否存在不满足条件的元素。

### 8.4\. 列表的自动并行处理

应用于列表的大多数计算都会依赖于折叠。折叠涉及将操作应用于列表中的每个元素。对于非常长的列表和持续时间长的操作，折叠可能需要相当长的时间。因为现在大多数计算机都配备了多核处理器（如果不是多个处理器），你可能想找到一种方法让计算机并行处理列表。

为了并行化一个折叠操作，你只需要一件事情（当然，除了多核处理器）：一个额外的操作，允许你重新组合每个并行计算的结果。

#### 8.4.1\. 并非所有计算都可以并行化

以整数列表为例。计算所有整数的平均值并不是可以直接并行化的任务。你可以将列表分成四部分（如果你有一台有四个处理器的计算机），然后计算每个子列表的平均值。但你无法从子列表的平均值中计算出整个列表的平均值。

另一方面，计算列表的平均值意味着计算所有元素的总和，然后除以元素的数量。而计算总和是可以通过计算子列表的总和，然后计算子列表总和的总和来轻松并行化的。

这是一个非常特殊的例子，其中用于折叠的操作（加法）与用于组装子列表结果的操作相同。这并不总是情况。以一个通过添加字符到 `String` 来折叠的字符列表为例。为了组装中间结果，你需要不同的操作：字符串连接。

#### 8.4.2\. 将列表拆分为子列表

首先，你必须将列表拆分为子列表，并且必须自动完成这项操作。一个重要的问题是应该获得多少个子列表。乍一看，你可能认为每个可用的处理器一个子列表是理想的，但这并不完全正确。处理器的数量（或者更精确地说，逻辑核心的数量）并不是最重要的因素。还有一个更关键的问题：所有子列表的计算是否需要相同的时间？可能不是，这取决于计算的类型。如果你决定将列表分成四个子列表，因为你要将四个线程 dedicate 给并行处理，一些线程可能会非常快地完成，而其他线程可能需要进行更长时间的计算。这会破坏并行化的好处，因为它可能会导致大部分计算任务由单个线程处理。

一个更好的解决方案是将列表分成大量的子列表，然后将每个子列表提交给线程池。这样，一旦一个线程完成处理一个子列表，它就会得到一个新的子列表来处理。因此，首要任务是创建一个将列表拆分为子列表的方法。

#### 练习 8.22

编写一个 `divide(int depth)` 方法，该方法将列表分成一定数量的子列表。列表将被分成两部分，每个子列表递归地分成两部分，`depth` 参数表示递归步骤的数量。此方法将在 `List` 父类中实现，以下是其签名：

```
List<List<A>> divide(int depth)
```

##### 提示

你首先定义一个新的 `splitAt` 方法版本，它返回一个列表的列表，而不是一个 `Tuple<List, List>`。让我们称这个方法为 `splitListAt`，并给它以下签名：

```
List<List<A>> splitListAt(int i)
```

#### 解决方案 8.22

`splitListAt` 方法是一个显式递归方法，通过使用 `TailCall` 类来确保栈安全：

```
public List<List<A>> splitListAt(int i) {
  return splitListAt(list(), this.reverse(), i).eval();
}

private TailCall<List<List<A>>> splitListAt(List<A> acc,
                                            List<A> list, int i) {
  return i == 0 || list.isEmpty()
      ? ret(List.list(list.reverse(), acc))
      : sus(() -> splitListAt(acc.cons(list.head()), list.tail(), i - 1));
}
```

当然，此方法将始终返回一个包含两个列表的列表。然后你可以定义 `divide` 方法如下：

```
public List<List<A>> divide(int depth) {
  return this.isEmpty()
      ? list(this)
      : divide(list(this), depth);
}

private List<List<A>> divide(List<List<A>> list, int depth) {
  return list.head().length() < depth || depth < 2 
      ? list
      : divide(list.flatMap(x -> x.splitListAt(x.length() / 2)), depth / 2);
}
```

注意，您不需要使此方法堆栈安全，因为递归步骤的数量将只有 log(length)。换句话说，您永远不会拥有足够的堆内存来持有足够长的列表以导致堆栈溢出。

#### 8.4.3\. 并行处理子列表

要并行处理子列表，您需要一个特殊版本的方法来执行，它将接受一个额外的参数，即配置了您想要并行使用的线程数的`ExecutorService`。

#### 练习 8.23

在`List<A>`中创建一个`parFoldLeft`方法，它将接受与`fold-Left`相同的参数，加上一个`ExecutorService`和一个从`B`到`B`到`B`的函数，并返回一个`Result<List<B>>`。额外的函数将用于组装子列表的结果。以下是方法的签名：

```
public<B> Result<B> parFoldLeft(ExecutorService es, B identity,
             Function<B, Function<A, B>> f, Function<B, Function<B, B>> m)
```

#### 解决方案 8.23

首先，您必须定义您想要使用的子列表数量，并相应地分割列表：

```
final int chunks = 1024;
final List<List<A>> dList = divide(chunks);
```

然后，您将使用一个将任务提交给`Executor-Service`的函数映射子列表。该任务包括折叠每个子列表并返回一个`Future`实例。将`Future`实例的列表映射到调用每个`Future`上的`get`的函数，以生成一个结果列表（每个子列表一个）。请注意，您必须捕获潜在的异常。

最终，结果列表将与第二个函数折叠，并将结果以`Result.Success`的形式返回。在发生异常的情况下，返回`Failure`。

```
try {
  List<B> result = dList.map(x -> es.submit(() -> x.foldLeft(identity,
                                                         f))).map(x -> {
    try {
      return x.get();
    } catch (InterruptedException | ExecutionException e) {
      throw new RuntimeException(e);
    }
  });
  return Result.success(result.foldLeft(identity, m));
} catch (Exception e) {
  return Result.failure(e);
}
```

您将在附带的代码中找到一个此方法的示例基准测试（[`github.com/fpinjava/fpinjava`](https://github.com/fpinjava/fpinjava)）。基准测试包括使用非常慢的算法计算 1 到 30 之间 35,000 个随机数的 10 次斐波那契值。在四核 Macintosh 上，并行版本执行时间为 22 秒，而串行版本需要 83 秒。

#### 练习 8.24

虽然映射可以通过折叠实现（因此可以受益于自动并行化），但它也可以在不使用折叠的情况下并行实现。这可能是可以在列表上实现的 simplest automatic parallelization。创建一个`parMap`方法，它将自动将给定的函数并行应用于列表的所有元素。以下是方法的签名：

```
public <B> Result<List<B>> parMap(ExecutorService es, Function<A, B> g)
```

##### 提示

实际上，在这个练习中几乎没有什么要做。只需将每个函数应用提交给`ExecutorService`，并从每个相应的`Callable`获取结果。

#### 解决方案 8.24

这是解决方案：

```
public <B> Result<List<B>> parMap(ExecutorService es, Function<A, B> g) {
  try {
    return Result.success(this.map(x -> es.submit(() -> g.apply(x)))
                                                             .map(x -> {
      try {
        return x.get();
      } catch (InterruptedException | ExecutionException e) {
        throw new RuntimeException(e);
      }
    }));
  } catch (Exception e) {
    return Result.failure(e);
  }
}
```

伴随此书的代码中可用的基准测试将允许您测量性能的提升。当然，这种提升可能因运行程序的机器而异。

### 8.5\. 总结

+   通过使用缓存，可以提高列表处理的效率。

+   您可以将`Result`实例的`List`转换为`List`的`Result`。

+   您可以通过`zip`操作组装两个列表。您还可以将元组的列表解包以生成列表的`Tuple`。

+   您可以使用显式递归来实现列表元素的索引访问。

+   您可以实现一个特殊的`foldLeft`版本，以在获得“零”结果时跳出折叠。

+   您可以通过一个函数和终止条件来展开创建列表。

+   列表可以自动拆分，这允许并行处理子列表。
