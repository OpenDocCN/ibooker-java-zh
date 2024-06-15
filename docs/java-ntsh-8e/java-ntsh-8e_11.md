# 第九章。处理常见数据格式

大多数编程是处理各种格式的数据。在本章中，我们将介绍 Java 处理两类重要数据——文本和数字的支持。本章的后半部分将专注于处理日期和时间信息。这尤其重要，因为 Java 8 发布了完全新的 API 来处理日期和时间。在讨论 Java 的原始日期和时间 API 之前，我们会对这个接口进行深入讨论。

许多应用程序仍在使用旧的 API，因此开发人员需要了解旧方法，但新的 API 要好得多，我们建议尽快转换。在我们开始处理更复杂的格式之前，让我们先讨论文本数据和字符串。

# 文本

我们已经在许多场合见过 Java 的字符串。它们由 Unicode 字符序列组成，并表示为`String`类的实例。字符串是 Java 程序处理的最常见数据类型之一（您可以通过使用我们将在第十三章中介绍的`jmap`工具自行验证这一点）。

在本节中，我们将更深入地了解`String`类，并理解它在 Java 语言中处于相当独特的位置。在本节的后面，我们将介绍正则表达式，这是一种非常常见的用于搜索文本模式的抽象（无论编程语言如何，都是程序员工具箱中的经典工具）。

## 字符串的特殊语法

`String`类在 Java 语言中以一种略微特殊的方式处理。这是因为尽管它不是原始类型，但字符串是如此常见，以至于 Java 具有许多特殊的语法功能，旨在使字符串处理变得容易。让我们看看 Java 为字符串提供的一些特殊语法功能的示例。

### 字符串字面量

正如我们在第二章中看到的，Java 允许将一系列字符放置在双引号中以创建文字字符串对象。就像这样：

```java
String pet = "Cat";
```

如果没有这种特殊语法，我们将不得不编写像这样可怕的大量代码：

```java
char[] pullingTeeth = {'C', 'a', 't'};
String pet = new String(pullingTeeth);
```

这种方法很快就会变得枯燥乏味，所以毫无意外，像所有现代编程语言一样，Java 提供了简单的字符串字面量语法。字符串字面量是完全合法的对象，因此像下面这样的代码完全合法：

```java
System.out.println("Dog".length());
```

使用基本双引号的字符串不能跨越多行，但是最近版本的 Java 已经包括了使用`"""`语法的多行文本块。生成的字符串对象在编译时创建，并且与`"`引号括起来的字符串没有区别，只是更易于表达：

```java
String lyrics = """
 This is the song that never ends
 This song goes on and one my friend
 ...""";
```

请参阅“字符串字面量”了解 Java 中字符串字面量的完整覆盖。

### toString()

此方法在`Object`上定义，旨在允许将任何对象轻松转换为字符串。这使得可以通过使用`System.out.println()`方法轻松打印出任何对象。实际上，该方法是`PrintStream::println`，因为`System.out`是一个类型为`PrintStream`的静态字段。让我们看看该方法的定义：

```java
    public void println(Object x) {
        String s = String.valueOf(x);
        synchronized (this) {
            print(s);
            newLine();
        }
    }
```

这通过使用静态方法`String::valueOf()`创建一个新的字符串：

```java
    public static String valueOf(Object obj) {
        return (obj == null) ? "null" : obj.toString();
    }
```

###### 注意

静态的`valueOf()`方法用于代替直接调用`toString()`，以避免在`obj`为`null`时抛出`NullPointerException`。

这种构造方式意味着`toString()`对于任何对象始终可用，这对 Java 提供的另一个主要语法特性非常有用：字符串连接。

### 字符串连接

Java 允许我们通过将一个字符串的字符“添加”到另一个字符串的末尾来创建新的字符串。这称为*字符串连接*，并使用`+`运算符。在 Java 8 及以下版本中，它的工作原理是首先创建一个`StringBuilder`对象作为“工作区”，其中包含与原始字符串相同的字符序列。

###### 注意

Java 9 引入了一种新的机制，使用`invokedynamic`指令而不是直接使用`StringBuilder`。这是一种高级功能，超出了本讨论的范围，但不会改变对 Java 开发人员可见的行为。

然后更新构建器对象，并将附加字符串的字符添加到末尾。最后，在`StringBuilder`对象上调用`toString()`（现在包含来自两个字符串的字符）。这给了我们一个包含所有字符的新字符串。无论何时使用`+`运算符连接字符串，`javac`都会自动创建所有这些代码。

连接过程返回一个全新的`String`对象，正如我们在这个例子中所见：

```java
String s1 = "AB";
String s2 = "CD";

String s3 = s1;
System.out.println(s1 == s3); // Same object? Yes.

s3 = s1 + s2;
System.out.println(s1 == s3); // Still same? Nope!
System.out.println(s1);
System.out.println(s3);
```

连接示例直接显示了`+`运算符不会直接修改（或*突变*）`s1`。这是一个更一般的原则示例：Java 的字符串是不可变的。这意味着一旦选择了组成字符串的字符并创建了`String`对象，那么这个`String`就不能被改变。这是 Java 中一个重要的语言原则，让我们稍微深入了解一下。

