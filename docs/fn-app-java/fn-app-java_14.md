# 第十二章\. 递归

递归是一种解决问题的方法，可以将其分解为其较小版本。许多开发人员将*递归*视为另一种 — 通常是复杂的 — 基于迭代的问题解决方法。但对于特定类型的问题，掌握不同的技术方式是很有益的。

本章介绍了递归的一般思想，如何实现递归方法以及它们在 Java 代码中与其他形式的迭代相比的位置。

# 什么是递归？

在“递归”中，你已经看到了计算阶乘的示例 — 所有小于或等于输入参数的正整数的乘积。许多书籍、指南和教程使用阶乘来演示递归，因为这是一个很好的部分解决问题的例子，并且也是本章的第一个示例。

计算阶乘的每一步都会将输入参数与下一个阶乘操作的结果相乘。当计算达到`fac(1)` — 定义为“1" — 时，链条终止并将值提供给前一步骤。完整的步骤可以在方程 12-1 中看到。

##### 方程 12-1\. 阶乘计算的正式表示

<math alttext="StartLayout 1st Row 1st Column Blank 2nd Column f a c left-parenthesis n right-parenthesis 2nd Row 1st Column right-arrow 2nd Column n asterisk f a c left-parenthesis n minus 1 right-parenthesis 3rd Row 1st Column right-arrow 2nd Column n asterisk left-parenthesis n minus 1 right-parenthesis asterisk f a c left-parenthesis n minus 2 right-parenthesis 4th Row 1st Column right-arrow 2nd Column 4 asterisk left-parenthesis n minus 1 right-parenthesis asterisk left-parenthesis n minus 2 right-parenthesis asterisk ellipsis asterisk f a c left-parenthesis 1 right-parenthesis 5th Row 1st Column right-arrow 2nd Column 4 asterisk left-parenthesis n minus 1 right-parenthesis asterisk left-parenthesis n minus 2 right-parenthesis asterisk ellipsis asterisk 1 EndLayout" display="block"><mtable displaystyle="true"><mtr><mtd columnalign="left"><mrow><mi>f</mi> <mi>a</mi> <mi>c</mi> <mo>(</mo> <mi>n</mi> <mo>)</mo></mrow></mtd></mtr> <mtr><mtd columnalign="right"><mo>→</mo></mtd> <mtd columnalign="left"><mrow><mi>n</mi> <mo>*</mo> <mi>f</mi> <mi>a</mi> <mi>c</mi> <mo>(</mo> <mi>n</mi> <mo>-</mo> <mn>1</mn> <mo>)</mo></mrow></mtd></mtr> <mtr><mtd columnalign="right"><mo>→</mo></mtd> <mtd columnalign="left"><mrow><mi>n</mi> <mo>*</mo> <mo>(</mo> <mi>n</mi> <mo>-</mo> <mn>1</mn> <mo>)</mo> <mo>*</mo> <mi>f</mi> <mi>a</mi> <mi>c</mi> <mo>(</mo> <mi>n</mi> <mo>-</mo> <mn>2</mn> <mo>)</mo></mrow></mtd></mtr> <mtr><mtd columnalign="right"><mo>→</mo></mtd> <mtd columnalign="left"><mrow><mn>4</mn> <mo>*</mo> <mo>(</mo> <mi>n</mi> <mo>-</mo> <mn>1</mn> <mo>)</mo> <mo>*</mo> <mo>(</mo> <mi>n</mi> <mo>-</mo> <mn>2</mn> <mo>)</mo> <mo>*</mo> <mo>⋯</mo> <mo>*</mo> <mi>f</mi> <mi>a</mi> <mi>c</mi> <mo>(</mo> <mn>1</mn> <mo>)</mo></mrow></mtd></mtr> <mtr><mtd columnalign="right"><mo>→</mo></mtd> <mtd columnalign="left"><mrow><mn>4</mn> <mo>*</mo> <mo>(</mo> <mi>n</mi> <mo>-</mo> <mn>1</mn> <mo>)</mo> <mo>*</mo> <mo>(</mo> <mi>n</mi> <mo>-</mo> <mn>2</mn> <mo>)</mo> <mo>*</mo> <mo>⋯</mo> <mo>*</mo> <mn>1</mn></mrow></mtd></mtr></mtable></math>

