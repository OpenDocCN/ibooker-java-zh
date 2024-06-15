# 第三章。字符串和其他东西

# 3.0 介绍

字符串是几乎任何编程任务中不可避免的一部分。我们用它们来向用户打印消息；用来引用磁盘上文件或其他外部媒体；以及用于人们的姓名、地址和所属关系。字符串的用途是多种多样的，几乎不计其数（实际上，如果你需要数字，我们会在第五章中介绍它们）。

如果你之前有 C 这样的编程语言的经验，你需要记住在 Java 中`String`是一个定义的类型（类）——也就是说，字符串是一个对象，因此有方法。它不是一个字符数组（尽管它包含一个），不应该被认为是一个数组。像`fileName.endsWith(".gif")`和`extension.equals(".gif")`（以及相应的 "`.gif".equals(extension)`）这样的操作是很常见的。^(1)

Java 老手应该注意，Java 11 和 12 添加了几个新的`String`方法，包括`indent(int n)`，`stripLeading()`和`stripTrailing()`，`Stream<T>` `lines()`，`isBlank()`和`transform()`。其中大多数提供了明显的功能；最后一个允许将一个函数接口的实例（参见食谱 9.0）应用于字符串并返回该操作的结果。

尽管我们还没有讨论`java.io`包的细节（我们将在第十章中讨论），你需要能够读取一些程序所需的文本文件。即使你不熟悉`java.io`，你可能已经从读取文本文件的例子中看到，`BufferedReader`允许你读取数据块，并且这个类有一个非常方便的`readLine()`方法。

反过来说，`System.out.println()`通常用于将字符串或其他值打印到终端或标准输出。在这里通常使用字符串连接，像这样：

```java
System.out.println("The answer is " + result);
```

一个关于字符串连接的警告是，如果你在追加一堆东西时，一个数字和一个字符在前面连接，由于 Java 的优先规则，它们会在连接之前被添加。因此，不要像我在这个假设的例子中那样做：

```java
int result = ...;
System.out.println(result + '=' + " the answer.");
```

假设`result`是一个整数，那么`result + '=' (`result`加上等号，这是一个数值类型的+字符`)`就是一个有效的*数值*表达式，其结果将是一个`int`类型的单一值。如果变量`result`的值是 42，并且考虑到 Unicode（或 ASCII）代码表中字符`=`的值是 61，那么这两行片段将会打印出：

```java
103 the answer.
```

错误的值和没有等号！更安全的方法包括使用括号、在等号周围加上双引号、使用`StringBuilder`（见 Recipe 3.2）或`String.format()`（见 Recipe 10.4）。当然，在这个简单的例子中，你可以将*=*移到字符串字面量的一部分，但选择这个例子是为了说明在`char`值上进行算术运算与字符串连接混淆的问题。我不会在这里向你展示如何对字符串数组进行排序；对对象集合进行排序的更一般概念将在 Recipe 7.11 中讨论。

Java 14 支持文本块，也称为多行文本字符串。这些文本块用一组三个双引号界定，开头的引号后面*必须*有一个换行符（这不会成为字符串的一部分；后续的换行符会）：

```java
String long = """
This is a long
text String."""
```

# 3.1 使用子字符串或标记化分解字符串

## 问题

你想要将字符串拆分成部分，可以通过索引位置或使用固定的令牌字符（例如，通过空格进行拆分以获取单词）。

## 解决方案

对于子字符串，使用`String`对象的`substring()`方法。对于标记化，构造一个围绕你的字符串的`StringTokenizer`，并调用它的方法`hasMoreTokens()`和`nextToken()`。

或者，使用正则表达式（见第四章）。

## 讨论

我们首先看子字符串，然后讨论标记化。

### 子字符串

`substring()`方法构造一个新的`String`对象，由原始字符串的某个位置包含的一系列字符组成，即你调用`substring()`的那个字符串。`substring`方法是重载的：两种形式都需要一个起始索引（始终是*从零开始*）。一种形式的参数返回从`startIndex`到末尾的内容。另一种形式采用一个结束索引（不是长度，如某些语言中那样），以便可以通过`String`方法`indexOf()`或`las⁠t​IndexOf()`生成索引：

```java
public class SubStringDemo {
    public static void main(String[] av) {
        String a = "Java is great.";
        System.out.println(a);
        String b = a.substring(5);    // b is the String "is great."
        System.out.println(b);
        String c = a.substring(5,7);// c is the String "is"
        System.out.println(c);
        String d = a.substring(5,a.length());// d is "is great."
        System.out.println(d);
    }
}
```

运行时，这将打印以下内容：

```java
C:> java strings.SubStringDemo
Java is great.
is great.
is
is great.
C:>
```

###### 警告

注意，结束索引是最后一个字符的索引加一！Java 在相当大程度上采用了这种半开区间（或包含起始，不包含结束）的策略；采用这种方法有很好的实际理由，一些其他语言也这样做。

### 标记化

最简单的方法是使用正则表达式。我们将在第四章讨论正则表达式，但现在，包含一个空格的字符串是一个有效的正则表达式，可以匹配空格字符，所以你可以最简单地这样将字符串拆分成单词：

```java
for (String word : some_input_string.split(" ")) {
    System.out.println(word);
}
```

如果需要匹配多个空格，或空格和制表符，请使用字符串`"\s+"`。

如果你想要拆分一个文件，可以尝试使用字符串`","`或使用几个第三方库来处理 CSV 文件。

另一种方法是使用`StringTokenizer`。`StringTokenizer`方法实现了`Iterator`接口和设计模式（见 Recipe 7.6）：

*main/src/main/java/strings/StrTokDemo.java*

```java
StringTokenizer st = new StringTokenizer("Hello World of Java");

while (st.hasMoreTokens( ))
    System.out.println("Token: " + st.nextToken( ));
```

`StringTokenizer`还实现了`Enumeration`接口（参见 Recipe 7.6），但如果使用其中的方法，您需要将结果转换为`String`。

一个`StringTokenizer`通常会在欧洲语言的单词边界处将`String`拆分为标记。有时您希望在其他字符处进行拆分。没问题。当您构造`StringTokenizer`时，除了传入要标记化的字符串外，还要传入第二个字符串，列出分隔字符，如下所示：

*main/src/main/java/strings/StrTokDemo2.java*

```java
StringTokenizer st = new StringTokenizer("Hello, World|of|Java", ", |");

while (st.hasMoreElements( ))
    System.out.println("Token: " + st.nextElement( ));
```

它按每行一个的方式输出四个单词，没有标点符号。

等等，还有更多！如果您正在读取这样的行

```java
FirstName|LastName|Company|PhoneNumber
```

而你亲爱的老阿姨贝贡尼亚在过去 38 年没有工作吗？她的`Company`字段很可能为空。^(3) 如果您仔细查看前面的代码示例，您会看到它有两个连续的分隔符（逗号和空格）；但是如果您运行它，没有“额外”的标记——也就是说，`StringTokenizer`通常会丢弃相邻的连续分隔符。对于像电话列表这样需要保留空字段的情况，有好消息和坏消息。好消息是您可以做到：在构造`StringTokenizer`时添加第二个参数`true`，表示希望将分隔符视为标记。坏消息是现在您会看到这些分隔符作为标记，因此您必须自己进行算术计算。想看看吗？运行这个程序：

*main/src/main/java/strings/StrTokDemo3.java*

```java
StringTokenizer st =
    new StringTokenizer("Hello, World|of|Java", ", |", true);

while (st.hasMoreElements( ))
    System.out.println("Token: " + st.nextElement( ));
```

您将获得此输出：

```java
C:\>java strings.StrTokDemo3
Token: Hello
Token: ,
Token:
Token: World
Token: |
Token: of
Token: |
Token: Java
C:\>
```

理想情况下，`StringTokenizer`不应该这样行事，但在大多数情况下，它足够使用了。Example 3-1 处理并忽略连续的标记，将结果作为`String`数组返回。

##### Example 3-1\. main/src/main/java/strings/StrTokDemo4.java (StringTokenizer)

```java
public class StrTokDemo4 {
    public final static int MAXFIELDS = 5;
    public final static String DELIM = "|";

    /** Processes one String, returns it as an array of Strings */
    public static String[] process(String line) {
        String[] results = new String[MAXFIELDS];

        // Unless you ask StringTokenizer to give you the tokens,
        // it silently discards multiple null tokens.
        StringTokenizer st = new StringTokenizer(line, DELIM, true);

        int i = 0;
        // Stuff each token into the current slot in the array.
        while (st.hasMoreTokens()) {
            String s = st.nextToken();
            if (s.equals(DELIM)) {
                if (i++>=MAXFIELDS)
                    // This is messy: See StrTokDemo4b which uses
                    // a List to allow any number of fields.
                    throw new IllegalArgumentException("Input line " +
                        line + " has too many fields");
                continue;
            }
            results[i] = s;
        }
        return results;
    }

    public static void printResults(String input, String[] outputs) {
        System.out.println("Input: " + input);
        for (String s : outputs)
            System.out.println("Output " + s + " was: " + s);
    }

    public static void main(String[] a) {
        printResults("A|B|C|D", process("A|B|C|D"));
        printResults("A||C|D", process("A||C|D"));
        printResults("A|||D|E", process("A|||D|E"));
    }
}
```

当您运行此代码时，您会发现`A`始终在字段 1 中，`B`（如果存在）在字段 2 中，依此类推。换句话说，空字段被正确处理了：

```java
Input: A|B|C|D
Output 0 was: A
Output 1 was: B
Output 2 was: C
Output 3 was: D
Output 4 was: null
Input: A||C|D
Output 0 was: A
Output 1 was: null
Output 2 was: C
Output 3 was: D
Output 4 was: null
Input: A|||D|E
Output 0 was: A
Output 1 was: null
Output 2 was: null
Output 3 was: D
Output 4 was: E
```

## 参见

许多`StringTokenizer`的出现可以用正则表达式替换（参见 Chapter 4），具有更大的灵活性。例如，要从一个`String`中提取所有数字，您可以使用以下代码：

```java
Matcher tokenizer = Pattern.compile("\\d+").matcher(inputString);
while (tokenizer.find( )) {
        String courseString = tokenizer.group(0);
        int courseNumber = Integer.parseInt(courseString);
        ...
```

这允许用户输入比您可以轻松处理的`StringTokenizer`更灵活。假设数字表示某个教育机构的课程号，输入“471,472,570”或“课程 471 和 472, 570”或只是“471 472 570”应该都给出相同的结果。

# 3.2 使用 StringBuilder 组合字符串

## 问题

你需要把一些`String`片段（再）组合在一起。

## 解决方案

使用字符串连接：`+`运算符。编译器会为您隐式构造一个`StringBuilder`，并使用其`append()`方法（除非所有字符串部分在编译时都已知）。

更好的方法是自己构建和使用`StringBuilder`。

## 讨论

`StringBuilder`类的对象基本上表示一个字符集合。它类似于一个`String`对象。^(4) 然而，正如前面提到的，`String`是不可变的；而`StringBuilder`是可变的，专门用于构建`String`。通常您会构造一个`StringBuilder`，调用需要的方法以便按照您想要的方式获取字符序列，然后调用`toString()`方法生成表示相同字符序列的`String`，以便在大多数处理`String`的 Java API 中使用。

`StringBuffer`是历史悠久的——它从时间的开始就存在。它的一些方法是同步的（参见 Recipe 16.5），这在单线程环境中会产生不必要的开销。在 Java 5 中，这个类被分成了`StringBuffer`（同步的）和`StringBuilder`（非同步的）；因此，对于单线程使用来说，它更快速且更可取。另一个新类`AbstractStringBuilder`是这两者的父类。在接下来的讨论中，我会使用“`StringBuilder`类”来指代这三者，因为它们大部分具有相同的方法。

书中的示例代码提供了一个`StringBuilderDemo`和一个`StringBufferDemo`。除了`StringBuilder`不是线程安全这一事实外，这些 API 类完全相同，可以互换使用，因此我的两个演示程序几乎相同，只是每个程序使用了适当的构建器类。

`StringBuilder`类提供了各种方法来插入、替换和修改给定的`StringBuilder`。方便的是，`append()`方法返回对`StringBuilder`本身的引用，因此像`.append(…).append(…)`这样的堆叠语句是相当常见的。这种编码风格被称为*流式 API*，因为它读起来流畅，像母语为人类语言的人的散文。例如，在`toString()`方法中甚至可能看到这种编码风格。Example 3-2 展示了三种字符串连接的方式。

##### 示例 3-2\. main/src/main/java/strings/StringBuilderDemo.java

```java
public class StringBuilderDemo {

    public static void main(String[] argv) {

        String s1 = "Hello" + ", " + "World";
        System.out.println(s1);

        // Build a StringBuilder, and append some things to it.
        StringBuilder sb2 = new StringBuilder();
        sb2.append("Hello");
        sb2.append(',');
        sb2.append(' ');
        sb2.append("World");

        // Get the StringBuilder's value as a String, and print it.
        String s2 = sb2.toString();
        System.out.println(s2);

        // Now do the above all over again, but in a more
        // concise (and typical "real-world" Java) fashion.

        System.out.println(
          new StringBuilder()
            .append("Hello")
            .append(',')
            .append(' ')
            .append("World"));
    }
}
```

实际上，所有修改`StringBuilder`内容超过一个字符的方法（即`append()`、`delete()`、`deleteCharAt()`、`insert()`、`replace()`和`reverse()`）都会返回对构建器对象的引用，以支持这种流畅的 API 编程风格。

再举一个使用`StringBuilder`的例子，考虑将项目列表转换为逗号分隔的列表的需求，同时避免在列表的最后一个元素后面得到额外的逗号。这可以通过`StringBuilder`来实现，尽管在 Java 8+中有一个静态的`String`方法来完成相同的工作。这些代码显示在 Example 3-3 中。

##### Example 3-3\. main/src/main/java/strings/StringBuilderCommaList.java

```java
        System.out.println(
            "Split using String.split; joined using 1.8 String join");
        System.out.println(String.join(", ", SAMPLE_STRING.split(" ")));

        System.out.println(
            "Split using String.split; joined using StringBuilder");
        StringBuilder sb1 = new StringBuilder();
        for (String word : SAMPLE_STRING.split(" ")) {
            if (sb1.length() > 0) {
                sb1.append(", ");
            }
            sb1.append(word);
        }
        System.out.println(sb1);

        System.out.println(
            "Split using StringTokenizer; joined using StringBuilder");
        StringTokenizer st = new StringTokenizer(SAMPLE_STRING);
        StringBuilder sb2 = new StringBuilder();
        while (st.hasMoreElements()) {
            sb2.append(st.nextToken());
            if (st.hasMoreElements()) {
                sb2.append(", ");
            }
        }
        System.out.println(sb2);
```

第一种方法显然是最紧凑的；静态的`String.join()`可以轻松完成这个任务。接下来的方法使用了`StringBuilder.length()`方法，因此只有在你从一个空的`StringBuilder`开始时才能正确工作。第二种方法依赖于在每个元素上多次调用枚举类型中的信息方法`hasMoreElements()`（或者在`Iterator`中调用`hasNext()`，如 Recipe 7.6 所讨论的那样）。一种替代方法，特别是当你不是从一个空的构建器开始时，可以使用一个`boolean`标志变量来跟踪你是否位于列表的开头。

# 3.3 逐个字符处理字符串

## 问题

您希望逐个字符处理字符串的内容。

## 解决方案

使用`for`循环和`String`的`charAt()`或`codePointAt()`方法。或使用“for each”循环和`String`的`toCharArray`方法。

## 讨论

字符串的`charAt()`方法通过索引号（从零开始）从`String`对象中检索给定字符。由于 Unicode 已经扩展到超过 16 位，不是所有的 Unicode 字符都能容纳在 Java 的`char`变量中。因此有一个类似的`codePointAt()`方法，其返回类型是`int`。要处理`String`中的所有字符，可以使用一个从零到`String.length()-1`的`for`循环。这里我们处理`String`中的所有字符：

*main/src/main/java/strings/strings/StrCharAt.java*

```java
public class StrCharAt {
    public static void main(String[] av) {
        String a = "A quick bronze fox";
        for (int i=0; i < a.length(); i++) { // no forEach, need the index
            String message = String.format(
                "charAt is '%c', codePointAt is %3d, casted it's '%c'",
                     a.charAt(i),
                     a.codePointAt(i),
                     (char)a.codePointAt(i));
            System.out.println(message);
        }
    }
}
```

鉴于“for each”循环已经存在很久，你可能原谅你期望能够像这样写一些东西`for (char ch : myString) {…}`。不幸的是，这并不起作用。但是你可以使用`myString.toCharArray()`如下所示：

```java
public class ForEachChar {
    public static void main(String[] args) {
        String mesg = "Hello world";

        // Does not compile, Strings are not iterable
        // for (char ch : mesg) {
        //        System.out.println(ch);
        // }

        System.out.println("Using toCharArray:");
        for (char ch : mesg.toCharArray()) {
            System.out.println(ch);
        }

        System.out.println("Using Streams:");
        mesg.chars().forEach(c -> System.out.println((char)c));
    }
}
```

*校验和*是表示和确认文件内容的数字量。如果你将文件的校验和与内容分开传输，接收方可以对文件进行校验——假设算法是已知的——并验证文件是否完整接收。Example 3-4 展示了一种最简单的校验和计算方法，只需将每个字符的数值相加。注意，在文件上，它不包括换行符的值；为了修正这一点，检索`System​.getProp⁠erty("line.separator");`并将其字符值添加到每行的末尾。或者放弃行模式，逐个字符读取文件。

##### Example 3-4\. main/src/main/java/strings/CheckSum.java

```java
    /** CheckSum one text file, given an open BufferedReader.
 * Checksum does not include line endings, so will give the
 * same value for given text on any platform. Do not use
 * on binary files!
 */
    public static int process(BufferedReader is) {
        int sum = 0;
        try {
            String inputLine;

            while ((inputLine = is.readLine()) != null) {
                for (char c : inputLine.toCharArray()) {
                    sum += c;
                }
            }
        } catch (IOException e) {
            throw new RuntimeException("IOException: " + e);
        }
        return sum;
    }
```

# 3.4 对齐、缩进和取消缩进字符串

## 问题

你想将字符串左对齐、右对齐或居中。

## 解决方案

自己做数学运算，使用 `substring`（参见 Recipe 3.1） 和 `StringBuilder`（参见 Recipe 3.2）。或者，使用我的 `StringAlign` 类，该类基于 `java.text.Format` 类。对于左对齐或右对齐，使用 `String.format()`。

## 讨论

文本居中和对齐经常会出现。假设你想打印一个简单的报告，并希望页面数字居中显示。标准 API 中似乎没有任何可以完全帮助你完成工作的东西。但我写了一个名为 `String​A⁠lign` 的类可以做到。下面是你可能如何使用它的方法：

```java
public class StringAlignSimple {

    public static void main(String[] args) {
        // Construct a "formatter" to center strings.
        StringAlign formatter = new StringAlign(70, StringAlign.Justify.CENTER);
        // Try it out, for page "i"
        System.out.println(formatter.format("- i -"));
        // Try it out, for page 4\. Since this formatter is
        // optimized for Strings, not specifically for page numbers,
        // we have to convert the number to a String
        System.out.println(formatter.format(Integer.toString(4)));
    }
}
```

如果你编译并运行这个类，它会打印出两个演示行号，并使它们居中显示，如下所示：

```java
> javac -d . StringAlignSimple.java
> java strings.StringAlignSimple
                                - i -
                                  4
>
```

示例 3-5 是 `StringAlign` 类的代码。注意，该类扩展了 `java.text` 包中的 `Format` 类。有一系列至少有一个名为 `format()` 的方法的 `Format` 类。因此，它与许多其他格式化程序（如 `DateFormat` 和 `NumberFormat`）一起，我们将在接下来的章节中看到。

##### 示例 3-5\. main/src/main/java/strings/StringAlign.java

```java
public class StringAlign extends Format {

    private static final long serialVersionUID = 1L;

    public enum Justify {
        /* Constant for left justification. */
        LEFT,
        /* Constant for centering. */
        CENTER,
        /** Constant for right-justified Strings. */
        RIGHT,
    }

    /** Current justification */
    private Justify just;
    /** Current max length */
    private int maxChars;

    /** Construct a StringAlign formatter; length and alignment are
 * passed to the Constructor instead of each format() call as the
 * expected common use is in repetitive formatting e.g., page numbers.
 * @param maxChars - the maximum length of the output
 * @param just - one of the enum values LEFT, CENTER or RIGHT
 */
    public StringAlign(int maxChars, Justify just) {
        switch(just) {
        case LEFT:
        case CENTER:
        case RIGHT:
            this.just = just;
            break;
        default:
            throw new IllegalArgumentException("invalid justification arg.");
        }
        if (maxChars < 0) {
            throw new IllegalArgumentException("maxChars must be positive.");
        }
        this.maxChars = maxChars;
    }

    /** Format a String.
 * @param input - the string to be aligned.
 * @parm where - the StringBuilder to append it to.
 * @param ignore - a FieldPosition (may be null, not used but
 * specified by the general contract of Format).
 */
    @Override
    public StringBuffer format(
        Object input, StringBuffer where, FieldPosition ignore)  {

        String s = input.toString();
        String wanted = s.substring(0, Math.min(s.length(), maxChars));

        // Get the spaces in the right place.
        switch (just) {
            case RIGHT:
                pad(where, maxChars - wanted.length());
                where.append(wanted);
                break;
            case CENTER:
                int toAdd = maxChars - wanted.length();
                pad(where, toAdd/2);
                where.append(wanted);
                pad(where, toAdd - toAdd/2);
                break;
            case LEFT:
                where.append(wanted);
                pad(where, maxChars - wanted.length());
                break;
            }
        return where;
    }

    protected final void pad(StringBuffer to, int howMany) {
        for (int i=0; i<howMany; i++)
            to.append(' ');
    }

    /** Convenience Routine */
    String format(String s) {
        return format(s, new StringBuffer(), null).toString();
    }

    /** ParseObject is required, but not useful here. */
    public Object parseObject (String source, ParsePosition pos)  {
        return source;
    }
}
```

Java 12 引入了一个名为 `public String indent(int n)` 的新方法，该方法将 *n* 个空格添加到字符串前面，字符串被视为带有换行符的行序列。这与 Java 11 的 `Stream<String> lines()` 方法配合使用效果很好。例如，对于一系列已经存储在单个字符串中的行需要相同的缩进的情况（`Streams` 和 “::” 符号的使用在 Recipe 9.0 中有介绍）：

```java
jshell> "abc\ndef".indent(30).lines().forEach(System.out::println);
                              abc
                              def

jshell> "abc\ndef".indent(30).indent(-10).lines().forEach(System.out::println);
                    abc
                    def

jshell>
```

## 参见

数字列的对齐考虑在 第 5 章 中。

# 3.5 Unicode 字符和字符串之间的转换

## 问题

你想在 Unicode 字符和 `String` 之间转换。

## 解决方案

使用 Java `char` 或 `String` 数据类型处理字符；这些类型本身支持 Unicode。如果需要，将字符打印为整数以显示它们的 *原始* 值。

## 讨论

Unicode 是一个国际标准，旨在表示人们在各种语言中使用的所有已知字符。尽管原始的 ASCII 字符集是其子集，但 Unicode 是庞大的。在 Java 创建时，Unicode 是一个 16 位字符集，因此将 Java 的 `char` 值设为 16 位宽度似乎是自然的选择，多年来，`char` 可以容纳任何 Unicode 字符。然而，随着时间的推移，Unicode 不断增长，现在包括超过一百万个代码点或字符，远远超过了 16 位可以表示的 65,525 个字符。[⁵] 并非所有可能的 16 位值都被定义为 UCS-2 中的字符，UCS-2 是 Java 中最初使用的 16 位版本的 Unicode。一些被保留为转义字符，允许将多字符长度映射到不常见的字符。幸运的是，有一个中间标准，称为 UTF-16（16 位 Unicode 转换格式）。正如 `String` 类文档所述：

> `String` 表示 UTF-16 格式中的字符串，其中*补充字符*由*代理对*表示（有关更多信息，请参见`Character` 类中 Unicode 字符表示部分）。索引值指的是字符代码单元，因此补充字符在 `String` 中使用两个位置。
> 
> `String` 类提供了处理 Unicode 代码点（即字符）的方法，除了处理 Unicode 代码单元（即 `char values`）的方法。

`String` 的 `charAt()` 方法返回指定偏移量处字符的 `char` 值。`StringBuilder append()` 方法有一种形式接受一个 `char`。因为 `char` 是一个整数类型，你甚至可以对 `char` 进行算术运算，尽管这不像在 C 中那样频繁需要。也不常推荐，因为 `Character` 类提供了在诸如 C 语言中通常使用这些操作的方法。这里是一个使用 `char` 算术控制循环并将字符附加到 `StringBuilder` 中的程序（见 Recipe 3.2）：

```java
        // UnicodeChars.java
        StringBuilder b = new StringBuilder();
        for (char c = 'a'; c<'d'; c++) {
            b.append(c);
        }
        b.append('\u00a5');    // Japanese Yen symbol
        b.append('\u01FC');    // Roman AE with acute accent
        b.append('\u0391');    // GREEK Capital Alpha
        b.append('\u03A9');    // GREEK Capital Omega

        for (int i=0; i<b.length(); i++) {
            System.out.printf(
                "Character #%d (%04x) is %c%n",
                i, (int)b.charAt(i), b.charAt(i));
        }
        System.out.println("Accumulated characters are " + b);
```

当你运行它时，预期的结果将为 ASCII 字符打印出来。在 Unix 和 Mac 系统中，默认字体不包括所有附加字符，因此它们要么被省略，要么映射到不规则字符：

```java
$ java -cp target/classes strings.UnicodeChars
Character #0 (0061) is a
Character #1 (0062) is b
Character #2 (0063) is c
Character #3 (00a5) is ¥
Character #4 (01fc) is Ǽ
Character #5 (0391) is Α
Character #6 (03a9) is Ω
Accumulated characters are abc¥ǼΑΩ
$
```

Windows 系统也不包含大多数这些字符，但至少它打印出它知道缺失的字符作为问号（Windows 系统字体比各种 Unix 系统更同质化，因此更容易知道哪些不起作用）。另一方面，它尝试将日元符号打印为带有波浪号的 N：

```java
Character #0 is a
Character #1 is b
Character #2 is c
Character #3 is ¥
Character #4 is ?
Character #5 is ?
Character #6 is ?
Accumulated characters are abc¥___
```

“_” 字符是不可打印的字符。

## 另请参见

本书在线源代码中的`Unicode`程序显示 Unicode 字符集的任何 256 字符部分。您可以从[Unicode Consortium](http://www.unicode.org)下载列出 Unicode 字符集中每个字符的文档。

# 3.6 按单词或按字符反转字符串

## 问题

您希望逐字符或逐单词反转字符串。

## 解决方案

您可以轻松地按字符反转字符串，使用`StringBuilder`。有几种方法可以按单词反转字符串。一种自然的方法是使用`StringTokenizer`和一个堆栈。`Stack`是一个类（定义在`java.util`中；参见 Recipe 7.16），实现了一个易于使用的后进先出（LIFO）对象堆栈。

## 讨论

要反转字符串中的字符，请使用`StringBuilder reverse()`方法：

*main/src/main/java/strings/StringRevChar.java*

```java
String sh = "FCGDAEB";
System.out.println(sh + " -> " + new StringBuilder(sh).reverse());
```

这个例子中的字母列出了西方音乐调号中升调的顺序；反过来，它列出了降调的顺序。当然，您也可以自己反转字符，使用逐字符模式（参见 Recipe 3.3）。

一种流行的记忆法或助记符，帮助音乐学生记住升调和降调的顺序，是用每个升调的一个单词而不是一个字母。让我们逐个*单词地*反转这一点。 示例 3-6 将每个添加到一个`Stack`（参见 Recipe 7.16），然后以 LIFO 顺序处理整个批次，以颠倒顺序。

##### 示例 3-6\. main/src/main/java/strings/StringReverse.java

```java
        String s = "Father Charles Goes Down And Ends Battle";

        // Put it in the stack frontwards
        Stack<String> myStack = new Stack<>();
        StringTokenizer st = new StringTokenizer(s);
        while (st.hasMoreTokens()) {
            myStack.push(st.nextToken());
        }

        // Print the stack backwards
        System.out.print('"' + s + '"' + " backwards by word is:\n\t\"");
        while (!myStack.empty()) {
            System.out.print(myStack.pop());
            System.out.print(' ');    // inter-word spacing
        }
        System.out.println('"');
```

# 3.7 扩展和压缩制表符

## 问题

您需要在文件中将空格字符转换为制表符字符，或者反之。您可能希望将空格替换为制表符以节省磁盘空间，或者反之以处理无法处理制表符的设备或程序。

## 解决方案

使用我的`Tabs`类或其子类`EnTab`。

## 讨论

因为处理带制表符文本或数据的程序期望制表位于固定位置，您不能使用典型的文本编辑器替换制表符为空格或反之。 示例 3-7 是`EnTab`的列表，包含一个示例主程序。该程序逐行工作。对于每行的每个字符，如果字符是空格，我们看看是否可以与之前的空格合并以输出单个制表符字符。该程序依赖于`Tabs`类，我们稍后会讲到。`Tabs`类用于确定哪些列位置代表制表位，哪些不是。

##### 示例 3-7\. main/src/main/java/strings/Entab.java

```java
public class EnTab {

    private static Logger logger = Logger.getLogger(EnTab.class.getSimpleName());

    /** The Tabs (tab logic handler) */
    protected Tabs tabs;

    /**
 * Delegate tab spacing information to tabs.
 */
    public int getTabSpacing() {
        return tabs.getTabSpacing();
    }

    /**
 * Main program: just create an EnTab object, and pass the standard input
 * or the named file(s) through it.
 */
    public static void main(String[] argv) throws IOException {
        EnTab et = new EnTab(8);
        if (argv.length == 0) // do standard input
            et.entab(
                new BufferedReader(new InputStreamReader(System.in)),
                System.out);
        else
            for (String fileName : argv) { // do each file
                et.entab(
                    new BufferedReader(new FileReader(fileName)),
                    System.out);
            }
    }

    /**
 * Constructor: just save the tab values.
 * @param n The number of spaces each tab is to replace.
 */
    public EnTab(int n) {
        tabs = new Tabs(n);
    }

    public EnTab() {
        tabs = new Tabs();
    }

    /**
 * entab: process one file, replacing blanks with tabs.
 * @param is A BufferedReader opened to the file to be read.
 * @param out a PrintWriter to send the output to.
 */
    public void entab(BufferedReader is, PrintWriter out) throws IOException {

        // main loop: process entire file one line at a time.
        is.lines().forEach(line -> {
            out.println(entabLine(line));
        });
    }

    /**
 * entab: process one file, replacing blanks with tabs.
 *
 * @param is A BufferedReader opened to the file to be read.
 * @param out A PrintStream to write the output to.
 */
    public void entab(BufferedReader is, PrintStream out) throws IOException {
        entab(is, new PrintWriter(out));
    }

    /**
 * entabLine: process one line, replacing blanks with tabs.
 * @param line the string to be processed
 */
    public String entabLine(String line) {
        int N = line.length(), outCol = 0;
        StringBuilder sb = new StringBuilder();
        char ch;
        int consumedSpaces = 0;

        for (int inCol = 0; inCol < N; inCol++) { // Cannot use foreach here
            ch = line.charAt(inCol);
            // If we get a space, consume it, don't output it.
            // If this takes us to a tab stop, output a tab character.
            if (ch == ' ') {
                logger.info("Got space at " + inCol);
                if (tabs.isTabStop(inCol)) {
                    logger.info("Got a Tab Stop " + inCol);
                    sb.append('\t');
                    outCol += consumedSpaces;
                    consumedSpaces = 0;
                } else {
                    consumedSpaces++;
                }
                continue;
            }

            // We're at a non-space; if we're just past a tab stop, we need
            // to put the "leftover" spaces back out, since we consumed
            // them above.
            while (inCol-1 > outCol) {
                logger.info("Padding space at " + inCol);
                sb.append(' ');
                outCol++;
            }

            // Now we have a plain character to output.
            sb.append(ch);
            outCol++;

        }
        // If line ended with trailing (or only!) spaces, preserve them.
        for (int i = 0; i < consumedSpaces; i++) {
            logger.info("Padding space at end # " + i);
            sb.append(' ');
        }
        return sb.toString();
    }
}
```

这段代码是模仿 Kernighan 和 Plauger 经典著作《软件工具》中的一个程序编写的。虽然他们的版本是用一种叫做 RatFor（Rational Fortran）的语言编写的，但我的版本经过多次翻译。他们的版本实际上是逐个字符处理的，我很长一段时间都试图保留这种整体结构。最终，我将其重写为逐行处理的程序。

反方向进行操作——将制表符插入而不是删除——的程序是`DeTab`类，示例见 Example 3-8；这里只展示了核心方法。

##### 示例 3-8\. main/src/main/java/strings/DeTab.java

```java
public class DeTab {
    Tabs ts;

    public static void main(String[] argv) throws IOException {
        DeTab dt = new DeTab(8);
        dt.detab(new BufferedReader(new InputStreamReader(System.in)),
                new PrintWriter(System.out));
    }

    public DeTab(int n) {
        ts = new Tabs(n);
    }
    public DeTab() {
        ts = new Tabs();
    }

    /** detab one file (replace tabs with spaces)
 * @param is - the file to be processed
 * @param out - the updated file
 */
    public void detab(BufferedReader is, PrintWriter out) throws IOException {
        is.lines().forEach(line -> {
            out.println(detabLine(line));
        });
    }

    /** detab one line (replace tabs with spaces)
 * @param line - the line to be processed
 * @return the updated line
 */
    public String detabLine(String line) {
        char c;
        int col;
        StringBuilder sb = new StringBuilder();
        col = 0;
        for (int i = 0; i < line.length(); i++) {
            // Either ordinary character or tab.
            if ((c = line.charAt(i)) != '\t') {
                sb.append(c); // Ordinary
                ++col;
                continue;
            }
            do { // Tab, expand it, must put >=1 space
                sb.append(' ');
            } while (!ts.isTabStop(++col));
        }
        return sb.toString();
    }
}
```

`Tabs`类提供了两个方法：`settabpos()`和`istabstop()`。Example 3-9 是`Tabs`类的源代码。

##### 示例 3-9\. main/src/main/java/strings/Tabs.java

```java
public class Tabs {
    /** tabs every so often */
    public final static int DEFTABSPACE =   8;
    /** the current tab stop setting. */
    protected int tabSpace = DEFTABSPACE;
    /** the longest line that we initially set tabs for */
    public final static int MAXLINE  = 255;

    /** Construct a Tabs object with a given tab stop settings */
    public Tabs(int n) {
        if (n <= 0) {
            n = 1;
        }
        tabSpace = n;
    }

    /** Construct a Tabs object with a default tab stop settings */
    public Tabs() {
        this(DEFTABSPACE);
    }

    /**
 * @return Returns the tabSpace.
 */
    public int getTabSpacing() {
        return tabSpace;
    }

    /** isTabStop - returns true if given column is a tab stop.
 * @param col - the current column number
 */
    public boolean isTabStop(int col) {
        if (col <= 0)
            return false;
        return (col+1) % tabSpace == 0;
    }
}
```

# 3.8 控制大小写

## 问题

您需要将字符串转换为大写或小写，或者在比较字符串时忽略大小写。

## 解决方案

`String`类有多个用于处理特定情况下文档的方法。`toUpperCase()`和`toLowerCase()`分别返回一个新的字符串副本，按照其名称进行转换。每个方法都可以使用无参数或使用指定转换规则的`Locale`参数调用；这是因为国际化的需要。Java 的 API 提供了重要的国际化和本地化功能，详见“Ian's Basic Steps: Internationalization and Localization”。而`equals()`方法告诉您另一个字符串是否完全相同，`equalsIgnoreCase()`告诉您所有字符是否相同，而不考虑大小写。在这里，您不能指定替代区域设置；系统使用默认区域设置：

```java
        String name = "Java Cookbook";
        System.out.println("Normal:\t" + name);
        System.out.println("Upper:\t" + name.toUpperCase());
        System.out.println("Lower:\t" + name.toLowerCase());
        String javaName = "java cookBook"; // If it were Java identifiers :-)
        if (!name.equals(javaName))
            System.err.println("equals() correctly reports false");
        else
            System.err.println("equals() incorrectly reports true");
        if (name.equalsIgnoreCase(javaName))
            System.err.println("equalsIgnoreCase() correctly reports true");
        else
            System.err.println("equalsIgnoreCase() incorrectly reports false");
```

如果您运行此代码，它将打印第一个名称的大写和小写版本，然后报告两种方法按预期工作：

```java
C:\javasrc\strings>java strings.Case
Normal: Java Cookbook
Upper:  JAVA COOKBOOK
Lower:  java cookbook
equals( ) correctly reports false
equalsIgnoreCase( ) correctly reports true
```

## 参见

正则表达式使得在字符串搜索中忽略大小写更加简单（见 Chapter 4）。

# 3.9 输入不可打印字符

## 问题

你需要将不可打印字符放入字符串中。

## 解决方案

使用反斜杠字符和 Java 字符串转义序列之一。

## 讨论

Java 字符串转义列在 Table 3-1 中。

表格 3-1\. 字符串转义

| 获取 | 使用 | 注释 |
| --- | --- | --- |
| 制表符 | `\t` |  |
| 换行符（Unix 换行符） | `\n` | 调用`System.getProperty("line.separator")`将给出平台的行尾。 |
| 回车符 | `\r` |  |
| 换页符 | `\f` |  |
| 退格符 | `\b` |  |
| 单引号 | `\`' |  |
| 双引号 | `\`" |  |
| Unicode 字符 | `\u` *`NNNN`* | 四个十六进制数字（无`\x`，类似于 C/C++）。参见[*http://www.unicode.org*](http://www.unicode.org)获取代码。 |
| 八进制（！）字符 | +\+*`NNN`* | 现在谁还使用八进制（基数 8）？ |
| 反斜杠 | `\\` |  |

这里是一个代码示例，展示了其中大部分功能：

```java
public class StringEscapes {
    public static void main(String[] argv) {
        System.out.println("Java Strings in action:");
        // System.out.println("An alarm or alert: \a");    // not supported
        System.out.println("An alarm entered in Octal: \007");
        System.out.println("A tab key: \t(what comes after)");
        System.out.println("A newline: \n(what comes after)");
        System.out.println("A UniCode character: \u0207");
        System.out.println("A backslash character: \\");
    }
}
```

如果你有很多非 ASCII 字符需要输入，你可能希望考虑使用 Java 的输入方法，这在[在线文档](https://docs.oracle.com/javase/8/docs/technotes/guides/imf/index.html)中简要讨论过。

# 3.10 剥离字符串末尾的空白

## 问题

你需要处理用户可能输入的额外前导或尾随空格的字符串。

## 解决方案

使用 `String` 类的 `strip()` 或 `trim()` 方法。

## 讨论

`String` 类中有四种方法可以实现这个功能：

`strip()`

返回一个去除了所有前导和尾随空白的字符串

`stripLeading()`

返回一个去除了所有前导空白的字符串

`stripTrailing()`

返回去除了所有尾随空白的字符串

`String trim()`

返回去除了所有前导和尾随空格的字符串

对于 `strip()` 方法，空白由 `Character.isSpace()` 定义。对于 `trim()` 方法，空格包括任何数值小于或等于 32 的字符，或 *U+0020*（空格字符）。

示例 3-10 使用 `trim()` 来剥离 Java 源代码行中任意数量的前导空格和/或制表符，以便查找字符 `//+` 和 `//-`。这些字符串是我之前用来标记本书中要包含在印刷本中的程序部分的特殊 Java 注释。

##### 示例 3-10\. main/src/main/java/strings/GetMark.java（剥离和比较字符串）

```java
public class GetMark {
    /** the default starting mark */
    public final String START_MARK = "//+";
    /** the default ending mark */
    public final String END_MARK = "//-";
    /** Set this to TRUE for running in "exclude" mode (e.g., for
 * building exercises from solutions) and to FALSE for running
 * in "extract" mode (e.g., writing a book and omitting the
 * imports and "public class" stuff).
 */
    public final static boolean START = true;
    /** True if we are currently inside marks */
    protected boolean printing = START;
    /** True if you want line numbers */
    protected final boolean number = false;

    /** Get Marked parts of one file, given an open LineNumberReader.
 * This is the main operation of this class, and can be used
 * inside other programs or from the main() wrapper.
 */
    public void process(String fileName,
        LineNumberReader is,
        PrintStream out) {
        int nLines = 0;
        try {
            String inputLine;

            while ((inputLine = is.readLine()) != null) {
                if (inputLine.trim().equals(START_MARK)) {
                    if (printing)
                        // These go to stderr, so you can redirect the output
                        System.err.println("ERROR: START INSIDE START, " +
                            fileName + ':' + is.getLineNumber());
                    printing = true;
                } else if (inputLine.trim().equals(END_MARK)) {
                    if (!printing)
                        System.err.println("ERROR: STOP WHILE STOPPED, " +
                            fileName + ':' + is.getLineNumber());
                    printing = false;
                } else if (printing) {
                    if (number) {
                        out.print(nLines);
                        out.print(": ");
                    }
                    out.println(inputLine);
                    ++nLines;
                }
            }
            is.close();
            out.flush(); // Must not close - caller may still need it.
            if (nLines == 0)
                System.err.println("ERROR: No marks in " + fileName +
                    "; no output generated!");
        } catch (IOException e) {
            System.out.println("IOException: " + e);
        }
    }
```

# 3.11 使用 I18N 资源创建消息

## 问题

你希望你的程序接受敏感性培训，以便能够在国际上良好地沟通。

## 解决方案

你的程序必须通过国际化软件获取所有控件和消息字符串。下面是方法：

1.  获取一个 `ResourceBundle`：

    ```java
    ResourceBundle rb = ResourceBundle.getBundle("Menus");
    ```

    我会在食谱 3.13 中讨论 `ResourceBundle`，但简要来说，`ResourceBundle` 表示一组名称-值对（资源）。名称是您分配给每个 GUI 控件或其他用户界面文本的名称，而值是分配给每个控件的文本在给定语言中的文本。

1.  使用这个 `ResourceBundle` 来获取每个控件名称的本地化版本。

    旧方法：

    ```java
    String label = "Exit";
    // Create the control, e.g., new JButton(label);
    ```

    新方法：

    ```java
    try { label = rb.getString("exit.label"); }
    catch (MissingResourceException e) { label="Exit"; } // fallback
    // Create the control, e.g., new JButton(label);
    ```

对于一个控件来说，这可能是相当多的代码，但是你可以编写一个方便的例程来简化它，就像这样：

```java
JButton exitButton = I18NUtil.getButton("exit.label", "Exit");
```

文件 *I18NUtil.java* 包含在书的代码分发中。

虽然示例是一个 Swing `JButton`，但同样的方法也适用于其他 UI，比如 Web 层。例如，在 JSF 中，你可以将字符串放在一个名为 *resources.properties* 的属性文件中，并将其存储在 *src/main/resources* 中。你可以在 *faces-config.xml* 中加载这个文件：

```java
  <application>
    <locale-config>
        <default-locale>en</default-locale>
        <supported-locale>en</supported-locale>
        <supported-locale>es</supported-locale>
        <supported-locale>fr</supported-locale>
    </locale-config>
    <resource-bundle>
        <base-name>resources</base-name>
        <var>msg</var>
    </resource-bundle>
  </application>
```

然后在每个需要这些字符串的 Web 页面中，使用表达式中的 `msg` 变量引用资源：

```java
// In signup.xhtml:
<h:outputText value="#{msg.prompt_firstname}"/>
<h:inputText required="true" id="firstName" value="#{person.firstName}" />
```

### 运行时会发生什么？

使用默认区域设置，因为我们没有指定任何一个。默认区域设置是依赖于平台的：

Unix/POSIX

LANG 环境变量（每个用户）

Windows

控制面板→区域设置

macOS

系统首选项→语言与文本

其他

查看平台文档

`ResourceBundle.getBundle()`会查找具有指定资源包名称的文件（在前面的示例中为`Menus`），加上下划线和语言环境名称（如果设置了非默认语言环境），再加上另一个下划线和语言环境变体（如果设置了变体），最后加上扩展名*.properties*。如果设置了变体但找不到文件，则会回退到只使用国家代码。如果连国家代码也找不到，则会回退到原始默认设置。表 3-2 显示了各种语言环境的示例。

请注意，Android 应用程序通常使用类似的机制（但是使用 XML 格式的文件而不是 Java 属性文件，并且在找到属性文件的文件名时进行了一些小的更改）。

表 3-2\. 不同语言环境的属性文件名

| 语言环境 | 文件名 |
| --- | --- |
| 默认语言环境 | *Menus.Properties* |
| 瑞典语 | *Menus_sv.properties* |
| 西班牙语 | *Menus_es.properties* |
| 法语 | *Menus_fr.properties* |
| 加拿大法语 | *Menus_fr_CA.properties* |

语言环境名称是两个字母的 ISO-639 语言代码（小写），通常是国家的*本土名称*（其语言使用者称呼它的名称）的缩写；因此，瑞典是*sv*表示*Sverige*，西班牙是*es*表示*Español*，等等。语言环境变体是两个字母的 ISO 国家代码（大写）；例如，CA 表示加拿大，US 表示美国，SV 表示瑞典，ES 表示西班牙，等等。

### 设置语言环境

在 Windows 上，进入控制面板中的区域设置。更改此设置可能需要重新启动，因此请关闭所有编辑器窗口。

在 Unix 上，设置您的`LANG`环境变量。例如，墨西哥的 Korn shell 用户可能在她的*.profile*文件中有如下行：

```java
export LANG=es_MX
```

在任一系统上，要测试不同的语言环境，只需在运行时通过命令行选项`-D`定义语言环境，例如：

```java
java -Duser.language=es i18n.Browser
```

在西班牙语环境中运行名为`Browser`的 Java 程序，位于`i18n`包中。

您可以通过调用`Locale.getAvailable​Lo⁠cales()`获取可用语言环境的列表。

# 3.12 使用特定的语言环境

## 问题

在特定操作中，您可能希望使用除默认语言环境之外的语言环境。

## 解决方案

使用预定义实例或`Locale`构造函数来获取`Locale`。如果需要，可以使用`Locale.setDefault(newLocale)`将其设置为应用程序的全局默认值。

## 讨论

提供格式化服务的类，如`DateTimeFormatter`和`NumberFormat`，提供了重载方法，可以带有或不带有与`Locale`相关的参数来调用。

要获取`Locale`对象，可以使用`Locale`类提供的预定义语言变量之一，或者构造自己的`Locale`对象，提供语言代码和国家代码：

```java
Locale locale1 = Locale.FRANCE;    // predefined
Locale locale2 = new Locale("en", "UK");    // English, UK version
```

这些语言环境可以在各种格式化操作中使用：

```java
DateFormat frDateFormatter = DateFormat.getDateInstance(
		DateFormat.MEDIUM, frLocale);
DateFormat ukDateFormatter = DateFormat.getDateInstance(
		DateFormat.MEDIUM, ukLocale);
```

可以使用其中任何一种来格式化日期或数字，如`Use​Lo⁠cales`类中所示：

```java
package i18n;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.time.format.FormatStyle;
import java.util.Locale;

/** Use some locales; based on user's OS "settings"
 * choices or -Duser.lang= or -Duser.region=.
 */
public class UseLocales {
    public static void main(String[] args) {

        Locale frLocale = Locale.FRANCE;    // predefined
        Locale ukLocale = new Locale("en", "UK");    // English, UK version

        DateTimeFormatter defaultDateFormatter =
            DateTimeFormatter.ofLocalizedDateTime(
                FormatStyle.MEDIUM);
        DateTimeFormatter frDateFormatter =
            DateTimeFormatter.ofLocalizedDateTime(
                FormatStyle.MEDIUM).localizedBy(frLocale);
        DateTimeFormatter ukDateFormatter =
            DateTimeFormatter.ofLocalizedDateTime(
                FormatStyle.MEDIUM).localizedBy(ukLocale);

        LocalDateTime now = LocalDateTime.now();
        System.out.println("Default: " + ' ' +
            now.format(defaultDateFormatter));
        System.out.println(frLocale.getDisplayName() + ' ' +
            now.format(frDateFormatter));
        System.out.println(ukLocale.getDisplayName() + ' ' +
            now.format(ukDateFormatter));
    }
}
```

程序在每个区域设置中打印区域名称并格式化日期：

```java
$ java i18n.UseLocales
Default:  Oct 16, 2019, 4:41:45 PM
French (France) 16 oct. 2019 à 16:41:45
English (UK) Oct 16, 2019, 4:41:45 PM$

```

# 3.13 创建资源包

## 问题

您需要为国际化（I18N）创建一个资源包。

## 解决方案

资源包只是名称和值的集合。您可以编写一个`java.util.ResourceBundle`子类，但创建文本`Properties`文件（参见 Recipe 7.10）然后使用`ResourceBundle.getBundle()`加载它们更容易。可以使用任何纯文本编辑器创建这些文件。将其保留在文本文件格式中还允许桌面应用程序中的用户进行自定义；不提供其语言或希望根据方言的地方变化更改措辞的用户应能够编辑文件。

请注意，资源包文本文件不应与任何您的 Java 类具有相同的名称。原因是`ResourceBundle`动态构建一个与资源文件同名的类。

## 讨论

这里是一份用于几个菜单项的示例属性文件：

```java
# Default Menu properties
# The File Menu
file.label=File Menu
file.new.label=New File
file.new.key=N
file.save.label=Save
file.new.key=S
```

创建默认属性文件通常不是问题，但为其他语言创建属性文件可能会是问题。除非您是一家大型跨国公司，否则可能没有资源（请原谅双关语）在内部创建资源文件。如果您正在发布商业软件或利用全球网络，您需要确定目标市场并了解其中哪些最关注希望以其语言显示菜单等内容。然后，聘请专业的翻译服务公司，该公司具有编制所需语言文件的专业知识。在发货前务必进行充分测试，就像您对软件的任何其他部分所做的那样。

如果您需要特殊字符、多行文本或其他复杂条目，请记住`ResourceBundle`也是`Properties`文件，因此请参阅`java.util.Properties`的文档。

# 3.14 程序：一个简单的文本格式化程序

此程序是一个简单的文本格式化程序，代表了在大多数计算平台上在独立图形化字处理器、激光打印机，以及最终桌面出版和办公套件兴起之前人们所使用的程序。它简单地从先前用文本编辑器创建的文件中读取单词，并在达到右边距时输出这些单词，然后调用`println()`以添加行结束符。例如，这是一个输入文件：

```java
It's a nice
day, isn't it, Mr. Mxyzzptllxy?
I think we should
go for a walk.
```

给定该文件作为输入，`Fmt`程序会将格式化后的行打印出来：

```java
It's a nice day, isn't it, Mr. Mxyzzptllxy? I think we should go for a
walk.
```

正如您所见，它将我们提供的文本适应到边距，并且丢弃原始文本中的所有换行符。以下是代码：

```java
public class Fmt {
    /** The maximum column width */
    public static final int COLWIDTH=72;
    /** The file that we read and format */
    final BufferedReader in;
    /** Where the output goes */
    PrintWriter out;

    /** If files present, format each one, else format the standard input. */
    public static void main(String[] av) throws IOException {
        if (av.length == 0)
            new Fmt(System.in).format();
        else for (String name : av) {
            new Fmt(name).format();
        }
    }

    public Fmt(BufferedReader inFile, PrintWriter outFile) {
        this.in = inFile;
        this.out = outFile;
    }

    public Fmt(PrintWriter out) {
        this(new BufferedReader(new InputStreamReader(System.in)), out);
    }

    /** Construct a Formatter given an open Reader */
    public Fmt(BufferedReader file) throws IOException {
        this(file, new PrintWriter(System.out));
    }

    /** Construct a Formatter given a filename */
    public Fmt(String fname) throws IOException {
        this(new BufferedReader(new FileReader(fname)));
    }

    /** Construct a Formatter given an open Stream */
    public Fmt(InputStream file) throws IOException {
        this(new BufferedReader(new InputStreamReader(file)));
    }

    /** Format the File contained in a constructed Fmt object */
    public void format() throws IOException {
        format(in.lines(), out);
    }

    /** Format a Stream of lines, e.g., bufReader.lines() */
    public static void format(Stream<String> s, PrintWriter out) {
        StringBuilder outBuf = new StringBuilder();
        s.forEachOrdered((line -> {
            if (line.length() == 0) {    // null line
                out.println(outBuf);    // end current line
                out.println();    // output blank line
                outBuf.setLength(0);
            } else {
                // otherwise it's text, so format it.
                StringTokenizer st = new StringTokenizer(line);
                while (st.hasMoreTokens()) {
                    String word = st.nextToken();

                    // If this word would go past the margin,
                    // first dump out anything previous.
                    if (outBuf.length() + word.length() > COLWIDTH) {
                        out.println(outBuf);
                        outBuf.setLength(0);
                    }
                    outBuf.append(word).append(' ');
                }
            }
        }));
        if (outBuf.length() > 0) {
            out.println(outBuf);
        } else {
            out.println();
        }
    }

}
```

这个程序的稍微花哨一点的版本，`Fmt2`，位于这本书的在线源代码中。它使用*dot commands*——以句点开头的行——来对格式进行有限控制。一系列点命令格式化程序包括 Unix 的*roff*，*nroff, troff*和*groff*，它们与 Digital Equipment 系统上称为*runoff*的程序处于同一系列中。这个的原始版本是 J. Saltzer 的*runoff*，它首先出现在 Multics 上，然后传入各种操作系统。为了节约树木，我没有在这里包含`Fmt2`；它是`Fmt`的子类，并重写了`format()`方法以包含附加功能（源代码在本书的完整*javasrc*代码库中）。

# 3.15 程序：Soundex 姓名比较

比较美式姓名的困难启发了美国人口调查局在 20 世纪初开发了 Soundex 算法。给定一组辅音，每个辅音都映射到一个特定的数字，其效果是将发音相似的姓名归为一类，因为在那个年代许多人不识字，无法一致拼写他们的姓氏。但是即使今天，例如在公司范围内的电话簿应用中，它仍然很有用。姓名 Darwin 和 Derwin 映射为 D650，而 Darwent 映射为 D653，这使得它邻近 D650。所有这些都被认为是相同姓名的历史变体。假设我们需要将包含这些姓名的行进行排序：如果我们可以在每行开头输出 Soundex 号码，这将很容易。以下是`Soundex`类的简单演示：

```java
public class SoundexSimple {

    /** main */
    public static void main(String[] args) {
        String[] names = {
            "Darwin, Ian",
            "Davidson, Greg",
            "Darwent, William",
            "Derwin, Daemon"
        };
        for (String name : names) {
            System.out.println(Soundex.soundex(name) + ' ' + name);
        }
    }
}
```

让我们来运行它：

```java
> javac -d . SoundexSimple.java
> java strings.SoundexSimple | sort
D132 Davidson, Greg
D650 Darwin, Ian
D650 Derwin, Daemon
D653 Darwent, William
>
```

正如你所见，Darwin-variant 姓名（包括 Daemon Derwin^(6））都被一起排序，并且与 Davidson（以及 Davis，Davies 等）的姓名在简单字母排序时出现在 Darwin 和 Derwin 之间的姓名是不同的。Soundex 算法已经完成了它的工作。

这里是`Soundex`类本身——它使用`String`和`StringBuilder`将姓名转换为 Soundex 代码：

*main/src/main/java/strings/Soundex.java*

```java
public class Soundex {

    static boolean debug = false;

    /* Implements the mapping
 * from: AEHIOUWYBFPVCGJKQSXZDTLMNR
 * to:   00000000111122222222334556
 */
    public static final char[] MAP = {
        //A  B   C   D   E   F   G   H   I   J   K   L   M
        '0','1','2','3','0','1','2','0','0','2','2','4','5',
        //N  O   P   W   R   S   T   U   V   W   X   Y   Z
        '5','0','1','2','6','2','3','0','1','0','2','0','2'
    };

    /** Convert the given String to its Soundex code.
 * @return null If the given string can't be mapped to Soundex.
 */
    public static String soundex(String s) {

        // Algorithm works on uppercase (mainframe era).
        String t = s.toUpperCase();

        StringBuilder res = new StringBuilder();
        char c, prev = '?', prevOutput = '?';

        // Main loop: find up to 4 chars that map.
        for (int i=0; i<t.length() && res.length() < 4 &&
            (c = t.charAt(i)) != ','; i++) {

            // Check to see if the given character is alphabetic.
            // Text is already converted to uppercase. Algorithm
            // only handles ASCII letters, do NOT use Character.isLetter()!
            // Also, skip double letters.
            if (c>='A' && c<='Z' && c != prev) {
                prev = c;

                // First char is installed unchanged, for sorting.
                if (i==0) {
                    res.append(c);
                } else {
                    char m = MAP[c-'A'];
                    if (debug) {
                        System.out.println(c + " --> " + m);
                    }
                    if (m != '0' && m != prevOutput) {
                        res.append(m);
                        prevOutput = m;
                    }
                }
            }
        }
        if (res.length() == 0)
            return null;
        for (int i=res.length(); i<4; i++)
            res.append('0');
        return res.toString();
    }
```

显然，这个应用程序没有实现完整的 Soundex 算法的一些细微差别。使用 JUnit 进行更完整的测试（参见 Recipe 1.10）也可以在线进行，名为*SoundexTest.java*，位于*src/tests/java/strings*目录中。热心的读者可以使用此来引发这些细微差别的失败，并发送拉取请求，更新测试和代码的版本。

## 参见

Levenshtein 字符串编辑距离算法可以用于以不同方式进行近似字符串比较。你可以在[Apache Commons StringUtils](http://commons.apache.org/proper/commons-lang)中找到这个算法。我展示了这个算法的非 Java（Perl）实现，在 Recipe 18.5 中可以找到。

^(1) 这两个`.equals()`调用在**除了第一个可能会抛出`NullPointerException`之外**是等价的。

^(2) `StringBuilder` 是在 Java 5 中添加的。它在功能上等同于旧版的 `StringBuffer`。我们将在 Recipe 3.2 中详细讨论细节。

^(3) 除非，也许，你更新个人记录的速度和我一样慢。

^(4) `String` 和 `StringBuilder` 在它们实现的 `CharSequence` 接口中有几个强制相同的方法。

^(5) 实际上，Unicode 中有如此多的字符，以至于出现了一种潮流，使用近似拉丁字母的倒置版本字符显示你的名字倒置。在网络上搜索“倒置 Unicode”即可了解更多。

^(6) 在 Unix 术语中，守护进程（daemon）是一个服务器。这个古老的英文单词与撒旦的恶魔无关，而是指一个帮助者或助手。Derwin Daemon 实际上是 Susannah Coleman 的 *Source Wars* 在线漫画中的一个角色，很久以前曾经在一个现在已经关闭的网站 *darby.daemonnews.org* 上在线。