## 字符串的不可变性

要“改变”一个字符串，就像我们讨论字符串连接时看到的那样，实际上需要创建一个中间的`StringBuilder`对象作为临时的工作区，并在其上调用`toString()`，以将其转换为一个新的`String`实例。让我们看看代码是如何工作的：

```java
String pet = "Cat";
StringBuilder sb = new StringBuilder(pet);
sb.append("amaran");
String boat = sb.toString();
System.out.println(boat);
```

像这样的代码在行为上等效于以下内容，尽管在 Java 9 及以上版本中实际的字节码序列将有所不同：

```java
String pet = "Cat";
String boat = pet + "amaran";
System.out.println(boat);
```

当然，除了在`javac`下使用之外，`StringBuilder`类也可以直接在应用程序代码中使用，我们已经看到了这一点。

###### 警告

除了 `StringBuilder`，Java 还有一个 `StringBuffer` 类。这来自 Java 的最古老版本，不应该用于新开发——应该使用 `StringBuilder`，除非你确实需要在多个线程之间共享新字符串的构造。

字符串的不可变性是一种非常有用的语言特性。例如，假设 `+` 修改了字符串而不是创建新的字符串；那么，每当任何线程连接两个字符串时，所有其他线程也会看到这种变化。对于大多数程序来说，这不太可能是有用的行为，因此不可变性是合理的选择。

### 哈希码和有效不可变性

我们已经在 第五章 中遇到了 `hashCode()` 方法，我们描述了该方法必须满足的合约。让我们看一下 JDK 源代码，看看 `String::hashCode()` 方法是如何定义的：

```java
    public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;

            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }
```

字段 `hash` 存储了字符串的哈希码，而字段 `value` 是一个 `char[]`，存储了构成字符串的实际字符。从代码中可以看出，Java 通过循环遍历字符串的所有字符来计算哈希值。因此，计算机指令的数量与字符串中的字符数成正比。对于非常大的字符串，这可能需要一些时间。Java 不会预先计算哈希值，而是在需要时才计算。

当方法运行时，通过遍历字符数组来计算哈希值。在数组的末尾，我们退出 `for` 循环并将计算得到的哈希值写回到 `hash` 字段中。现在，当再次调用此方法时，值已经被计算过了，所以我们可以直接使用缓存的值，后续调用 `hashCode()` 将立即返回。

###### 注意

字符串的哈希码计算是一个*良性数据竞争*的例子。在具有多个线程的程序中，它们可能会竞争计算哈希码。但是，它们最终会得出完全相同的答案，因此称为*良性*。

`String` 类的所有字段都是 `final` 的，除了 `hash`。因此，Java 的字符串严格来说并不是不可变的。但是，因为 `hash` 字段只是从其他所有不可变字段中确定性计算的值的缓存，只要 `String` 类被正确编码，它就会表现得像是不可变的。具有这种属性的类称为*有效不可变*——它们在实践中非常罕见，工作程序员通常可以忽略真正不可变和有效不可变数据之间的区别。

## 字符串格式化

在 “字符串连接” 中，我们看到 Java 如何支持通过连接较小的字符串来构建更大的字符串。虽然这很有效，但在构造更复杂的输出字符串时，这往往会变得单调和容易出错。Java 提供了许多其他方法和类来进行更丰富的字符串格式化。

`String`类上的静态方法`format`允许我们指定一个模板，然后动态地插入各种值：

```java
// Result is the string "The 1 pet is a cat: true?"
var s = String.format("The %d pet is a %s: %b?%n", 1, "cat", true);

// Same result, but called on string instance instead of statically
s = "The %d pet is a %s: %b?%n".formatted(1, "cat", true);
```

格式字符串中的占位符，其中将引入值，以`%`字符开始。在这个例子中，我们用`%d`替换整数，用`%s`替换字符串，用`%b`替换布尔值，并最终用`%n`换行。

那些在 C 或类似语言中有背景的人会从这个受人尊敬的`printf`函数中认识到这种格式。Java 支持许多相同的格式，尽管不是全部，具有各种各样的选项。Java 的`printf`还提供了更复杂的日期和时间格式化，就像在 C 的`strftime`函数中看到的一样。请参阅 Java 文档中的`java.util.Formatter`，了解可用的所有选项的完整列表。

Java 还通过在出现无效条件时抛出异常来改进使用这些格式字符串的体验，比如占位符与值的数量不匹配，或者无法识别的`%`值。

`String.format()`提供了构建复杂字符串的强大工具，但是，特别是在使输出在各个国家正确时，需要更多的帮助。 `NumberFormat`是 Java 提供的支持更复杂的、与区域设置相关的值格式化的类的一个示例。其他格式化程序也可以在`java.text`下找到：

```java
// Some common locales are available as constants
// A much longer list can be accessed at runtime
var locale = Locale.US;

NumberFormat.getNumberInstance(locale).format(1_000_000_000L)
// 1,000,000,000

NumberFormat.getCurrencyInstance(locale).format(1_000_000_000L)
// $1,000,000,000.00

NumberFormat.getPercentInstance(locale).format(0.1)
// 10%

NumberFormat.getCompactNumberInstance(locale , NumberFormat.Style.LONG)
            .format(1_000_000_000L)
// 1 billion

NumberFormat.getCompactNumberInstance(locale, NumberFormat.Style.SHORT)
            .format(1_000_000_000L)
// 1B
```

