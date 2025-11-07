## 附录 B. 单子

在阅读这本书之后，你可能会对我没有谈论单子的事实感到惊讶（并且可能感到沮丧）。单子是一个热门话题，你可以在网上找到许多所谓的“单子教程”。单子的主题似乎非常令人生畏，许多程序员一个接一个地阅读这些教程，希望他们最终能理解单子是什么。当然，许多其他程序员确实理解单子，但很少有人能够用简单的话解释单子。

有这么多单子教程的原因可能是因为没有确切的教程，所以人们一直在尝试自己编写。这个附录不是另一个单子教程。我不愿意写一个，有两个原因：

+   如果你已经阅读了这本书，你不需要单子教程。尽管我从未使用过*单子*这个术语，但你已经知道什么是单子。你知道这个概念，并在整本书中大量使用了它。你只需要给它命名。

+   关于单子（monads）有一种古老的谚语：一旦你理解了它们，你就失去了向他人解释它们的能力。

但让我们看看别人对单子的看法。在网上搜索，你可以找到许多定义：

+   “单子是端点函子范畴中的单群。”

+   “单子是某个值的计算上下文。”

+   “单子是一个具有`unit`方法和`flatmap`方法的类。”

你也可能找到一些更奇特的定义：

+   “单子是卷饼。”

+   “单子是象。”

在第一个列表中，定义在某些上下文中是有效的...。第一个定义可能是范畴论上下文中最严谨的定义，范畴论是大多数程序员不太关心的数学的一个分支。（他们应该关心，但这又是另一个故事。）

第三个定义可能是 Java 程序员最容易理解的。方法的名字并不重要。重要的是这些方法必须遵守的规则。

第二个定义可能是理解单子最有用的。单子是计算上下文，函数式编程是用函数编程。安全的函数式编程是用全函数编程。不是全函数的函数被称为部分函数，这意味着它们并不总是有一个值（见第二章）。当它们没有值时，它们就不高兴，开始做可怕的事情。它们就不再是纯函数了。

考虑以下函数：

```
f(x, y) = x + y
```

这是一个纯函数吗？没有人能说。这取决于所使用的编程语言。在某些语言中，它可能会抛出一个算术溢出异常，因此它不会是一个全函数，因为它不会为所有`(x, y)`对定义。它不会是一个纯函数，因为抛出异常是一个副作用。

然而，在 Java 中使用整数，这个函数是纯函数。这意味着无论你给函数提供哪一对整数，它都会返回一个值，并且对于相同的对总是返回相同的值。所以你可以信任这个函数。这并不意味着结果总是正确的。在溢出的情况下，结果可能不是你想要的，但这又是另一个问题。总会有一个结果（这意味着程序不会在野外挂起）并且这个结果总是相同的。

这个函数怎么样？

```
g(x, y) = x / y
```

在 Java 和整数的背景下，这意味着一个函数接受一对整数作为其参数并返回一个整数，这不是一个全函数。对于某些整数对，它可能没有结果。如果第二个参数是 0，就没有结果，函数会抛出异常。这是因为如果将`g`视为从（整数，整数）到整数的函数，它不是一个全函数。

要使`g`成为一个全函数，有两种方法：改变定义域，使其成为（整数，非空整数）的函数，或者改变值域，使其成为（整数，整数）到（整数 | 异常）的函数。

要实现第一种方法，你必须创建一个新的类型：`NonNullInteger`。这是完全可能的。

要实现第二种方法，你还得创建一个新的类型：`IntegerOrException`。

函数式程序员更喜欢第二种方法。

但如果你将函数`g`改为返回`IntegerOrException`，你就不能再将其与`f`组合。更准确地说，`f . g (x)`，或者如果你喜欢这种表示法，`f(g(x))`将无法编译，因为类型不再匹配。

解决方案是在一个计算环境中创建函数，以便可以安全地执行。如果你喜欢隐喻，你可以将这个环境想象成一个安全盒。

所以，你需要做的是

+   一个安全盒

+   一种方法是将参数值放入盒子里

+   一种方法是将修改后的函数放入盒子里，以便它可以应用于参数值

就这样。结果将是一个包含函数结果的盒子。

以 Java 为例，简单来说，你需要稍微修改一下要求，因为 Java 没有提供这样的安全盒。它提供了三种安全盒类型，但没有一种适合这种情况，所以你必须通过说在出错的情况下，结果不会抛出异常，而是简单地返回空值来修改要求。（注意，Java 有这种类型：`Void`。但实例化这种类型有点棘手。）

你可以用返回结果或无结果的`Option`来作为安全盒的类型，这是你在第六章中开发的，或者（更好）是第七章中的`Result`类型。在标准的 Java 8 中，可以是`Optional`类型。

在函数式语言中，将值放入盒子的方法通常命名为`unit`或`return`，但你将其命名为`of`用于`Option`和`Result`，就像 Java 8 设计者对`Optional`所做的那样。这并没有改变什么。

