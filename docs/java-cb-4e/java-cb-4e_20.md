# 附录 A. Java 的过去与现在

# 引言：Java 总是在不断变化

Java 一直是开发者和作者的移动目标。在我的商业培训项目中，我会遇到一些开发者，他们甚至还不知道一些早期的 Java 发行版新增的功能，更不用说现在了。本附录将审视每个主要的 Java 发行版。请参阅乔恩·拜厄斯的 Sun Microsystems 文章《Java 技术：早期岁月》以了解 Java 的早期历史。你也可以在帕德博恩大学网站找到一份副本。¹

Java 8 之前的发布详情被视为古代历史，并已移至我的网站，[*https://darwinsys.com/java/ancientHistory.html*](https://darwinsys.com/java/ancientHistory.html)。

# Java 8 中的新功能

## Java 8 语言变更

Java 8 语言中最重要的新特性是 lambda 表达式。经过十年的讨论如何实现闭包或 lambda 表达式，它们终于在 Java 8 中到来了。这个话题如此广泛，以至于在本版本中专门有一章来讨论；请参阅 第九章。

现在可以在结构化类型上放置注解。

## Java 8 API 变更

Java 8 引入了来自 JSR-310 的新日期/时间 API。这提供了一组更一致和合理的类和例程来处理时间。第六章 已完全重写以使用新 API，并以一个示例展示了旧 API 和新 API 之间的各种转换。

Java 8 引入了闭包、`Streams` 和并行集合等函数式编程技术，我们将在 第九章 中讨论它们。为了支持 `Streams`，接口中有了新的方法，如 `List`、`Map` 和 `Set`，这些接口自 Java 1.1 以来基本没有变化。幸运的是，Java 8 语言支持在接口中添加了一种 `default` 方法类型，因此你的自定义接口实现不需要更改（只要确保将 IDE 设置为最新的编译器级别）。

作为默认方法的一个示例，`Iterable` 获得了一个名为 `forEach()` 的新默认方法，使你可以编写这样的代码：

```java
myList.forEach(e -> /* do something with e here... */);
```

这在 “Iterable.forEach method (Java 8)” 中进一步讨论。

一种名为 *Nashorn* 的新 JavaScript 实现现在通过 `javax.script` 可用（参见 Recipe 18.3），也可以从命令行运行。

Javadoc（参见 Recipe 15.2）已扩展到 `javax.tools` API。

注解可以重复使用，无需手动编写包装注解，例如 `javax.persistence.NamedQueries`（复数形式），它只是 `javax.persistence.NamedQuery`（单数形式）注解的一个容器。

最后，Java 提供了对 Base 64 编码/解码的支持，形式为 `java.util.Base64`，其中包括用于编码和解码的两个嵌套类。

还有许多其他小的变更，例如那些由 [OpenJDK](http://openjdk.java.net/projects/jdk8/features) 讨论的。

# Java 9 中的新功能

Java 9 以引入 Java 平台模块系统（JPMS）而闻名。

由于 JDK 本身已经模块化（JPMS 的最初意图！），新的 `jlink` 工具允许您仅为模块化应用程序构建仅包含所需部分的最小 JDK。

另一个新工具是 JShell，一个用于 Java 的 REPL（读取-求值-打印-循环）表达式求值器。也被称为交互式 Java，JShell 对于原型设计、尝试新想法等非常有用。JShell 在 Recipe 1.4 中有介绍。

这个版本还标志着每六个月发布一个新的主要版本（Java 10、Java 11 等）的开始。同时，Java 8 和 Java 11 被宣布为 LTS（长期支持）版本。

## Java 9 语言变更

新的 *module-info* 文件引入了几个伪关键字，这些词在 *module-info* 文件中具有保留意义，但仍可用作 Java 类中的用户定义名称。其中包括 module、requires、exports、provides、with 等。这也影响了在模块内部使用时的可见性修饰符的含义。

接口（在 Java 8 中添加了默认方法）现在还允许私有方法，供默认方法使用。

## Java 9 API 变更

流 API 的改进，`Stream` 接口新增了几种新方法。

改进的集合 API，包括 `of()` 工厂方法，可以快速创建包含多个值的 `List` 或 `Set`。

# Java 10 新特性（2018 年 3 月）

Java 10 以 `var` 关键字及首次实现六个月发布周期而闻名。

Java 10 引入了 GraalVM，一个基于 Java 编写的即时编译器（类似于 HotSpot）。

在 Java 10 中，OpenJDK 版本的 *cacerts* 文件已完全填充，使得通过 `https` 连接的工作更加顺利。

本地代码头文件工具 `javah` 被移除，其功能由 `javac` 自身提供了等效或更好的功能。

## Java 10 语言变更

`var` 关键字仅适用于局部变量，允许您不必担心变量的实际类型。当然，编译器必须能够推断出变量的类型。让我们在 `jshell` 中探索一些选项：

```java
jshell> var x = 10;
x ==> 10

jshell> var y = 123.4d;
y ==> 123.4

jshell> var z = java.time.LocalDateTime.now();
z ==> 2019-08-31T20:47:36.440491

jshell> var map = new HashMap<String,Integer>();
map ==> {}

jshell> map.put("Meh", 123);
$4 ==> null

jshell> var huh;
|  Error:
|  cannot infer type for local variable huh
|    (cannot use 'var' on variable without initializer)
|  var huh;
|  ^------^

jshell>
```

有些出人意料的是，`var` 实际上不是语言关键字，因此此词仍可用作用户定义名称：

```java
jshell> var var = 123;
var ==> 123

jshell> var
var ==> 123
```

查看 [*https://developer.oracle.com/java/jdk-10-local-variable-type-inference.html*](https://developer.oracle.com/java/jdk-10-local-variable-type-inference.html) 了解关于 `var` 的解释和更多细节。

## Java 10 API 变更

`List` 和 `Set` 添加了新的 `copyOf()` 方法来创建真正的不可修改副本；以前的 `List.unmodifiableList()` 创建了一个*不可修改视图*，如果底层 `List` 更改，它将会发生变化。

## 参见

删除或弃用了相当多的旧功能；请参阅这个[DZone 上的列表](https://dzone.com/articles/java-10-new-features-and-enhancements)。

Simon Ritter 撰写了一篇题为[“Java 10 不慎落入陷阱”](https://www.azul.com/jdk-10-pitfalls-for-the-unwary)的文章。

# Java 11 的新特性（2018 年 9 月）

Java 11 引入了我所称之为“从源码单文件运行”（JEP 330）；您现在可以输入以下内容：

```java
java HelloWorld.java
```

并且 `java` 命令将编译并运行指定的程序。这使得处理单个文件变得*更加*容易，这也是它的主要应用。如果您有两个或更多文件，则必须在 `CLASSPATH` 上编译第二个到 *n* 个文件；您在命令行中指定的源文件必须是带有 `main()` 方法的文件，并且不能在 `CLASSPATH` 上编译。因此，对于简单的事物很有用，但对于复杂的应用则不适用。

还请参阅[DZone 上的此列表](https://dzone.com/articles/90-new-features-and-apis-in-jdk-11)。

## Java 11 API 变更

有关 Java 11 变更的更完整列表，请参阅[此 DZone 列表](https://dzone.com/articles/90-new-features-and-apis-in-jdk-11)。

# Java 12 的新变更（2019 年 3 月）

Java 12 引入了*预览变更*的概念，向 JDK 中添加的功能尚未成为官方规范的一部分。这基本上是其他人可能称之为测试模式；如果足够多的用户表示对预览模式功能有严重问题，JDK 团队可以在将其声明为 JDK 规范的一部分之前修复或淘汰它。

## Java 12 语言变更

+   `switch` 语句可以返回一个值（预览版）

## Java 12 API 变更

一些更显著的变更：

+   用于 `Stream` 的 Tee Collector（将输入复制到多个输出 `Stream`）。

+   `CompactNumberFormat` 替换了我的 `ScaledNumberFormat`（例如，将数字 2,048 打印为 2K）。

+   `String.indent(n)` 返回在 `String` 前面添加了 *n* 个空格的副本。

+   垃圾收集改进（JEP 189：Shenandoah：低暂停时间 GC）；G1 GC 的暂停时间改进。

还有许多其他次要变更，请参阅[*https://www.azul.com/39-new-features-and-apis-in-jdk-12*](https://www.azul.com/39-new-features-and-apis-in-jdk-12)和[*https://openjdk.java.net/projects/jdk/12*](https://openjdk.java.net/projects/jdk/12)。

# Java 13 的新特性（2019 年 9 月）

至撰写本文时，Java 13 是最新的正式版本。它包括以下功能：

+   改进的垃圾收集（再次）

+   改进的应用程序类数据共享（AppCDS）允许编写一个包含应用程序中所有使用的类的归档文件。

+   [文本块](http://cr.openjdk.java.net/~jlaskey/Strings/TextBlocksGuide_v9.html)来替代和简化多行 `String` 文字（预览版）

+   改进的 `switch` 语句，可以返回一个值

+   重写了`Socket`和`ServerSocket`的实现（不改变 API）。

参见[这篇 JavaWorld 文章](https://www.javaworld.com/article/3341388/jdk-13-the-new-features-coming-to-java-13.html)。

# 展望未来

2020 年将会有一个 Java 14，大约在本书印刷时期。

这些是正在开发中的一些功能：

+   记录类型（在预览中；参见 Recipe 7.18）。

+   封闭类型，允许类设计者通过列出所有允许的子类来控制子类化。目前的语法看起来是这样的：

    ```java
    public abstract sealed class Person permits Customer, SalesRep {
        ...
    }
    class Customer extends Person {
        ...
    }
    ```

+   文本块，即用三重双引号括起来的多行文本字符串：

    ```java
    String long = """
    This is a long
    text String."""
    ```

+   一个新的打包工具，`jpackage`，将在主要支持的操作系统上生成一个完整的自安装应用程序。

Java 14 还有几个其他有趣的 JEPs。完整列表可在[OpenJDK](https://openjdk.java.net/projects/jdk/14)找到。从该页面链接的 JEP 对于那些对每个新功能的原因（以及所需工作量）感兴趣的人来说是有趣的阅读。

2020 年还将有一个 Java 15，但它在本书印刷时刚进入早期访问阶段，因此我们在这个版本中没有覆盖它。“Java 的未来总是在变动中”，如 Yoda 所言。

¹ Sun Microsystems，“Java 技术：早期岁月”的文章可以在[*https://web.archive.org/web/20090311011509/http://java.sun.com/features/1998/05/birthday.html*](https://web.archive.org/web/20090311011509/http://java.sun.com/features/1998/05/birthday.html)和 Paderborn 大学网站[*http://gcc.upb.de/www/WI/WI2/wi2_lit.nsf/7544f3043ee53927c12573e70058bbb6/abf8d70f07c12eb3c1256de900638899/$FILE/Java%20Technology%20-%20An%20early%20history.pdf*](http://gcc.upb.de/www/WI/WI2/wi2_lit.nsf/7544f3043ee53927c12573e70058bbb6/abf8d70f07c12eb3c1256de900638899/$FILE/Java%20Technology%20-%20An%20early%20history.pdf)找到。