## 正则表达式

Java 支持*正则表达式*（通常缩写为*regex*或*regexp*）。这些是用于扫描和匹配文本的搜索模式的表示。正则表达式是我们要搜索的字符序列。它们可以非常简单——例如，`abc`表示我们正在搜索的文本中的任何位置的*a*，后面紧跟着*b*，后面紧跟着*c*。请注意，搜索模式可能在输入文本中匹配零个、一个或多个位置。

最简单的正则表达式只是文本的字面字符序列，比如`abc`。然而，正则表达式的语言可以表达比字面序列更复杂、更微妙的想法。例如，正则表达式可以表示如下所示的匹配模式：

+   数字字符

+   任意字母

+   任意数量的字母，它们必须都在*a*到*j*的范围内，但可以是大写或小写

+   *a*后面跟着任意四个字符，然后是*b*

我们用来编写正则表达式的语法很简单，但是由于我们可以构建复杂的模式，通常可以编写一个没有精确实现我们想要的内容的表达式。在使用正则表达式时，全面测试它们非常重要。这应该包括应该通过的测试用例和应该失败的情况。

为了表达这些更复杂的模式，正则表达式使用*元字符*。这些是指示需要特殊处理的特殊字符。这可以类比于操作系统 shell 中使用 `*` 字符的用法。在这些情况下，理解 `*` 不是字面上解释，而是表示“任何东西”。如果我们想在 Unix 的当前目录中列出所有的 Java 源文件，我们会发出以下命令：

```java
ls *.java
```

正则表达式的元字符是相似的，但它们的数量要多得多，并且比 shell 中可用的集合要灵活得多。它们的含义也不同于它们在 shell 脚本中的含义，因此不要感到困惑。

###### 警告

世界上存在许多不同的正则表达式模式。Java 的正则表达式与 PCRE 兼容，支持一组常见的 Perl 编程语言中广泛使用的元字符。但请注意，网上找到的随机正则表达式可能实际上可能有效，也可能无效，取决于你使用的正则表达式库。

让我们来看几个例子。假设我们想要一个拼写检查程序，它对英式英语和美式英语之间的拼写差异宽松。这意味着 *honor* 和 *honour* 都应该被接受为有效的拼写选择。这在正则表达式中很容易做到。

Java 使用一个称为 `Pattern`（来自包 `java.util.regex`）的类来表示正则表达式。但是，这个类不能直接实例化。相反，通过使用静态工厂方法 `compile()` 来创建新实例。然后，从模式派生一个 `Matcher` 用于特定输入字符串，我们可以用它来探索输入字符串。例如，让我们来看一下莎士比亚戏剧 *凯撒大帝* 中的一部分：

```java
Pattern p = Pattern.compile("honou?r");

String caesarUK = "For Brutus is an honourable man";
Matcher mUK = p.matcher(caesarUK);

String caesarUS = "For Brutus is an honorable man";
Matcher mUS = p.matcher(caesarUS);

System.out.println("Matches UK spelling? " + mUK.find());
System.out.println("Matches US spelling? " + mUS.find());
```

###### 注意

当使用 `Matcher` 时要小心，因为它有一个名为 `matches()` 的方法。但是，这个方法指示模式是否可以覆盖整个输入字符串。如果模式只在字符串中间开始匹配，它将返回 `false`。

最后一个示例介绍了我们的第一个正则表达式元字符 `?`，在模式 `honou?r` 中。这意味着“前面的字符是可选的”——所以 `honour` 和 `honor` 都将匹配。让我们看另一个例子。假设我们想匹配 *minimize* 和 *minimise*（后一种拼写在英国英语中更常见）。我们可以使用方括号来指示可以从集合中选择任何一个字符（但只能有一个备选项）`[]`——就像这样：

```java
Pattern p = Pattern.compile("minimi[sz]e");
```

表 9-1 提供了 Java 正则表达式中可用的扩展元字符列表。

表 9-1\. 正则表达式元字符

| 元字符 | 含义 | 注释 |
| --- | --- | --- |
| `?` | 可选字符——零个或一个实例 |  |
| `*` | 前面字符的零个或多个 |  |
| `+` | 前面字符的一个或多个 |  |
| `{M,N}` | 在前面字符的 `M` 和 `N` 个实例之间 |  |
| `\d` | 一个数字 |  |
| `\D` | 一个非数字字符 |  |
| `\w` | 一个单词字符 | 数字，字母和 `_` |
| `\W` | 一个非单词字符 |  |
| `\s` | 空白字符 |  |
| `\S` | 非空白字符 |  |
| `\n` | 换行符 |  |
| `\t` | 制表符 |  |
| `.` | 任意单个字符 | 在 Java 中不包括换行符 |
| `[ ]` | 方括号中的任何字符 | 称为字符类 |
| `[^ ]` | 不在方括号中的任何字符 | 称为否定字符类 |
| `( )` | 构建模式元素组 | 称为组（或捕获组） |
| `&#124;` | 定义替代可能性 | 实现逻辑`OR` |
| `^` | 字符串开头 |  |
| `$` | 字符串结尾 |  |
| `\\` | 字面转义（`\\`）字符 |  |

