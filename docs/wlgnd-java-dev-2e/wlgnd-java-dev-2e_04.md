# 3 Java 17

本章涵盖

+   Text blocks

+   Switch 表达式

+   记录

+   封闭类型

这些代表了自 Java 11 发布以来，直到包括 Java 17 在内，添加到 Java 语言和平台中的主要新特性。

注意：为了理解自 Java 8 以来 Java 发布方法的变化，回顾第一章或附录 A 的讨论可能是个好主意。

除了主要的、用户可见的语言升级之外，Java 17 还包含了许多内部改进（特别是性能升级）。然而，本章重点介绍我们预计将改变开发者编写 Java 方式的主要特性。

## 3.1 Text Blocks

自从 Java 1.0 的第一个版本以来，开发者们一直在抱怨 Java 的字符串。与其他编程语言，如 Groovy、Scala 或 Kotlin 相比，Java 的字符串有时似乎有点原始。

Java 历史上只提供了一种字符串类型——简单的双引号字符串，其中某些字符（特别是 `"` 和 `\`）必须转义才能安全使用。这些字符在出人意料广泛的情境下，导致了需要产生复杂的转义字符串，即使在非常常见的编程情况下也是如此。

*Text Blocks* 项目作为一个预览特性已经经历了多次迭代，现在在 Java 17 中已成为标准特性（我们已在第一章简短地讨论了预览特性）。它的目标是通过允许跨多行的字符串字面量来扩展 Java 语法中的字符串概念。反过来，这应该可以避免大多数历史上 Java 程序员发现是过度阻碍的转义序列。

注意：与各种其他编程语言不同，Java Text Blocks 目前不支持 *插值*，尽管这个特性正在积极考虑中，以包含在未来的版本中。

除了帮助 Java 程序员摆脱处理字符过度转义的麻烦之外，Text Blocks 的一个具体目标是允许嵌入在 Java 程序中的可读字符串代码，而这些字符串代码不是 Java 代码。毕竟，你有多经常需要在你的 Java 程序中包含 SQL 或 JSON（甚至 XML）？

在 Java 17 之前，这个过程确实可能很痛苦，实际上，许多团队求助于使用带有所有额外复杂性的外部模板库。自从 Text Blocks 出现以来，在许多情况下，这不再是必要的。

让我们通过考虑一个 SQL 查询来看看它们是如何工作的。在本章中，我们将使用一些来自金融交易的例子——特别是外汇货币交易（FX）。也许我们的客户订单存储在一个 SQL 数据库中，我们将使用如下查询来访问：

```
String query = """
        SELECT "ORDER_ID", "QUANTITY", "CURRENCY_PAIR" FROM "ORDERS"
        WHERE "CLIENT_ID" = ?
        ORDER BY "DATE_TIME", "STATUS" LIMIT 100;
        """;