允许你将修改后的函数应用于盒子内值的那个方法被称为`flatMap`。让我们以一个更简单的函数为例，它接受一个`String`并返回第一个字符。一个“正常”的方法可能看起来像这样：

```
public static char firstChar(String a) {
  if (a == null || a.length() == 0) {
    throw new IllegalArgumentException();
  } else {
    return a.charAt(0);
  }
}
```

要使这个函数返回第一个字符或无，你必须将其改为以下内容：

```
public static Optional<Character> firstChar(String a) {
  return a == null || a.length() == 0
      ? Optional.empty()
      : Optional.of(a.charAt(0));
}
```

要使用这些工具，你需要将一个`String`放入上下文中：

```
Optional<String> data = Optional.of("Hello!");
Optional<Character> character = data.flatMap(ThisClass::firstChar);
```

`unit`（单位）和`flatMap`方法就足够将`Optional`转换为单子。

然而，还有一些用例足够频繁，以至于在大多数单子实现中都被添加了。例如，你可能需要使用返回原始值的函数，如下所示：

```
public static int toUpper(char c) {
  return c >= 'a' && c <= 'z'
      ? c - 32
      : c;
}
```

你可以用以下方式做到这一点

```
Optional<Integer> upperChar = character.flatMap(x -> Optional.of(toUpper(x)));
```

但这并不非常高效，因为函数将结果包装在一个`Optional`中，只是为了`flatMap`方法解包它。因此，有一个针对这种情况的映射方法：

```
Optional<Integer> upperChar = character.map(ThisClass::toUpper);
```

注意，如果你使用返回`Optional`的函数与`map`一起使用，例如以下示例，你会得到`Optional<Optional<Character>>`：

```
Optional<String> data = Optional.of("Hello!");
Optional<???> character = data.map(ThisClass::firstChar);
```

这可以通过一个名为`flatten`（或`join`）的方法转换为`Optional<Character>`，但这个方法在`Optional`中缺失。正如你所见，`unit`（`of`）、`flatMap`、`map`和`flatten`之间存在强烈的关联。`flatten`方法可以如下实现：

```
public static <T> Optional<T> flatten(Optional<Optional<T>> oot) {
  return oot.flatMap(Function.identity());
}
```

许多其他用例可以在单子内部抽象，但它们对于将类型转换为单子并不是必要的。最常用的之一是`fold`。这种方法通常被视为特定于向量类型，如`List`或`Stream`，但实际上并非如此。例如，对于`Option`，`fold`可以在`None<T>`中实现如下：

```
public <U> U fold(U z, Function<U, Function<T, U>> f) {
  return z;
}
```

在`Some<T>`中也是如此

```
public <U> U fold(U z, Function<U, Function<T, U>> f) {
  return f.apply(z).apply(value);
}
```

（技术上，这是一个左折叠，但这个例子中的差异无关紧要。）

你不能将此方法添加到 Java 8 类`Optional`中，因为它是`final`的。但你可以编写一个外部实现：

```
public static <T, U> U fold(U z, Function<U, Function<T, U>> f, Optional<T> ot) {
  return ot.isPresent() ? f.apply(z).apply(ot.get()) : z;
}
```

这里，你使用了`Optional.get()`，这非常糟糕。（从上下文外部访问值是被禁止的，这意味着这个方法不应该公开。）没有聪明的解决方案来解决这个问题。你知道如果值不存在，`get`方法永远不会被调用，所以你可以写出以下内容：

```
public static <T, U> U fold(U z, Function<U, Function<T, U>> f, Optional<T> ot) {
  return ot.isPresent() ? f.apply(z).apply(ot.orElse(null)) : z;
}
```

但这很丑。不那么丑的实现可能看起来像这样：

```
public static <T, U> U fold(U z, Function<U, Function<T, U>> f, Optional<T> ot) {
  return ot.isPresent()
    ? f.apply(z).apply(ot.orElseThrow(() ->
         new IllegalStateException("This exception is (never) thrown by dead code!")))
    : z;
}
```

无论如何，折叠`Optional`是没有用的，除非是为了理解`orElse`方法实际上是一个`fold`，可以定义为以下内容：

```
public static <T> T orElse(Optional<T> ot, T defaultValue) {
  return fold(defaultValue, ignore -> t -> t, ot);
}
```

是的，这同样毫无用处，但它在研究其他单子，如`Stream`或`List`（当然不是`java.util.List`）时很有帮助。

许多其他方法本可以添加到`Optional`中，但它们缺失了，而且你不能添加它们，因为`Optional`是最终的。这是开发一个全新的`Option`单子的好理由。但与此同时，`Optional`几乎毫无用处，因为它不能携带数据缺失的原因。这就是为什么还需要另一个单子的原因。在这本书中，我们称之为`Result`，它大致对应于 Scala 的`Try`类。