还有一些其他内容，但这是基本列表。`java.util.regex.Pattern`的 Java 文档是获取所有详细信息的良好来源。通过这些信息，我们可以构建更复杂的匹配表达式，比如本节前面提到的示例：

```java
String text = "Apollo 13";

// A numeric digit. Note we must use \\ because we need a literal \
// and Java uses a single \ as an escape character, as per the table
Pattern p = Pattern.compile("\\d");
Matcher m = p.matcher(text);
System.out.print(p + " matches " + text + "? " + m.find());
System.out.println(" ; match: " + m.group());

// A single letter
p = Pattern.compile("[a-zA-Z]");
m = p.matcher(text);
System.out.print(p + " matches " + text + "? " + m.find());
System.out.println(" ; match: " + m.group());

// Any number of letters, which must all be in the range 'a' to 'j'
// but can be upper- or lowercase
p = Pattern.compile("([a-jA-J]*)");
m = p.matcher(text);
System.out.print(p + " matches " + text + "? " + m.find());
System.out.println(" ; match: " + m.group());

// 'a' followed by any four characters, followed by 'b'
text = "abacab";
p = Pattern.compile("a....b");
m = p.matcher(text);
System.out.print(p + " matches " + text + "? " + m.find());
System.out.println(" ; match: " + m.group());
```

正则表达式极其有用，可以确定字符串是否与给定模式匹配，还可以从字符串中提取片段。这是通过*组*机制完成的，模式中通过`()`表示：

```java
String text = "Apollo 13";

Pattern p = Pattern.compile("Apollo (\\d*)");
Matcher m = p.matcher(text);
System.out.print(p + " matches " + text + "? " + m.find());
System.out.println("; mission: " + m.group(1));
```

调用`Matcher.group(1)`返回我们模式中`(\\d*)`匹配的文本。允许多个组，还有通过名称而非位置使用组的语法。详细信息请参阅 Java 文档。

处理正则表达式时的一个常见困难是需要同时为 Java 字符串和正则表达式使用转义字符。文本块中少了一些转义字符，比如引号字符，它们可以提供更清晰的表达：

```java
// Detect if there are any double-quoted passages in string
// Note standard string literal requires escaping quotations
Pattern oldQuoted = Pattern.compile(".*\".*\".*");

Pattern newQuoted = Pattern.compile("""
 .*".*".*""");
```

让我们通过介绍 Java 8 中作为`Pattern`的一部分新增的方法`asPredicate()`来结束我们对正则表达式的快速导览。该方法的存在使我们能够轻松地从正则表达式过渡到 Java 集合及其对 lambda 表达式的新支持。

例如，假设我们有一个正则表达式和一组字符串。很自然地会问一个问题：“哪些字符串与该正则表达式匹配？”我们可以使用过滤惯用法，并使用辅助方法将正则表达式转换为`Predicate`，像这样：

```java
// Contains a numeric digit
Pattern p = Pattern.compile("\\d");

List<String> ls = List.of("Cat", "Dog", "Ice-9", "99 Luftballoons");
List<String> containDigits = ls.stream()
        .filter(p.asPredicate())
        .toList();

System.out.println(containDigits);
```

Java 内置的文本处理支持对于大多数商业应用通常需要的文本处理任务已经足够。更高级的任务，例如搜索和处理非常大的数据集，或复杂的解析（包括形式语法），超出了本书的范围，但 Java 拥有大量有用的库和专门技术的绑定，用于文本处理和分析。

# 数字和数学

在这一节中，我们将更详细地讨论 Java 对数值类型的支持。特别是，我们将讨论 Java 使用的整数类型的二进制补码表示。我们将介绍浮点数的表示方式，并涉及它们可能引起的一些问题。我们还将通过一些使用 Java 标准数学操作库函数的示例来说明。

## Java 如何表示整数类型

Java 的整数类型都是有符号的，正如我们在“原始数据类型”中首次提到的那样。这意味着所有整数类型都可以表示正数和负数。由于计算机使用二进制，这意味着表示这些数字的唯一合理方式是分割可能的位模式并使用其中一半来表示负数。

让我们用 Java 的`byte`类型来研究 Java 如何表示整数。它有 8 位，因此可以表示 256 个不同的数字（即 128 个负数和 128 个非负数）。用`0b0000_0000`模式表示零是合乎逻辑的（回忆一下 Java 使用`0b<二进制数字>`的语法来表示二进制数），然后可以轻松地找出正数的位模式：

```java
byte b = 0b0000_0001;
System.out.println(b); // 1

b = 0b0000_0010;
System.out.println(b); // 2

b = 0b0000_0011;
System.out.println(b); // 3

// ...

b = 0b0111_1111;
System.out.println(b); // 127
```