这种计算步骤的泛化可视化了递归的基本概念：通过使用调用自身的方法并传入修改后的参数来解决问题，直到达到基本条件。

递归由两种不同的操作类型组成：

基本条件

基本条件是一个预定义的情况 — 问题的*解决方案* — 它将返回一个实际值并展开递归调用链。它将其值提供给前一步骤，后者现在可以计算结果并将其返回给其前任，依此类推。

递归调用

直到调用链达到其*基本条件*，每一步都会通过使用修改后的输入参数调用自身来创建另一个步骤。

图 12-1 显示了递归调用链的一般流程。

![用更小的问题解决问题](img/afaj_1201.png)

###### 图 12-1\. 用更小的问题解决问题

问题会变小，直到找到最小部分的解决方案。然后，此解决方案将成为下一个更大问题的输入，依此类推，直到所有部分的总和构建出原始问题的解决方案。

## 头递归与尾递归

递归调用分为两类，*头*递归和*尾*递归，取决于方法体中递归调用的位置：

头递归

在递归方法调用后执行/评估其他语句/表达式，这使得它不是最后一个语句。

尾递归

递归调用是方法的最后一条语句，没有进一步的计算将其结果链接到当前调用。

让我们来看看使用这两种类型计算阶乘以更好地说明它们的差异。示例 12-1 展示了如何使用头递归。

##### 示例 12-1\. 使用头递归计算阶乘

```java
long factorialHead(long n) { ![1](img/1.png)

  if (n == 1L) { ![2](img/2.png)
    return 1L;
  }

  var nextN = n - 1L;

  return n * factorialHead(nextN); ![3](img/3.png)
}

var result = factorialHead(4L);
// => 24
```