```

你应该注意到两点。首先，Text Block 以 `"""` 序列开始和结束，这在 Java 15 之前是不合法的 Java 语法。其次，Text Block 可以在每行的开头使用空白字符缩进——这些空白字符将被忽略。

如果我们打印出 `query` 变量，那么我们得到的就是我们构造的字符串，如下所示：

```
SELECT "ORDER_ID", "QUANTITY", "CURRENCY_PAIR" FROM "ORDERS"
WHERE "CLIENT_ID" = ?
ORDER BY "DATE_TIME", "STATUS" LIMIT 100;
```

这是因为 Text Block 是一个常量表达式（String 类型），就像字符串字面量一样。区别在于 Text Block 在 `javac` 处理并记录到类文件中的常量之前。

1.  行终止符字符被翻译为 LF (\u000A)，即 Unix 行结束约定。

1.  块周围的额外空白被移除，以便允许 Java 源代码有额外的缩进，正如我们的示例所示。

1.  块中的任何转义序列都将被解释。

这些步骤按照上述顺序执行是有原因的。具体来说，最后解释转义序列意味着块可以包含字面转义序列（如 `\n`），而不会被早期步骤修改或删除。

注意：在运行时，从字面量或 Text Block 获取的字符串常量之间绝对没有任何区别。类文件不会以任何方式记录常量的原始来源。

关于 Text Blocks 的更多详细信息，请参阅 JEP 378 ([`openjdk.java.net/jeps/378`](https://openjdk.java.net/jeps/378))。让我们继续前进，了解新的 *Switch Expressions* 功能。

## 3.2 Switch 表达式

自从它的早期版本以来，Java 就支持 *switch 语句*。Java 从 C 和 C++ 中的现有形式中汲取了许多语法灵感，`switch` 语句也不例外，如下所示：

```
switch(month) {
  case 1:
    System.out.println("January");
    break;
  case 2:
    System.out.println("February");
    break;
  // ... and so on
}
```

特别地，Java 的 switch 语句继承了这样一个属性：如果一个 `case` 没有以 `break` 结尾，执行将在下一个 `case` 之后继续。这个规则允许将需要相同处理的案例分组，如下所示：

```
switch(month) {
  case 12:
  case 1:
  case 2:
    System.out.println("Winter, brrrr");
    break;
  case 3:
  case 4:
  case 5:
    System.out.println("Spring has sprung!");
    break;
  // ... and so on
}
```

尽管这种情况带来了便利，但也带来了暗淡和有缺陷的一面。遗漏单个 `break` 是程序员（无论是新手还是老手）容易犯的错误，并且经常引入错误。在我们的例子中，我们会得到错误的答案，因为省略第一个 `break` 会导致 winter 和 spring 的消息都被打印出来。

当尝试捕获一个值以供以后使用时，switch 语句也会显得笨拙。例如，如果我们想获取那条消息以供其他地方使用，而不是打印它，我们就必须在 `switch` 之外设置一个变量，并在每个分支中正确设置它，并可能确保在 `switch` 之后实际上设置了值；类似于以下这样：

```
String message = null;
switch(month) {
  case 12:
  case 1:
  case 2:
    message = "Winter, brrrr";
    break;
  case 3:
  case 4:
  case 5:
    message = "Spring has sprung!";
    break;
  // ... and so on
}
```

就像遗漏 `break` 一样，我们现在必须确保每个 case 正确设置消息变量，否则我们未来的错误报告会有风险。当然，我们可以做得更好。

在 Java 14（JEP 361）中引入的 *Switch 表达式*提供了解决这些不足的替代方案，同时也旨在打开未来的语言前沿。这个目标包括帮助缩小与更多面向函数式语言（例如 Haskell、Scala 或 Kotlin）的语言差距。Switch 表达式的第一个版本更加简洁，如下所示：

```
String message = switch(month) {
  case 12:
  case 1:
  case 2:
    yield "Winter, brrrr";
  case 3:
  case 4:
  case 5:
    yield "Spring has sprung!";
  // ... and so on
}
```

在这种修改后的形式中，我们不再在每个分支中设置变量。相反，每个情况都使用新的 `yield` 关键字将我们想要的值返回以分配给 `String` 变量，整个表达式 *产生一个值*——从一个情况分支到另一个分支（并且每个情况分支都必须产生一个 `yield`）。

带着这个例子，这个新特性——*Switch 表达式*与现有的*Switch 语句*——的含义更加丰富。在编程语言中，*语句*是指执行时产生副作用的一段代码。而*表达式*则是指执行时产生值的一段代码。在 Java 14 之前，`switch` 只是一个产生副作用的语句，但现在它可以用作表达式来产生值。

Switch 表达式还带来了另一种更加简洁的语法，这可能会被更广泛地采用，如下所示：

```
String message = switch(month) {
  case 1, 2, 12  -> "Winter, brrrr";
  case 3, 4, 5   -> "Spring has sprung!";
  case 6, 7, 8   -> "Summer is here!";
  case 9, 10, 11 -> "Fall has descended";
  default        -> {
    throw new IllegalArgumentException("Oops, that's not a month");
  }
}
```

`->` 表示我们处于 switch 表达式中，因此那些情况不需要显式的 `yield`。我们的 `default` 情况显示了如何在没有单个值的情况下使用 `{}` 包裹的块。如果你正在使用 switch 表达式的值（就像我们通过将其分配给 `message` 一样），多行情况必须 `yield` 或 `throw`。

但新的标签格式不仅更有帮助且更短，它还解决了实际问题。首先，多个情况通过 `case` 后的逗号分隔列表直接支持。这解决了之前需要危险的 switch 穿透问题。新的标签语法中的 switch 表达式永远不会穿透，为每个人关闭了这个障碍。

增加的安全保障还不止这些。破坏 switch 语句的另一种常见方式是遗漏了你应该处理的情况。如果我们从上一个例子中删除 `default` 行，就会得到一个编译错误，如下所示：

```
error: the switch expression does not cover all possible input values
    String message = switch(month) {
                     ^
```

与 switch 语句不同，Switch 表达式必须处理你输入类型的每个可能的情况，否则你的代码甚至无法编译。这是一个很好的保证，可以帮助你覆盖所有的基础。它还很好地与 Java 的枚举结合，正如我们通过将 `switch` 重写为使用类型安全的常量而不是 `int` 一样：

```
String message = switch(month) {
    case JANUARY, FEBRUARY, DECEMBER  -> "Winter, brrrr";
    case MARCH, APRIL, MAY            -> "Spring has sprung!";
    case JUNE, JULY, AUGUST           -> "Summer is here!";
    case SEPTEMBER, OCTOBER, NOVEMBER -> "Fall has descended";
};
```

这种新功能作为一个独立的功能很有用，因为它允许我们简化 `switch` 的一个非常常见的用法，表现得有点像函数，根据输入值产生一个输出值。实际上，Switch 表达式的规则是，必须保证每个可能的输入值都能产生一个输出值。

注意：如果所有可能的枚举常量都出现在一个 Switch 表达式中，匹配是 *完全的*，因此不需要包含一个 `default` 情况——编译器可以使用枚举常量的完备性。

然而，对于例如接受 `int` 类型的 Switch 表达式，我们必须包含一个 `default` 子句，因为列出大约四十亿个可能值是不切实际的。

Switch 表达式也是通往 Java 未来版本中一个主要特性——模式匹配——的垫脚石，我们将在本章后面和书中后面讨论这一点。现在，让我们继续前进，来认识下一个新特性，即 Records。

## 3.3 Records

Records 是一种新的 Java 类形式，旨在执行以下操作：

+   提供一种一等方法来建模仅包含数据的聚合

+   弥合 Java 类型系统的一个可能差距

+   为常见的编程模式提供语言级别的语法

+   减少类样板代码

这些项目符号的顺序很重要，实际上，Records 更关注语言语义，而不是样板代码的减少和语法（尽管第二个方面是许多开发者倾向于关注的）。让我们首先解释一下 Java 记录的基本概念。

Records 的理念是扩展 Java 语言，并创建一种方式来说明一个类是“仅仅是字段，没有其他”。通过对我们类的这种声明，编译器可以帮助我们自动创建所有方法，并让所有字段参与像 `hashCode()` 这样的方法。

注意：这是语义“记录是字段的透明载体”定义语法的这种方式：“访问器方法和其他样板代码自动从记录定义中派生。”

要了解它在日常编程中的表现，请记住，关于 Java 最常见的抱怨之一是你需要编写很多代码才能使一个类变得有用。通常，我们需要编写

+   `toString()`

+   `hashCode()` 和 `equals()`

+   获取器方法

+   公共构造函数

等等。

对于简单的领域类，这些方法通常是无聊的、重复的，并且很容易被机械地生成（IDEs 通常提供这种功能），但直到我们有了 Records，语言并没有提供直接这样做的方法。当我们阅读别人的代码时，这种令人沮丧的差距实际上更糟。例如，它可能看起来作者正在使用 IDE 生成的 `hashCode()` 和 `equals()`，它使用了类的所有字段，但我们如何能确定而不检查实现中的每一行呢？如果重构期间添加了字段而方法没有重新生成会发生什么？

Records 解决了这些问题。如果一个类型被声明为记录，它就是一个强有力的声明，编译器和运行时会相应地处理它。让我们看看它是如何付诸实践的。

要完全解释这个特性，我们需要一个非平凡的示例领域，所以让我们继续使用 FX 货币交易。如果你对这个领域使用的概念不熟悉，请不要担心——我们会随着我们的进展解释你需要知道的内容。在本书的后面部分，我们将继续探讨金融示例的主题，所以这是一个好的开始点。

让我们来看看我们如何使用 Records 和一些其他功能来改进我们的领域建模，并因此得到更干净、更简洁、更简单的代码。考虑当我们进行 FX 交易时想要下订单的情况。基本订单类型可能包括以下内容：

+   我要购买或出售的单位数量（以货币单位的百万计）

+   “方向”——我是买入还是卖出（通常称为*Bid*和*Ask*）

+   我要兑换的货币（*货币对*）

+   我下单的时间

+   我的订单在超时前有效的时间（*TTL*或*存活时间*）

因此，如果我有一千万英镑，想在下一秒内换成美元，并且我想要每英镑 1.25 美元，那么我现在就是“以 1.25 美元的价格买入 GBP/USD 汇率，有效期为 1 秒。”在 Java 中，我们可能会声明一个领域类，如下所示（我们称之为“classic”，以表明我们目前必须使用类来做这件事——更好的方法即将到来）：

```
public final class FXOrderClassic {
    private final int units;
    private final CurrencyPair pair;
    private final Side side;
    private final double price;
    private final LocalDateTime sentAt;
    private final int ttl;

    public FXOrderClassic(int units, CurrencyPair pair, Side side,
                          double price, LocalDateTime sentAt, int ttl) {
        this.units = units;
        this.pair = pair; // CurrencyPair is a simple enum
        this.side = side; // Side is a simple enum
        this.price = price;
        this.sentAt = sentAt;
        this.ttl = ttl;
    }

    public int units() {
        return units;
    }

    public CurrencyPair pair() {
        return pair;
    }

    public Side side() {
        return side;
    }

    public double price() {
        return price;
    }

    public LocalDateTime sentAt() {
        return sentAt;
    }

    public int ttl() {
        return ttl;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        FXOrderClassic that = (FXOrderClassic) o;

        if (units != that.units) return false;
        if (Double.compare(that.price, price) != 0) return false;
        if (ttl != that.ttl) return false;
        if (pair != that.pair) return false;
        if (side != that.side) return false;
        return sentAt != null ? sentAt.equals(that.sentAt) :
                                that.sentAt == null;
    }

    @Override
    public int hashCode() {
        int result;
        long temp;
        result = units;
        result = 31 * result + (pair != null ? pair.hashCode() : 0);
        result = 31 * result + (side != null ? side.hashCode() : 0);
        temp = Double.doubleToLongBits(price);
        result = 31 * result + (int) (temp ^ (temp >>> 32));
        result = 31 * result + (sentAt != null ? sentAt.hashCode() : 0);
        result = 31 * result + ttl;
        return result;
    }

    @Override
    public String toString() {
        return "FXOrderClassic{" +
                "units=" + units +
                ", pair=" + pair +
                ", side=" + side +
                ", price=" + price +
                ", sentAt=" + sentAt +
                ", ttl=" + ttl +
                '}';
    }
}
```

这段代码很多，但它的意思是我的订单可以像这样创建：

```
var order = new FXOrderClassic(1, CurrencyPair.GBPUSD, Side.Bid,
                                1.25, LocalDateTime.now(), 1000);
```

但声明类的代码中，真正必要的部分有多少呢？在 Java 的旧版本中，大多数开发者可能会只是声明字段，然后使用他们的 IDE 来自动生成所有方法。让我们看看 Records 如何改善这种情况。

注意，Java 没有提供除了通过定义一个类之外的方式来谈论数据聚合的方法，所以很明显，任何只包含“只是字段”的类型都将是一个类。

新的概念是一个 *record class*（或通常简称为 record）。这是一个不可变（在通常的“所有字段都是 final”的 Java 意义上），透明的固定值集的载体，被称为 *record components*。每个组件都会产生一个 final 字段来存储提供的值和一个访问器方法来检索该值。字段名和访问器名与组件名匹配。

字段列表为 record 提供了一个 *状态描述*。在一般类中，字段`x`、构造函数参数`x`和访问器`x()`之间可能没有关系，但在 record 中，它们是 *定义上* 指的是同一件事——record *就是* 它的状态。

为了让我们能够创建 record 类的新的实例，还会生成一个构造函数——称为 *规范构造函数*——它有一个参数列表，与声明的状态描述完全匹配。Java 语言现在也提供了简洁的语法来声明 Records，其中程序员只需要声明构成 record 的组件名称和类型，如下所示：

```
public record FXOrder(int units,
                      CurrencyPair pair,
                      Side side,
                      double price,
                      LocalDateTime sentAt,
                      int ttl) {}
```

通过编写这个记录声明，我们不仅节省了一些输入，我们还做出了一个更强的、语义上的声明。`FXOrder` 类型 *就是* 提供的状态，任何实例 *就是* 字段值的透明聚合。

如果我们现在使用 `javap` 检查类文件（我们将在第四章中详细介绍），我们可以看到编译器已经为我们自动生成了一堆样板代码：

```
$ javap FXOrder.class
Compiled from "FXOrder.java"
public final class FXOrder extends java.lang.Record {
  public FXOrder(int, CurrencyPair, Side,
                 double, java.time.LocalDateTime, int);

  public java.lang.String toString();
  public final int hashCode();
  public final boolean equals(java.lang.Object);
  public int units();
  public CurrencyPair pair();
  public Side side();
  public double price();
  public java.time.LocalDateTime sentAt();
  public int ttl();
}
```

这看起来非常像我们为了基于类的实现而必须编写的那些方法集。实际上，构造函数和访问器方法的行为与之前完全相同。然而，像 `toString()` 和 `equals()` 这样的方法使用了一种可能让一些开发者感到惊讶的实现方式，如下所示：

```
public java.lang.String toString();
    Code:
       0: aload_0
       1: invokedynamic #51,  0            // InvokeDynamic #0:toString:
                                           // (LFXOrder;)Ljava/lang/String;
       6: areturn
```

也就是说，`toString()` 方法（以及 `equals()` 和 `hashCode()`）是通过基于 `invokedynamic` 的机制实现的。这是一种我们将在本书后面的章节（第四章和第十六章）中遇到的有力技术。

我们还可以看到，有一个新的类 `java.lang.Record`，它将作为所有记录类的超类型。它是抽象的，并声明 `equals()`、`hashCode()` 和 `toString()` 为抽象方法。`java.lang.Record` 类不能直接扩展，正如我们可以通过尝试编译一些像这样的代码来看到：

```
public final class FXOrderClassic extends Record {
    private final int units;
    private final CurrencyPair pair;
    private final Side side;
    private final double price;
    private final LocalDateTime sentAt;
    private final int ttl;

    // ... rest of class elided
}
```

编译器将拒绝这个尝试：

```
$ javac FXOrderClassic.java
FXOrderClassic.java:3: error: records cannot directly extend Record
public final class FXOrderClassic extends Record {
             ^
1 error
```

获取记录的唯一方法是通过显式声明一个记录并让 `javac` 创建类文件。这也确保了所有记录类都被创建为最终的。

除了方法的自动生成和样板代码的减少之外，当应用于记录时，Java 的几个核心特性也具有特殊的特点。首先，记录必须遵守关于 `equals()` 方法的特殊契约：如果一个记录 `R` 有组件 `c1`、`c2`、... `cn`，并且如果记录实例按照以下方式复制：

```
R copy = new R(r.c1(), r.c2(), ..., r.cn());
```

那么，`r.equals(copy)` 必须是 `true`。请注意，这个不变性是在关于 `equals()` 和 `hashCode()` 的通常熟悉契约的基础上增加的——它并不取代它。

到目前为止，让我们继续讨论记录功能的一些更设计层面的方面。为了做到这一点，回忆一下 Java 中枚举的工作方式是有帮助的。Java 中的枚举是一种特殊的类形式，它实现了一个设计模式（*有限数量的类型安全实例*）但具有最少的语法开销——编译器为我们生成了一堆代码。

同样，Java 中的记录是一种特殊的类形式，它实现了一个模式（*数据载体* 即 *仅包含字段*）并具有最少的语法。我们期望的所有样板代码都将由编译器自动生成。然而，尽管仅包含字段的数据载体类的简单概念在直觉上是有意义的，但这在细节上究竟意味着什么呢？

当首次讨论记录时，考虑了许多可能的不同设计。例如：

+   POJOs 的样板代码减少

+   Java Beans 2.0

+   命名元组

+   产品类型（一种代数数据类型的形式）

布赖恩·戈茨在他的原始设计草图（[`mng.bz/M5j8`](http://mng.bz/M5j8)）中详细讨论了这些可能性。每个设计选项都伴随着从记录设计中心的选择中产生的附加次要问题，例如：

+   Hibernate 能否代理它们？

+   它们是否与经典 Java Bean 完全兼容？

+   它们是否支持名称擦除/“形状可塑性”？

+   它们是否会包含模式匹配和结构化？

如果记录功能基于上述四种方法中的任何一种——每种方法都有其优点和缺点——似乎是合理的。然而，最终的设计决策是记录是*命名元组*。这在一定程度上是由 Java 类型系统中的一个关键设计理念驱动的——*名义类型化*。让我们更深入地了解一下这个关键理念。

### 3.3.1 名义类型化

静态类型化的名义方法是指每个 Java 存储（变量、字段）都有一个确定的类型，并且每种类型都有一个名称，这个名称应该（至少在一定程度上）对人类有意义。

即使在匿名类的情况下，类型仍然有名称——只是编译器分配了名称，这些名称在 Java 语言中不是有效的类型名称（但在 JVM 内部仍然是可以的）。例如，我们可以在 jshell 中看到这一点：

```
jshell> var o = new Object() {
   ...>   public void bar() { System.out.println("bar!"); }
   ...> }
o ==> $0@37f8bb67

jshell> var o2 = new Object() {
   ...>   public void bar() { System.out.println("bar!"); }
   ...> }
o2 ==> $1@31cefde0

jshell> o = o2;
|  Error:
|  incompatible types: $1 cannot be converted to $0
|  o = o2;
|      ^^
```

注意，尽管匿名类是以完全相同的方式声明的，但编译器仍然生成了两个不同的匿名类`$0`和`$1`，并且不允许赋值，因为在 Java 类型系统中，变量具有不同的类型。

注意，在其他（非 Java）语言中，类的整体形状（例如，它有哪些字段和方法）可以用作类型（而不是显式的类型名称）。这被称为*结构化类型化*。

如果记录与 Java 的传统相悖，引入了结构化类型，这将是一个重大的变化。因此，“记录是名义元组”的设计选择意味着我们期望记录在可能使用其他语言中的元组的地方表现最佳。这包括复合映射键的使用，或者模拟方法的多返回值。一个复合映射键的例子可能如下所示：

```
record OrderPartition(CurrencyPair pair, Side side) {}
```

相反，记录不一定能很好地作为现有代码的替代品，这些代码目前使用 Java Bean。存在许多原因，特别是 Java Bean 是可变的，而记录是不可变的，并且它们对访问器的约定不同。记录将访问器方法命名为与字段名相同（这是因为在 Java 中字段和方法名称是分开命名空间的）而 Bean 则使用`get`和`set`前缀。

记录确实允许一些额外的灵活性，超出简单的单行声明形式，因为它们是真正的类。具体来说，开发者可以定义额外的函数、构造函数和静态字段，除了自动生成的默认值之外。然而，这些功能应该谨慎使用。记住，记录的设计意图是允许开发者将相关字段组合成一个单一的不可变数据项。

记录可能创建的一个使用额外方法的例子是一个静态工厂方法，用于模拟某些记录参数的默认值。另一个例子可能是一个`Person`类（具有不可变的出生日期），它可能定义一个`currentAge()`方法。

一个很好的经验法则是：越是有诱惑力添加大量额外的方法等到基本数据载体（或使其实现多个接口），就越有可能应该使用一个完整的类，而不是记录。

### 3.3.2 紧凑记录构造函数

简单性/“完整类”规则的一个可能的重要例外是使用*紧凑构造函数*，语言规范中是这样描述的：

记录类的紧凑构造函数的正式参数是隐式声明的。它们由记录类的派生正式参数列表给出。

紧凑构造函数声明的意图是，在规范构造函数体中只需要给出验证和/或规范化代码；其余的初始化代码由编译器提供。

——Java 语言规范

例如，我们可能想要验证订单，以确保它们不会尝试购买或出售负数量或设置无效的存活时间，如下所示：

```
public record FXOrder(int units, CurrencyPair pair, Side side,
                      double price, LocalDateTime sentAt, int ttl) {
    public FXOrder {
        if (units < 1) {
            throw new IllegalArgumentException(
                    "FXOrder units must be positive");
        }
        if (ttl < 0) {
            throw new IllegalArgumentException(
                    "FXOrder TTL must be positive, or 0 for market orders");
        }
        if (price <= 0.0) {
            throw new IllegalArgumentException(
                    "FXOrder price must be positive");
        }
    }
}
```

Java 记录相对于其他语言中找到的匿名元组的一个优点是，记录的构造函数体允许在创建记录时运行代码。这允许进行验证（如果传递了无效状态，则抛出异常）。这在纯结构元组中是不可能的。

在记录体中使用静态工厂方法也可能是有意义的，例如，为了解决 Java 中默认参数值的缺乏。在我们的交易示例中，我们可能包括一个这样的静态工厂：

```
public static FXOrder of(CurrencyPair pair, Side side, double price) {
    var now = LocalDateTime.now();
    return new FXOrder(1, pair, side, price, now, 1000);
}
```

声明一个快速创建具有默认参数的订单的方法。当然，这也可以声明为一个替代构造函数。开发者应该选择在每个情况下对他们有意义的做法。

另一个使用替代构造函数的用途是创建用于作为复合映射键的记录，如下例所示：

```
record OrderPartition(CurrencyPair pair, Side side) {
    public OrderPartition(FXOrder order) {
        this(order.pair(), order.side());
    }
}
```

然后，类型`OrderPartition`可以很容易地用作映射键。例如，我们可能想要构建一个用于我们的交易匹配引擎的订单簿，如下所示：

```
public final class MatchingEngine {
    private final Map<OrderPartition, RankedOrderBook> orderBooks =
                                                            new TreeMap<>();

    public void addOrder(final FXOrder o) {
        orderBooks.get(new OrderPartition(o)).addAndRank(o);
        checkForCrosses(o.pair());
    }

    public void checkForCrosses(final CurrencyPair pair) {
        // Do any buy orders match with sells now?
    }

    // ...
}
```

现在，当收到新的订单时，`addOrder()` 方法会提取适当的订单分区（由货币对和买卖双方的元组组成）并使用它将新订单添加到相应的价格排名订单簿中。新订单可能与簿上已有的订单相匹配（这被称为“订单交叉”），因此我们需要在 `checkForCrosses()` 方法中检查它是否匹配。

有时我们可能不想使用紧凑的构造函数，而是有一个完整、显式的规范构造函数。这表明我们需要在构造函数中实际工作——对于简单数据载体类，这种用法的用例数量很少。然而，在某些情况下，例如需要制作传入参数的防御性副本时，这是必要的。因此，编译器允许使用显式规范构造函数的可能性——但在使用这种方法之前请仔细思考。

Records 的目的是作为简单的数据载体，是元组的一种版本，它以逻辑和一致的方式适应 Java 的既定类型系统。这将帮助许多应用程序使域类更清晰、更小。它还将帮助团队消除许多底层模式的代码实现。它还应该减少或消除对 Lombok 等库的需求。

许多开发者已经开始报告在使用 Records 后的显著改进。它们还与 Java 17 中出现的另一个新特性——Sealed Types 结合得非常好。

## 3.4 密封类型

Java 的枚举是一个众所周知的语言特性。它允许程序员对有限集合的替代方案进行建模，这些替代方案代表了一个类型的所有可能值——实际上是类型安全的常量。

为了继续我们的 FX 示例，让我们考虑一个 `OrderType` 枚举来表示不同的订单类型：

```
enum OrderType {
    MARKET,
    LIMIT
}
```

这代表了两种可能的 FX 订单类型：一种 *市价* 订单，它将接受当前最佳价格，以及一种 *限价* 订单，它仅在特定价格可用时才会执行。平台通过让 Java 编译器自动生成特殊形式的类类型来实现枚举。

注意：运行时实际上以与其他类略有不同的方式处理库类型 `java.lang.Enum`（所有枚举类都直接扩展），但这里的细节不需要我们关心。

让我们反编译这个枚举，看看编译器生成了什么，如下所示：

```
$ javap -c -p OrderType.class
final class OrderType extends java.lang.Enum<OrderType> {
  public static final OrderType MARKET;

  public static final OrderType LIMIT;

  ...
  // Private constructor
}
```

在类文件中，枚举的所有可能值都定义为 `public` `static final` 变量，构造函数是私有的，因此不能构造额外的实例。

实际上，枚举类似于 Singleton 模式的泛化，除了它不是类的一个实例，而是有限数量的实例。这个模式非常有用，尤其是因为它给了我们一个关于 *完备性* 的概念——给定一个非空的 `OrderType` 对象，我们可以确定它要么是 `MARKET` 实例，要么是 `LIMIT` 实例。

然而，假设我们想在 Java 11 中模拟许多不同的订单。我们必须在两种不令人满意的替代方案之间做出选择。首先，我们可以选择只有一个实现类（或记录），`FXOrder`，其中包含一个状态字段来持有实际类型。这种模式之所以有效，是因为状态字段是枚举类型，并提供了指示哪个类型真正适用于此特定对象的位。这显然是不太理想的，因为它要求应用程序程序员跟踪类型系统真正关心的位。或者，我们可以声明一个抽象基类，`BaseOrder`，并具有具体类型，`MarketOrder` 和 `LimitOrder`，它们是它的子类。

这里的问题是，Java 一直被设计为一个开放的语言，默认可扩展。类一次编译，子类可以在多年（甚至几十年）后编译。截至 Java 11，Java 语言中允许的唯一类继承结构是开放继承（默认）和没有继承（`final`）。

类可以声明一个包私有构造函数，这实际上意味着“只能被包成员扩展”，但在运行时没有任何东西可以阻止用户在平台不包含的包中创建新类，所以这最多只是一种不完整的保护。

如果我们定义一个 `BaseOrder` 类，那么没有任何东西可以阻止第三方创建一个继承自 `BaseOrder` 的 `EvilOrder` 类。更糟糕的是，这种不受欢迎的扩展可以在 `BaseOrder` 类型编译多年（甚至几十年）之后发生，这是非常不希望的。

结论是，到目前为止，开发者一直受到限制，如果他们想要确保未来兼容性，就必须使用一个字段来持有 `BaseOrder` 的实际类型。Java 17 改变了这种状况，通过允许一种新的方式以更细粒度的方式控制继承：*密封类型*。

注意：这种能力以各种形式存在于几种其他编程语言中，近年来已经变得相当流行，尽管实际上它是一个相当古老的想法。

在其 Java 实现（incarnation）中，密封（sealing）所表达的概念是一个类型可以被扩展，但只能由已知的子类型列表扩展，而不能由其他类型扩展。让我们通过一个简单的 `Pet` 类（我们稍后会回到 FX 示例）的例子来看看新的语法：

```
public abstract sealed class Pet {
    private final String name;

    protected Pet(String name) {
        this.name = name;
    }

    public String name() {
        return name;
    }

    public static final class Cat extends Pet {
        public Cat(String name) {
            super(name);
        }

        void meow() {
            System.out.println(name() +" meows");
        }
    }

    public static final class Dog extends Pet {
        public Dog(String name) {
            super(name);
        }

        void bark() {
            System.out.println(name() +" barks");
        }
    }
}
```

类 `Pet` 被声明为 `sealed`，这并不是 Java 之前允许的关键字。未指定的情况下，`sealed` 表示该类只能在本编译单元内部扩展。因此，子类必须嵌套在当前类内部。我们还声明 `Pet` 为 `abstract`，因为我们不希望有任何一般的 `Pet` 实例，只有 `Pet.Cat` 和 `Pet.Dog` 对象。这为我们提供了一种很好的方式来实现我们之前描述的面向对象（OO）建模模式，而不必承担我们讨论的缺点。

密封也可以与接口一起使用，并且在实际应用中，接口形式可能比类形式更广泛地使用。让我们看看当我们想要使用密封来帮助建模不同类型的 FX 订单时会发生什么：

```
public sealed interface FXOrder permits MarketOrder, LimitOrder {
    int units();
    CurrencyPair pair();
    Side side();
    LocalDateTime sentAt();
}

public record MarketOrder(int units,
                          CurrencyPair pair,
                          Side side,
                          LocalDateTime sentAt,
                          boolean allOrNothing) implements FXOrder {

    // constructors and factories elided
}

public record LimitOrder(int units,
                         CurrencyPair pair,
                         Side side,
                         LocalDateTime sentAt,
                         double price,
                         int ttl) implements FXOrder {

    // constructors and factories elided
}
```

这里有几个需要注意的地方。首先，`FXOrder`现在是一个`sealed`接口。其次，我们可以看到第二个新关键字`permits`的使用，它允许开发者列出这个密封接口的允许实现——我们的实现是记录。

注意：当你使用`permits`时，实现类不必位于同一文件中，可以是独立的编译单元。

最后，我们有一个很好的额外奖励——因为`MarketOrder`和`LimitOrder`是合适的类，它们可以拥有特定于其类型的操作。例如，市价单立即采取可用的最佳价格，无需指定价格。另一方面，限价单需要指定订单将接受的价格以及它准备等待多长时间来尝试实现它（生存时间或*TTL*）。如果我们使用字段来指示对象的“真实类型”，这不会很直接，因为所有子类型的方法都必须存在于基类型上，或者迫使我们使用丑陋的下转型。

如果我们现在用这些类型编程，我们知道我们遇到的任何`FXOrder`实例必须是`MarketOrder`或`LimitOrder`之一。更重要的是，编译器也可以使用这个信息。库代码现在可以安全地假设这些是唯一可能的选择，并且这个假设不能被客户端代码违反。

Java 的 OO 模型代表了类型之间关系的两个最基本概念。具体来说，是`"Type X IS-A Y"`和`"Type X HAS-A Y"`。密封类型代表了一种面向对象的概念，以前在 Java 中无法建模：`"Type X IS-EITHER-A Y OR Z"`。或者，它们也可以被视为

+   在`final`和开放类之间的折中方案

+   将枚举模式应用于类型而不是实例

在面向对象编程理论方面，它们代表了一种新的形式关系，因为`o`的可能类型集合是`Y`和`Z`的并集。因此，这被称为各种语言中的*联合类型*或*求和类型*，但不要混淆——它们与 C 的`union`不同。

例如，Scala 程序员可以使用类似的想法，使用 case 类和它们自己的`sealed`关键字版本（我们稍后会遇到 Kotlin 对这个想法的看法）。

除了 JVM 之外，Rust 语言还提供了一种称为 *不交联合* 类型的概念，尽管它使用 `enum` 关键字来引用，这对 Java 程序员来说可能 *极其* 混淆。在函数式编程领域，一些语言（例如 Haskell）提供了一种称为 *代数数据类型* 的功能，其中包含作为特殊情况的和类型。实际上，Sealed Types 和 Records 的组合也为 Java 17 提供了这一功能的版本。

表面上看，这些类型在 Java 中似乎是一个全新的概念，但它们与枚举的深层相似性应该为许多 Java 程序员提供一个良好的起点。事实上，与这些类型相似的东西已经存在于一个地方：多捕获子句中异常参数的类型。

根据 Java 语言规范（JLS 11，第 14.20 节）：

```
The declared type of an exception parameter that denotes its type as a union
with alternatives D1 | D2 | ... | Dn is lub(D1, D2, ..., Dn).
```

然而，在多捕获的情况下，真正的联合类型不能写成局部变量的类型——它是 *不可表示的*。在多捕获的情况下，我们不能创建一个类型为真正联合类型的局部变量。

我们应该关于 Java 的 Sealed Types 提出一点：它们必须有一个基类，所有允许的类型都扩展（或一个所有允许的类型都必须实现的公共接口）。无法表达一个“ISA-String-OR-Integer”的类型，因为 `String` 和 `Integer` 类型除了 `Object` 之外没有共同的继承关系。

注意：一些其他语言允许构建通用联合类型，但在 Java 中这是不可能的。

让我们继续讨论 Java 17 中引入的另一个新语言特性——`instanceof` 关键字的新形式。

## 3.5 新的 instanceof 形式

尽管自 Java 1.0 以来就是语言的一部分，但 `instanceof` 操作符有时会从一些 Java 开发者那里得到一些负面评价。在其最简单的形式中，它提供了一个简单的测试：`x` `instanceof` `Y` 如果值 `x` 可以赋给类型 `Y` 的变量，则返回 `true`，否则返回 `false`（前提是 `null` `instanceof` `Y` 对于每个 `Y` 都是 `false`）。

这种定义被批评为破坏面向对象设计，因为它暗示了对象类型和参数类型选择的不精确性。然而，在实践中，在某些场景中，开发者必须面对一个在编译时类型不完全已知的对象。例如，考虑一个通过反射获得的对象，对其了解甚少。

在这种情况下，正确的方法是使用 `instanceof` 来检查类型是否符合预期，然后进行向下转型。`instanceof` 测试提供了一个保护条件，确保转型不会在运行时引发 `ClassCastException`。生成的代码如下所示：

```
Object o = // ...
if (o instanceof String) {
    String s = (String)o;
    System.out.println(s.length());
} else {
    System.out.println("Not a String");
}
```

从开发者的角度来看，Java 17 中提供的新的 `instanceof` 功能非常简单——它仅仅提供了一种避免转型的方法，如下所示：

```
if (o instanceof String s) {
    System.out.println(s.length());                           ❶
} else {
    System.out.println("Not a String");                       ❷
}

// ... More code                                              ❸
```

❶ s 在此分支中是有效的。

❷ 在“else”分支上，s 不在作用域内。

❸ 一旦 if 语句结束，s 就不再在作用域内。

然而，尽管这可能看起来并不重要，但我们从为这个功能的 JEP 命名方式中得到了一个重要的线索。JEP 394 的标题是“为 instanceof 的模式匹配”，它引入了一个新概念——*模式*。

注意 非常重要的一点是要理解，这与在文本处理和正则表达式中所使用的*模式匹配*是不同的用法。

在这个上下文中，一个模式是以下两个事物的组合：

1.  一个将被应用于值的谓词（又称测试）

1.  一组局部变量，称为*模式变量*，将从值中提取

关键点是，只有当谓词成功应用于值时，才会提取模式变量。

在 Java 17 中，`instanceof`运算符已被扩展为接受类型或*类型模式*，其中类型模式由指定类型的谓词和一个模式变量组成。

注意 我们将在下一节中更详细地介绍类型模式。

目前看来，升级后的`instanceof`似乎并不非常显著，但这却是 Java 语言中首次出现模式，正如我们将看到的，更多的用法即将到来！这只是第一步。

完成了对 Java 17 新语言特性的巡礼后，是时候展望未来，并回到预览功能的主题上。

## 3.6 模式匹配和预览功能

在第一章，我们介绍了预览功能的概念，但我们无法给出一个很好的例子，因为 Java 11 没有任何预览功能！现在我们正在谈论 Java 17，我们可以继续讨论。

事实上，我们在本章中遇到的所有新语言特性，包括 Switch 表达式、记录和密封类型，都经历了相同的生命周期。它们最初作为预览功能出现，并在成为最终功能之前经历了一轮或多轮公开预览。例如，密封类在 Java 15 中进行了预览，然后在 16 中也进行了预览，最终在 Java 17 LTS 中作为最终功能发布。

在本节中，我们将遇到一个扩展模式匹配从`instanceof`到`switch`的预览功能。Java 17 包括这个功能的版本，但仅作为第一个预览版本（有关预览功能的更多详细信息，请参阅第一章）。语法可能在最终发布前发生变化（尽管这个功能被撤回的可能性很小，但对于模式匹配来说，这种情况最不可能发生）。

让我们看看模式匹配如何在简单情况下被用来改进一些必须处理未知类型对象的代码。我们可以使用新的`instanceof`形式来编写一些安全的代码，如下所示：

```
Object o = // ...

if (o instanceof String s) {
    System.out.println("String of length:"+ s.length());
} else if (o instanceof Integer i) {
    System.out.println("Integer:"+ i);
} else {
    System.out.println("Not a String or Integer");
}
```

这很快就会变得繁琐且冗长。相反，我们可以在`switch`表达式和简单的`instanceof`布尔表达式中引入类型模式。在当前（Java 17）预览功能的语法中，我们可以将之前的代码重写为简单形式：

```
var msg = switch (o) {
    case String s      -> "String of length:"+ s.length();
    case Integer i     -> "Integer:"+ i;
    case null, default -> "Not a String or Integer";             ❶
};
System.out.println(msg);
```

❶ 现在允许将`Null`作为情况标签，以防止`NullPointerException`的可能性。

对于那些想要尝试类似代码的开发者，我们应该解释如何使用预览功能进行构建和运行。如果我们尝试编译使用预览功能的类似上一个示例的代码，我们会得到一个错误，如下所示：

```
$ javac ch3/Java17Examples.java
ch3/Java17Examples.java:68: error: patterns in switch statements are a
  preview feature and are disabled by default.

            case String s -> "String of length:"+ s.length();
                 ^
  (use --enable-preview to enable patterns in switch statements)
1 error
```

编译器友好地提示我们可能需要启用预览功能，所以我们再次尝试启用标志：

```
$ javac --enable-preview -source 17 ch3/Java17Examples.java
Note: ch3/Java17Examples.java uses preview features of Java SE 17.
Note: Recompile with -Xlint:preview for details.
```

运行时的情况也类似：

```
$ java ch3.Java17Examples
Error: LinkageError occurred while loading main class ch3.Java17Examples
    java.lang.UnsupportedClassVersionError: Preview features are not enabled
  for ch16/Java17Examples (class file version 61.65535). Try running with
  '--enable-preview'
```

最后，如果我们包含预览标志，那么代码最终将运行：

```
$ java --enable-preview ch13.Java17Examples
```

恒常地启用预览功能确实很麻烦，但它是为了保护开发者，防止任何使用未完成功能的代码逃入生产环境并造成问题。同样，重要的是要注意，当我们尝试在没有运行时标志的情况下运行包含预览功能的类时出现的关于类文件版本的提示。如果我们明确编译了预览功能，我们不会得到标准的类文件，而且大多数团队不应该在生产环境中运行该代码。

Java 17 中模式匹配的预览版本还具有与 Sealed Types 紧密集成的功能。具体来说，模式可以利用 Sealed Types 提供的可能看到类型的唯一性。例如，在处理 FX 订单响应时，我们可能有以下基本类型：

```
public sealed interface FXOrderResponse
        permits FXAccepted, FXFill, FXReject, FXCancelled {
    LocalDateTime timestamp();
    long orderId();
}
```

我们可以将此与`switch`表达式和类型模式结合起来，给出以下代码：

```
FXOrderResponse resp = // ... response from market
var msg = switch (resp) {
    case FXAccepted a  -> a.orderId() + " Accepted";
    case FXFill f      -> f.orderId() + " Filled "+ f.units();
    case FXReject r    -> r.orderId() + " Rejected: "+ r.reason();
    case FXCancelled c -> c.orderId() + " Cancelled";
    case null          -> "Order is null";
};
System.out.println(msg);
```

注意到 a)我们明确包含了一个`case null`来确保此代码是 null 安全的（并且不会抛出`NullPointerException`），以及 b)我们不需要默认情况。第二个原因是因为编译器可以检查`FXOrderResponse`的所有允许的子类型，并且可以得出结论，模式匹配是*完全的*，它涵盖了可能发生的所有可能性，因此默认情况在所有情况下都是无效代码。在匹配不是完全的情况下，并且某些情况没有被覆盖时，需要默认情况。

第一次预览还包括*受保护的模式*，这允许一个模式被布尔守卫条件装饰，因此只有当模式谓词和守卫都为真时，整体模式才匹配。例如，假设我们只想看到大型填充订单的详细信息。我们可以将上一个示例中的填充情况更改为以下代码：

```
case FXFill f && f.units() < 100 -> f.orderId() + " Small Fill";
case FXFill f                    -> f.orderId() + " Fill "+ f.units();
```

注意，更具体的案例（小于 100 个单位的少量订单）首先进行测试，并且只有当它失败时，匹配尝试下一个案例，即填充的无保护匹配。模式变量也已经适用于任何保护条件。我们将在第十八章中回到模式匹配，当我们讨论 Java 的未来并讨论一些未能及时加入 Java 17 的特性时。

## 摘要

+   Java 17 引入了许多新特性，开发者可以立即在自己的代码中利用这些特性：

    +   文本块，用于多行字符串。

    +   开关表达式，提供更现代的开关体验。

    +   记录作为透明数据载体。

    +   封装类型——一个重要的新面向对象建模概念。

    +   模式匹配——尽管在 Java 17 中尚未完全实现，但它清楚地显示了语言在即将到来的版本中的发展方向。