当我们设置字节的第一个位时，符号会改变（因为我们已经用完了为非负数保留的所有位模式）。所以模式`0b1000_0000`应该表示某个负数——但是具体是哪个呢？

###### 注意

由于我们定义的方式，我们可以很简单地识别出一个位模式是否表示负数：如果位模式的最高位是`1`，则表示的数字是负数。

考虑一个由所有位设置为 1 的位模式：`0b1111_1111`。如果我们给这个数字加上`1`，结果会溢出一个`byte`类型的 8 位存储空间，导致`0b1_0000_0000`。如果我们希望将其限制在`byte`数据类型内，则应忽略溢出，因此这变为`0b0000_0000`，也就是零。因此，自然地采用“所有位设置为 1 表示`-1`”的表示方式。这样可以实现自然的算术行为，如下所示：

```java
b = (byte) 0b1111_1111; // -1
System.out.println(b);
b++;
System.out.println(b);

b = (byte) 0b1111_1110; // -2
System.out.println(b);
b++;
System.out.println(b);
```

最后，让我们看一下`0b1000_0000`表示的数字。它是该类型可以表示的最负的数字，所以对于`byte`类型：

```java
b = (byte) 0b1000_0000;
System.out.println(b); // -128
```

这种表示方式被称为*二进制补码*，是有符号整数最常见的表示方式。要有效使用它，你只需记住两点：

+   一个所有位为 1 的模式表示为-1。

+   如果最高位设置为 1，则数字为负数。

Java 的其他整数类型（`short`、`int`和`long`）的行为方式非常相似，但其表示中包含更多的位数。`char`数据类型不同，因为它表示 Unicode 字符，但在某些方面它的行为类似于无符号的 16 位数值类型。在 Java 程序员眼中，它通常不被视为整数类型。

## Java 和浮点数

计算机使用二进制表示数字。我们已经看到 Java 如何使用补码表示整数。但是对于分数或小数呢？Java 和几乎所有现代编程语言一样，使用*浮点算术*来表示它们。让我们首先看看这是如何工作的，首先是十进制，然后是二进制。Java 将两个最重要的数学常数，`e`和`π`（pi），定义为`java.lang.Math`中的常量，如下：

```java
public static final double E = 2.7182818284590452354;
public static final double PI = 3.14159265358979323846;
```

当然，这些常数实际上是*无理数*，不能精确地表示为分数或任何有限小数。^(1) 这意味着每当我们尝试在计算机中表示它们时，总会存在舍入误差。假设我们只想处理π的八位数，我们希望将这些数字表示为一个整数。我们可以使用以下表示方法：

`314159265 • 10^(–8)`

```java
This starts to suggest the basis of how floating-point numbers work. We
use some of the bits to represent the significant digits (`314159265`,
in our example) of the number and some bits to represent the *exponent*
of the base (`-8`, in our example). The collection of significant digits
is called the *significand* and the exponent describes whether we need
to shift the significand up or down to get to the desired number.
Of course, in the examples we’ve met until now, we’ve been working in
base-10\. Computers use binary, so we need to use this as the base in our
floating-point examples. This introduces some additional complications.

###### Note

The number `0.1` cannot be expressed as a finite sequence of binary
digits. This means that virtually all calculations that humans care
about will lose precision when performed in floating point, and rounding
error is essentially inevitable.

Let’s look at an example that shows the rounding problem:

```

double d = 0.3;

System.out.println(d); // 为了避免丑陋的表示而特别处理

double d2 = 0.2;

// 应该是 -0.1，但打印出 -0.09999999999999998

System.out.println(d2 - d);

```java

The official standard that describes floating-point arithmetic is IEEE-754, and
Java’s support for floating point is based on that standard. The standard uses
24 binary digits for standard precision and 53 binary digits for double
precision.
As we mentioned briefly in Chapter 2, Java previously allowed deviation
from this standard, resulting in greater precision when some hardware features
were used to accelerate calculations. As of Java 17, this is no longer allowed,
and all floating-point operations comply with the IEEE-754 standard.

### BigDecimal

Rounding error is a constant source of headaches for programmers who
work with floating-point numbers. In response, Java has a class
`java.math.BigDecimal` that provides arbitrary precision arithmetic, in
a decimal representation. This works around the problem of `0.1` not
having a finite representation in binary, but there are still some edge
conditions when converting to or from Java’s primitive types, as you can
see:

```

double d = 0.3;

System.out.println(d);

BigDecimal bd = new BigDecimal(d);

System.out.println(bd);

bd = new BigDecimal("0.3");

System.out.println(bd);

```java

However, even with all arithmetic performed in base-10, there are still
numbers, such as `1/3`, that do not have a terminating decimal
representation. Let’s see what happens when we try to represent such
numbers using `BigDecimal`:

```

bd = new BigDecimal(BigInteger.ONE);

bd.divide(new BigDecimal(3.0));

System.out.println(bd); // 应该是 1/3