![1](img/#co_recursion_CO1-1)

方法签名只包含当前递归步骤的输入参数。中间状态不在递归调用之间移动。

![2](img/#co_recursion_CO1-2)

基本条件必须出现在递归调用之前。

![3](img/#co_recursion_CO1-3)

返回值是依赖于递归调用结果的表达式，这使得它不是方法中唯一的最后语句。

现在是时候看尾递归了，正如示例 12-2 所示。

##### 示例 12-2\. 使用尾递归计算阶乘

```java
long factorialTail(long n, long accumulator) { ![1](img/1.png)

  if (n == 1L) { ![2](img/2.png)
    return accumulator;
  }

  var nextN = n - 1L;
  var nextAccumulator = n * accumulator;

  return factorialTail(nextN, nextAccumulator); ![3](img/3.png)
}

var result = factorialTail(4L, 1L); ![4](img/4.png)
// => 24
```

![1](img/#co_recursion_CO2-1)

方法签名包含一个累加器。

![2](img/#co_recursion_CO2-2)

与头递归相比，基本条件没有改变。

![3](img/#co_recursion_CO2-3)

不返回依赖于下一个递归调用的表达式，两个`factorialTail`参数都会在此之前求值。该方法仅返回递归调用本身。

![4](img/#co_recursion_CO2-4)

累加器需要一个初始值。它反映了基本条件。

头递归和尾递归之间的主要区别在于调用堆栈的构建方式。

在*头递归*中，递归调用在返回值之前执行。因此，最终结果在运行时从每个递归调用返回之后才可用。

使用*尾递归*时，先解决分解的问题，然后再将结果传递给下一个递归调用。实际上，任何给定递归步骤的返回值都与下一个递归调用的结果相同。如果运行时支持，这样可以优化调用堆栈，如下一节所示。

## 递归和调用堆栈

如果你再次查看图 12-1，你可以将每个方框看作是一个单独的方法调用，因此，在调用堆栈上会有一个新的栈帧。这是必需的，因为每个方框必须与先前的计算隔离开来，这样它们的参数不会相互影响。总的递归调用次数仅受到达到基本条件的时间限制。问题在于，可用的堆栈大小是有限的。过多的调用将填满可用的堆栈空间，并最终引发`StackOverflowError`。

###### 注意

栈帧包含单个方法调用的状态。每次代码调用一个方法时，JVM 都会创建并推送一个新的栈帧到线程的堆栈上。在方法返回后，其栈帧被弹出并丢弃。

实际的最大堆栈深度取决于可用堆栈大小^(1)，以及存储在各个帧中的内容。

为了防止栈溢出，许多现代编译器使用*尾递归优化/消除*来删除递归调用链中不再需要的帧。如果在递归调用之后没有额外的计算发生，那么栈帧就不再需要并且可以被移除。这将递归调用的栈帧空间复杂度从`O(N)`降低到`O(1)`，产生更快、更节省内存的机器代码，避免栈溢出。

不幸的是，截至 2023 年初，Java 编译器和运行时尚缺乏这种特定能力。

尽管如此，递归仍然是特定问题子集的有价值工具，即使在没有调用栈优化的情况下也是如此。

# 更复杂的例子

阶乘计算虽然可以很好地解释递归，但并不是典型的“真实世界”问题。因此，现在是时候看一个更现实的例子了：遍历类似树结构的数据，就像在图 12-2 中看到的那样。

![类似树结构的数据遍历](img/afaj_1202.png)

###### 图 12-2\. 类似树结构的数据遍历

数据结构有一个根节点，每个节点都有一个可选的左右子节点。它们的编号是为了标识，并不是遍历顺序。

节点由通用记录`Node<T>`表示，如示例 12-3 所示。

##### 示例 12-3\. 树节点结构

```java
public record Node<T>(T value, Node<T> left, Node<T> right) {

  public static <T> Node<T> of(T value, Node<T> left, Node<T> right) {
    return new Node<>(value, left, right);
  }

  public static <T> Node<T> of(T value) {
    return new Node<>(value, null, null);
  }

  public static <T> Node<T> left(T value, Node<T> left) {
    return new Node<>(value, left, null);
  }

  public static <T> Node<T> right(T value, Node<T> right) {
    return new Node<>(value, null, right);
  }
}

var root = Node.of("1",
                   Node.of("2",
                           Node.of("4",
                                   Node.of("7"),
                                   Node.of("8")),
                           Node.of("5")),
                   Node.right("3",
                              Node.left("6",
                                        Node.of("9"))));
```

目标是“按顺序”遍历树。这意味着先遍历每个节点的左子节点，直到找不到其他左节点为止。然后，它将继续遍历其右子节点的左节点，然后再向上。

首先，我们将使用迭代方法实现树遍历，然后与递归方法进行比较。

## 迭代树遍历

借助`while`循环，遍历树的方式如您所料。这需要临时变量和协调遍历的样板代码，就像在示例 12-4 中看到的那样。

##### 示例 12-4\. 迭代树遍历

```java
void traverseIterative(Node<String> root) {
  var tmpNodes = new Stack<Node<String>>(); ![1](img/1.png)
  var current = root;

  while(!tmpNodes.isEmpty() || current != null) { ![2](img/2.png)

    if (current != null) { ![3](img/3.png)
      tmpNodes.push(current);
      current = current.left();
      continue;
    }

    current = tmpNodes.pop(); ![4](img/4.png)

    System.out.print(current.value()); ![5](img/5.png)

    current = current.right(); ![6](img/6.png)
  }
}
```

![1](img/#co_recursion_CO3-1)

需要辅助变量来保存迭代的当前状态。

![2](img/#co_recursion_CO3-2)

迭代直到没有节点存在，或者`nodeStack`不再为空。

![3](img/#co_recursion_CO3-3)

`java.util.Stack`保存直到底部节点被访问。

![4](img/#co_recursion_CO3-4)

在此时，循环无法继续更深，因为它遇到了`current == null`，所以它将`current`设置为在`tmpNodes`中保存的最后一个节点。

![5](img/#co_recursion_CO3-5)

输出节点值。

![6](img/#co_recursion_CO3-6)

重复以上步骤，右子节点也是如此。

输出结果如预期：*748251396*。

尽管它按预期工作，但代码并不十分简洁，需要可变的辅助变量才能正常运行。

让我们看看递归方法是否比迭代更好。

## 递归树遍历

要创建一个递归解决方案以遍历树，必须首先明确定义包括基本条件在内的不同步骤。

遍历树需要两次递归调用，一个动作和一个基本条件：

+   遍历左节点

+   遍历右节点

+   打印节点的值

+   如果找不到更多节点，则停止

这些不同步骤在 Java 中的实现顺序如示例 12-5 所示。

##### 示例 12-5\. 递归树遍历

```java
void traverseRecursion(Node<String> node) {
  if (node == null) { ![1](img/1.png)
    return;
  }

  traverseRecursion(node.left()); ![2](img/2.png)

  System.out.print(node.value()); ![3](img/3.png)

  traverseRecursion(node.right()); ![4](img/4.png)
}
```

![1](img/#co_recursion_CO4-1)

如果没有剩余节点，则停止遍历的基本条件。

![2](img/#co_recursion_CO4-2)

首先，递归地遍历左子节点。只要左节点存在，这将再次调用`traverse`。

![3](img/#co_recursion_CO4-3)

其次，因为不再存在左子节点，需要打印当前值。

![4](img/#co_recursion_CO4-4)

第三步，像之前一样使用相同逻辑遍历可能的右子节点。

输出与之前相同：*748251396*。

代码不再需要外部迭代器或辅助变量来保存状态，实际处理逻辑被减少到最小。遍历不再处于*做什么*的命令式思维中。相反，它以更声明式的方式反映了*如何实现*目标的函数式方法。

通过将遍历过程移到类型本身并接受一个`Consumer<Node<T>>`来使树的遍历更加功能化，如示例 12-6 所示。

##### 示例 12-6\. 用遍历方法扩展`Node<T>`

```java
record Node<T>(T value, Node<T> left, Node<T> right) {

  // ...

  private static <T> void traverse(Node<T> node, ![1](img/1.png)
                                   Consumer<T> fn) { ![2](img/2.png)
    if (node == null) {
      return;
    }

    traverse(node.left(), fn);

    fn.accept(node.value());

    traverse(node.right(), fn);
  }

  public void traverse(Consumer<T> fn) { ![3](img/3.png)
    Node.traverse(this, fn);
  }
}

root.traverse(System.out::print);
```

![1](img/#co_recursion_CO5-1)

先前的`traverse`方法可以很容易地重构为原始类型上的`private static`方法。

![2](img/#co_recursion_CO5-2)

新的`traverse`方法接受`Consumer<Node<T>>`以支持任何类型的操作。

![3](img/#co_recursion_CO5-3)

一个用于遍历的`public`方法简化了调用，省略了作为其第一个参数的`this`。

遍历类型变得更加容易。类型本身现在负责最佳遍历方式，并为使用者提供了灵活的解决方案。

与迭代方法相比，这种方式更为简洁、功能性强，更易于理解。但是，使用循环也有其优势。最大的优势是性能差异，通过交换所需的堆栈空间来获得可用的堆空间。在每次递归遍历操作时，不再创建新的堆栈帧，节点在`tmpNodes`上累积于堆中。这使得代码对于可能导致堆栈溢出的较大图形更加健壮。

正如你所见，没有一个简单的答案来确定哪种方法最佳。这完全取决于数据结构的类型以及需要处理的数据量。即使如此，你个人的偏好和对特定方法的熟悉程度可能比寻找“最佳”解决方案更为重要，用于编写简单明了、无 bug 的处理代码。

# 类似递归的 Streams

Java 的运行时可能不支持尾递归优化，然而，你仍然可以通过 lambda 表达式和 Streams 实现类似递归的体验，这些方式不会遭受堆栈溢出问题的困扰。

多亏了 Streams 的惰性特性，你可以构建一个流水线，直到递归问题得以解决。但与其递归调用 lambda 表达式不同，它返回一个新的表达式。这样，无论执行多少递归步骤，堆栈深度都将保持恒定。

与递归或者使用循环相比，这种方法相当复杂。虽然不常用，但它展示了如何结合 Java 的各种新功能组件来解决递归问题。如果你想进一步了解，请查看书籍的[代码库](https://github.com/benweidig/a-functional-approach-to-java)。

# 递归的最终思考

递归通常被忽视，因为很容易出错。例如，错误的基础条件可能是不可能实现的，这必然会导致堆栈溢出。总体来说，递归的流程更难以跟踪和理解，如果你不习惯的话。因为 Java 没有*尾调用优化*，你必须考虑不可避免的开销，这导致执行时间比迭代结构慢，而且可能会遇到`StackOverflowError`的问题。

在选择递归和其它替代方案时，务必考虑额外的开销和堆栈溢出问题。如果你在拥有充足内存和足够大栈大小的 JVM 中运行，即使是更大的递归调用链也不会成问题。但如果你的问题规模未知或不固定，长远来看采用替代方法可能更明智，以预防`StackOverflowError`。

一些场景更适合采用递归方法，即使在 Java 中缺乏尾递归优化的情况下。递归将会感觉更自然地解决特定的问题，特别是对于像链表或树这样的自引用数据结构。遍历类似树结构的数据也可以通过迭代方式实现，但很可能会导致更复杂、更难以理解的代码。

但请记住，仅从技术角度选择最佳解决方案可能会削弱代码的可读性和合理性，从而影响长期可维护性。

表 12-1 提供了递归与迭代之间的区别概述，以便您更有效地选择使用它们。

表 12-1\. 递归与迭代对比

|  | 递归 | 迭代 |
| --- | --- | --- |
| 方法 | 自调用函数 | 循环结构 |
| 状态 | 存储在堆栈上 | 存储在控制变量中（例如循环索引） |
| 进展 | 向基本条件 | 向控制值条件 |
| 终止条件 | 达到基本条件 | 达到控制变量条件 |
| 冗余性 | 较低冗余性，需要的最小样板和协调代码 | 较高冗余性，需要显式协调控制变量和状态。 |
| 如果未终止 | `StackOverflowError` | 无尽循环 |
| 开销 | 重复方法调用的开销较高。 | 恒定堆栈深度减少了开销。 |
| 性能 | 由于开销和缺少尾递归优化而性能较低。 | 由于恒定的调用堆栈深度而性能更好。 |
| 内存使用 | 每次调用都需要堆栈空间。 | 除了控制变量外没有额外的内存使用。 |
| 执行速度 | 较慢 | 较快 |

选择递归还是迭代取决于你要解决的问题以及代码运行的环境。对于解决更抽象的问题，递归通常是首选工具，而迭代更适合于更低级别的代码。迭代可能提供更好的运行时性能，但递归可以提高程序员的生产力。

别忘了你始终可以从熟悉的迭代方法开始，稍后转换为使用递归。

# 要点

+   递归是对“传统”迭代的功能替代。

+   递归最适合部分可解决的问题。

+   Java 缺乏尾递归优化，这可能导致 `StackOverflowExceptions`。

+   不要为了函数式而强制使用递归。你始终可以从迭代方法开始，稍后转换为递归方法。

^(1) 大多数 JVM 实现的默认堆栈大小为一兆字节。你可以使用 `-Xss` 标志设置更大的堆栈大小。请参阅[Oracle Java 工具文档](https://docs.oracle.com/en/java/javase/11/tools/java.xhtml#GUID-3B1CE181-CD30-4178-9602-230B800D4FAE__GUID-72BC3B70-49FF-4588-979F-7F8A32FEE6DA)获取更多信息。