```java

As `BigDecimal` can’t represent `1/3` precisely, the call to `divide()`
blows up with `ArithmeticException`. When you are working with `BigDecimal`, it
is therefore necessary to be acutely aware of exactly which operations
could result in a nonterminating decimal result. To make matters worse,
`ArithmeticException` is an unchecked, runtime exception and so the Java
compiler does not even warn about possible exceptions of this type.

As a final note on floating-point numbers, the paper “What Every
Computer Scientist Should Know About Floating-Point Arithmetic” by David
Goldberg should be considered essential further reading for all
professional programmers. It is easily and freely obtainable on the
internet.

```

## Java 的标准数学函数库

结束对 Java 对数值数据和数学支持的探索，让我们快速浏览一下 Java 附带的标准库函数。这些主要是静态辅助方法，位于`java.lang.Math`类上，包括函数如下：

`abs()`

返回一个数的绝对值。具有多种基本类型的重载形式。

三角函数

计算正弦、余弦、正切等的基本函数。Java 还包括双曲函数版本和反函数（如反正弦）。

`max()`，`min()`

重载函数用于返回两个参数中较大和较小的一个（相同的数值类型）。

`ceil()`，`floor()`

用于舍入到整数。`floor()`返回小于参数的最大整数（参数为 double）。`ceil()`返回大于参数的最小整数。

`pow()`，`exp()`，`log()`

用于计算一个数的幂以及计算指数和自然对数的函数。`log10()`提供以 10 为底的对数，而不是自然底数。

让我们看一些如何使用这些函数的简单示例：

```java
System.out.println(Math.abs(2));
System.out.println(Math.abs(-2));

double cosp3 = Math.cos(0.3);
double sinp3 = Math.sin(0.3);
System.out.println((cosp3 * cosp3 + sinp3 * sinp3)); // Always 1.0

System.out.println(Math.max(0.3, 0.7));
System.out.println(Math.max(0.3, -0.3));
System.out.println(Math.max(-0.3, -0.7));

System.out.println(Math.min(0.3, 0.7));
System.out.println(Math.min(0.3, -0.3));
System.out.println(Math.min(-0.3, -0.7));

System.out.println(Math.floor(1.3));
System.out.println(Math.ceil(1.3));
System.out.println(Math.floor(7.5));
System.out.println(Math.ceil(7.5));

System.out.println(Math.round(1.3)); // Returns long
System.out.println(Math.round(7.5)); // Returns long

System.out.println(Math.pow(2.0, 10.0));
System.out.println(Math.exp(1));
System.out.println(Math.exp(2));
System.out.println(Math.log(2.718281828459045));
System.out.println(Math.log10(100_000));
System.out.println(Math.log10(Integer.MAX_VALUE));

System.out.println(Math.random());
System.out.println("Let's toss a coin: ");
if (Math.random() > 0.5) {
    System.out.println("It's heads");
} else {
    System.out.println("It's tails");
}
```

结束本节时，让我们简要讨论一下 Java 的 `random()` 函数。第一次调用时，它设置一个新的 `java.util.Random` 实例。这是一个 *伪随机数生成器*（PRNG）—一个通过数学公式产生 *看起来* 随机但实际上是由公式产生的确定性代码片段。^(2) 在 Java 的情况下，用于 PRNG 的公式非常简单，例如：

```java
    // From java.util.Random
    public double nextDouble() {
        return (((long)(next(26)) << 27) + next(27)) * DOUBLE_UNIT;
    }
```

如果伪随机数的序列总是从同一位置开始，那么将生成完全相同的数字流。为了解决这个问题，伪随机数生成器的种子由一个应包含尽可能多真实随机性的值来设置。对于这种随机性来源于种子值的来源，Java 使用通常用于高精度计时的 CPU 计数器值。

###### 警告

虽然 Java 内置的伪随机数对大多数一般应用来说足够了，但某些专业应用（尤其是密码学和某些类型的模拟）对随机数有更严格的要求。如果你正在开发这类应用，请寻求已在该领域工作的程序员的专家建议。

现在我们已经看过了文本和数值数据，让我们继续看一下另一种最常见的数据类型：日期和时间信息。

# 日期和时间

几乎所有的业务软件应用都涉及到日期和时间的概念。在建模真实世界的事件或交互时，收集事件发生时间点对未来的报告或领域对象比较至关重要。Java 8 对开发者处理日期和时间的方式进行了彻底的改革。本节介绍了这些概念。在早期版本中，唯一的支持是通过类似 `java.util.Date` 的类，这些类不能很好地建模这些概念。使用旧的 API 的代码应尽快迁移。

## 引入 Java 8 日期和时间 API

Java 8 引入了新的包 `java.time`，其中包含大多数开发者使用的核心类。它还包含四个子包：

`java.time.chrono`

开发者使用非 ISO 标准的日历系统时将与之交互的备选年表。例如，日本的日历系统。

`java.time.format`

包含用于将日期和时间对象转换为 `String`，以及将字符串解析为日期和时间对象的 `DateTimeFormatter`。

`java.time.temporal`

包含核心日期和时间类所需的接口，还包括用于日期高级操作的抽象（如查询和调整器）。

`java.time.zone`

用于底层时区规则的类；大多数开发者不需要这个包。

在表示时间时，最重要的概念之一是某个实体时间线上瞬时点的概念。虽然这个概念在例如特殊相对论中有明确定义，但在计算机中表示它需要我们做出一些假设。在 Java 中，我们将时间的单个点表示为`Instant`，它有以下关键假设：

+   我们无法表示超过`long`类型可以容纳的秒数。

+   我们无法以比纳秒精度更精确地表示时间。

这意味着我们限制自己以一种与当前计算机系统能力相一致的方式来建模时间。然而，还应引入另一个基本概念。

一个`Instant`是关于时空中单个事件的概念。然而，程序员经常需要处理两个事件之间的间隔，因此 Java 还包含了`java.time.Duration`类。该类忽略了可能出现的日历效应（例如夏令时）。通过这种对瞬时事件和事件之间持续时间的基本理解，让我们继续探讨关于瞬时事件的可能思考方式。

### 时间戳的部分

在图 9-1 中，我们展示了时间戳的不同部分在多种可能的方式下的分解。

![JN7 0901](img/jns8_0901.png)

###### 图 9-1\. 时间戳的分解

这里的关键概念是在不同的时间可能适用于多种不同的抽象。例如，有些应用程序中，`LocalDate`对业务处理至关重要，所需的粒度是工作日。另外，有些应用程序要求亚秒甚至毫秒的精度。开发人员应了解他们的领域，并在应用程序中使用合适的表示。

### 示例

日期和时间 API 一开始可能会让人感到困惑，所以让我们从看一个示例开始，并讨论一个日记类，用于跟踪生日。如果你对生日很健忘，那么这样的类（特别是像`getBirthdaysInNextMonth()`这样的方法）可能会非常有帮助：

```java
public class BirthdayDiary {
    private Map<String, LocalDate> birthdays;

    public BirthdayDiary() {
        birthdays = new HashMap<>();
    }

    public LocalDate addBirthday(String name, int day, int month,
                                 int year) {
        LocalDate birthday = LocalDate.of(year, month, day);
        birthdays.put(name, birthday);
        return birthday;
    }

    public LocalDate getBirthdayFor(String name) {
        return birthdays.get(name);
    }

    public int getAgeInYear(String name, int year) {
        Period period = Period.between(
              birthdays.get(name),
              birthdays.get(name).withYear(year));

        return period.getYears();
    }

    public Set<String> getFriendsOfAgeIn(int age, int year) {
        return birthdays.keySet().stream()
                .filter(p -> getAgeInYear(p, year) == age)
                .collect(Collectors.toSet());
    }

    public int getDaysUntilBirthday(String name) {
        Period period = Period.between(
              LocalDate.now(),
              birthdays.get(name));

        return period.getDays();
    }

    public Set<String> getBirthdaysIn(Month month) {
        return birthdays.entrySet().stream()
                .filter(p -> p.getValue().getMonth() == month)
                .map(p -> p.getKey())
                .collect(Collectors.toSet());
    }

    public Set<String> getBirthdaysInCurrentMonth() {
        return getBirthdaysIn(LocalDate.now().getMonth());
    }

    public int getTotalAgeInYears() {
        return birthdays.keySet().stream()
                .mapToInt(p -> getAgeInYear(p,
                      LocalDate.now().getYear()))
                .sum();
    }
}
```

这个课程展示了如何使用低级 API 来构建有用的功能。它还使用了像 Java Streams API 这样的创新技术，并演示了如何将`LocalDate`作为不可变类使用，以及如何将日期视为值进行处理。

## 查询

在广泛的情况下，我们可能会发现自己想要回答有关特定时间对象的问题。一些可能需要回答的示例问题包括：

+   日期是否在三月一日之前？

+   日期是否在闰年中？

+   从今天到我的下一个生日还有多少天？

这是通过使用`TemporalQuery`接口来实现的，其定义如下：

```java
public interface TemporalQuery<R> {
    R queryFrom(TemporalAccessor temporal);
}
```

`queryFrom()`的参数不应为`null`，但如果结果表明未找到值，则可以使用`null`作为返回值。

###### 注意

`Predicate` 接口可以被视为只能代表是或否问题的查询。时间查询更为通用，可以返回“多少？”或“哪个？”的值，而不仅仅是“是”或“否”。

让我们通过考虑一个回答以下问题的查询来看一个查询的示例在操作中的应用：“这个日期属于一年的哪个季度？”Java 不直接支持季度的概念。而是使用这样的代码：

```java
LocalDate today = LocalDate.now();
Month currentMonth = today.getMonth();
Month firstMonthofQuarter = currentMonth.firstMonthOfQuarter();
```

这仍然没有提供季度作为一个单独的抽象，而是仍然需要特殊的情况代码。所以让我们通过定义这个枚举类型稍微扩展 JDK 的支持：

```java
public enum Quarter {
    FIRST, SECOND, THIRD, FOURTH;
}
```

现在，查询可以写为：

```java
public class QuarterOfYearQuery implements TemporalQuery<Quarter> {
    @Override
    public Quarter queryFrom(TemporalAccessor temporal) {
        LocalDate now = LocalDate.from(temporal);

        if(now.isBefore(now.with(Month.APRIL).withDayOfMonth(1))) {
            return Quarter.FIRST;
        } else if(now.isBefore(now.with(Month.JULY)
                               .withDayOfMonth(1))) {
            return Quarter.SECOND;
        } else if(now.isBefore(now.with(Month.NOVEMBER)
                               .withDayOfMonth(1))) {
            return Quarter.THIRD;
        } else {
           return Quarter.FOURTH;
        }
    }
}
```

`TemporalQuery` 对象可以直接或间接使用。让我们分别看一些示例：

```java
QuarterOfYearQuery q = new QuarterOfYearQuery();

// Direct
Quarter quarter = q.queryFrom(LocalDate.now());
System.out.println(quarter);

// Indirect
quarter = LocalDate.now().query(q);
System.out.println(quarter);
```

在大多数情况下，最好使用间接方法，其中查询对象作为参数传递给 `query()`。因为这样在代码中通常更容易阅读。

## 调整器

调整器修改日期和时间对象。例如，假设我们想返回包含特定时间戳的季度的第一天：

```java
public class FirstDayOfQuarter implements TemporalAdjuster {
    @Override
    public Temporal adjustInto(Temporal temporal) {
        final int currentQuarter = YearMonth.from(temporal)
                .get(IsoFields.QUARTER_OF_YEAR);

        final Month firstMonthOfQuarter = switch (currentQuarter) {
            case 1 -> Month.JANUARY;
            case 2 -> Month.APRIL;
            case 3 -> Month.JULY;
            case 4 -> Month.OCTOBER;
            default -> throw new IllegalArgumentException("Impossible");
        };

        return LocalDate.from(temporal)
                .withMonth(firstMonthOfQuarter.getValue())
                .with(TemporalAdjusters.firstDayOfMonth());
    }
}
```

让我们看一个使用调整器的例子：

```java
LocalDate now = LocalDate.now();
Temporal fdoq = now.with(new FirstDayOfQuarter());
System.out.println(fdoq);
```

这里的关键是 `with()` 方法，代码应该被解读为接受一个 `Temporal` 对象并返回另一个已修改的对象。对于处理不可变对象的 API 来说，这是完全正常的。

## 时区

如果你处理关于日期的代码，几乎肯定会遇到来自时区的复杂性。除了向用户清晰地展示信息的简单问题之外，时区还会引起问题，因为它们会变化。无论是来自夏令时的调整还是政府重新分配给定领土的区域，今天的时区定义不能保证下个月相同。

JVM 自带标准 IANA 时区数据的副本，因此通常需要 JDK 升级来获取时区更新。对于那些需要更频繁变更的人，Oracle 发布了一个 [`tzupdater` 工具](https://oreil.ly/RIMLr)，可用于在原地修改 JDK 安装以使用更新的数据。

## 遗留日期和时间

不幸的是，许多应用程序尚未转换为使用随 Java 8 一起提供的优秀日期和时间库。因此，为了完整起见，我们简要提到了基于 `java.util.Date` 的遗留日期和时间支持。

###### 警告

遗留的日期和时间类，特别是 `java.util.Date`，不应在现代 Java 环境中使用。考虑重构或重新编写任何仍使用旧类的代码。

在旧版 Java 中，`java.time` 不可用。相反，程序员依赖于由 `java.util.Date` 提供的传统和基础支持。从历史上看，这是表示时间戳的唯一方法，尽管被称为 `Date`，但实际上这个类包含了日期和时间组件 —— 这导致许多程序员感到困惑。

`Date`提供的遗留支持存在许多问题，例如：

+   `Date`类的设计有误。它实际上并不指代一个日期，而更像是一个时间戳。事实证明，我们需要不同的表示形式来表示日期、日期时间和瞬时时间戳。

+   `Date`是可变的。我们可以获得一个日期的引用，然后改变它所指向的日期。

+   `Date`类实际上不接受 ISO-8601，即通用的 ISO 日期标准，作为有效的日期。

+   `Date`类有大量被弃用的方法。

当前的 JDK 为`Date`使用了两个构造函数——一个是旨在成为“现在构造函数”的`void`构造函数，另一个是接受自纪元以来的毫秒数的构造函数。

如果无法避免使用`java.util.Date`，你仍然可以通过像以下示例中的代码进行转换，以利用更新的 API：

```java
// Defaults to timestamp when called
var oldDate = new java.util.Date();

// Note both forms require specifying timezone -
// part of the failing in the old API
var newDate = LocalDate.ofInstant(
                  oldDate.toInstant(),
                  ZoneId.systemDefault());

var newTime = LocalDateTime.ofInstant(
                  oldDate.toInstant(),
                  ZoneId.systemDefault());
```

# 摘要

在本章中，我们遇到了几种不同类别的数据。文本和数字数据是最明显的例子，但作为工作程序员，我们将遇到许多不同类型的数据。让我们继续看看如何处理整个数据文件以及使用新的 I/O 和网络工作方式。幸运的是，Java 提供了处理许多这些抽象的良好支持。

^(1) 实际上，它们实际上是*超越数*的两个已知例子之一。

^(2) 让计算机产生真正的随机数是非常困难的，在确实需要这样做的罕见情况下，通常需要专门的硬件。
