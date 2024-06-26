# 附录 B. 练习答案

本附录包含每章末尾的复习问题答案（通常还包含一些背景信息）。代码练习的答案随附示例程序的源码下载，存储在 *exercises* 文件夹中。附录 A 中详细介绍了获取源码并在 IntelliJ IDEA 中设置的方法。

# 第一章：现代语言

1.  哪家公司当前维护 Java？

    尽管 Java 是在 1990 年代由 Sun Microsystems 开发的，但 Oracle 在 2009 年购买了 Sun（因此也购买了 Java）。Oracle 拥有并积极参与其商业 JDK 和开源 OpenJDK 的开发和分发。

1.  Java 的开源开发工具包的名称是什么？

    JDK 的开源版本被称为 OpenJDK。

1.  名称 Java 安全运行字节码的两个主要组件。

    Java 具有许多与安全相关的特性，但在每个 Java 应用程序中起作用的主要组件是类加载器和字节码验证器。

# 第二章：首个应用程序

1.  您应该使用什么命令来编译 Java 源文件？

    如果您在终端中工作，*javac* 命令会编译 Java 源文件。虽然在使用像 IntelliJ IDEA 这样的 IDE 时，细节通常被隐藏，但是这些 IDE 在幕后也使用 *javac*。

1.  当您运行 Java 应用程序时，JVM 如何知道从哪里开始？

    任何可执行的 Java 类必须定义 `main()` 方法。JVM 使用此方法作为入口点。

1.  在创建新类时，您可以扩展多个类吗？

    不。Java 不支持从多个单独的类直接进行多重继承。

1.  在创建新类时，您可以实现多个接口吗？

    是的。Java 允许您实现尽可能多的接口。使用接口为程序员提供了多重继承的大部分有用功能，而避开了许多陷阱。

1.  哪个类代表图形应用程序中的窗口？

    `JFrame` 类代表 Java 图形应用程序中使用的主窗口，尽管后续章节将介绍一些也能创建特定窗口的低级类。

## 代码练习

一般情况下，我们不会在本附录中列出代码解决方案，但我们希望可以轻松地检查您对这个第一个程序的解决方案。简单文本版的“Goodbye, Java!” 应该看起来类似于这样：

```java
public class GoodbyeJava {
  public static void main(String[] args) {
    System.out.println("Goodbye, Java!");
  }
}
```

对于图形版本，您的代码应该类似于这样：

```java
import javax.swing.*;

public class GoodbyeJava {
  public static void main(String[] args) {
    JFrame frame = new JFrame("Chapter 2 Exercises");
    frame.setSize(300, 150);
    frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    JLabel label = new JLabel("Goodbye, Java!", JLabel.CENTER);
    frame.add(label);
    frame.setVisible(true);
  }
}
```

请注意，我们添加了额外的 `EXIT_ON_CLOSE`，这是在 `HelloJava2` 中引入的，以便在关闭应用程序时正确退出。如果您使用 IDEA，可以使用 IDE 内部的绿色播放按钮运行任何一个类。如果您使用终端，可以切换到 *GoodbyeJava.java* 所在的目录并输入以下命令：

```java
% javac GoodbyeJava.java
% java GoodbyeJava
```

# 第三章：工具

1.  哪个语句允许您在应用程序中访问 Swing 组件？

    `import` 语句从指定的类或包加载编译器需要的信息。对于 Swing 组件，您通常使用 `import javax.swing.*;` 导入整个包。

1.  什么环境变量确定 Java 在编译或执行时查找类文件的位置？

    `CLASSPATH` 环境变量保存了一个包含其他类或 JAR 文件的目录列表，这些文件可用于编译和执行。如果您使用的是 IDE，`CLASSPATH` 仍然被定义，但通常不需要您自己编辑。

1.  您可以使用哪些选项查看 JAR 文件的内容而不解压缩它？

    您可以运行以下命令来显示 JAR 文件的内容，而不实际将其解压到当前目录：

    ```java
    % jar tvf MyApp.jar
    ```

    `tvf` 标志代表目录（`t`）、详细信息（`v`）和文件（`f` 后跟文件名）的表格。

1.  使 JAR 文件可执行所需的 *MANIFEST.MF* 文件条目是什么？

    您必须包含一个 `Main-Class` 条目，该条目给出具有有效 `main()` 方法的类的完全限定名称，以使给定的 JAR 文件可执行。

1.  什么工具允许您交互式地尝试 Java 代码？

    您可以从终端运行 *jshell* 来交互式地尝试简单的 Java 代码。

## 代码练习

您将在 *ch03/solutions* 文件夹中找到我们对代码练习的解决方案。(附录 A 中有关于下载示例的详细信息。) 我们的解决方案并不是解决这些问题的唯一或者最佳方式。我们尝试呈现整洁、可维护的代码，并遵循最佳实践，但解决编码问题还有其他方法。希望您能够编写和运行自己的答案，但如果遇到困难，这里有一些提示。

要创建可执行的 *hello.jar* 文件，我们将在 *ch03/exercises* 文件夹中的终端中进行所有工作。（您当然也可以在[IDEA 中进行](https://oreil.ly/l0akz)此类工作。）请打开终端并切换到该文件夹。

在创建 JAR 文件本身之前，我们需要编辑 *manifest.mf* 文件。添加 `Main-Class` 条目。最终文件应如下所示：

```java
Manifest-Version: 1.0
Created-By: 11.0.16
Main-Class: ch03.exercises.HelloJar
```

现在，您可以使用以下命令创建和测试您的 JAR 文件：

```java
% jar cvmf manifest.mf hello.jar *.class
% java -jar hello.jar
```

请记住，标志中的 `m` 元素是必要的，以包含我们的清单。还值得提醒的是，`m` 和 `f` 标志的顺序决定了随后跟随的 *manifest.mf* 和 *hello.jar* 命令行参数的顺序。您还记得如何查看您新创建的 JAR 的内容以验证清单是否存在吗？¹

# 第四章：Java 语言

1.  Java 在编译类时默认使用什么文本编码格式？

    默认情况下，Java 使用 8 位 Unicode 转换格式（UTF-8）编码。8 位（或一个字节）编码可以容纳单字节和多字节字符。

1.  用于包围多行注释的字符是什么？这些注释可以嵌套吗？

    Java 借鉴了 C 和 C++的注释语法。单行注释以双斜杠（`//`）开头，而多行注释则用`/*`和`*/`括起来。多行样式也可以用于在代码行中嵌入小注释。

1.  Java 支持哪些循环结构？

    Java 支持`for`循环（传统的 C 风格和用于遍历集合的增强形式）、`while`循环和`do/while`循环。

1.  在`if/else if/else`测试链中，如果多个条件都为真会发生什么？

    与第一个评估为`true`的测试相关联的块将被执行。在该块完成后，控制将在整个链路后继续执行——无论有多少其他测试也会返回`true`。

1.  如果你想要将美国股市的总市值（大约在 2022 财年结束时为 31 万亿美元）存储为整数，你可以使用什么原始数据类型？

    你可以使用`long`整型；它可以存储高达 9 千亿（正负数皆可）的数字。虽然你也可以使用`double`类型，但随着数字变大，精度会降低。而且，“整数”意味着没有小数部分，整数类型更为合理。

1.  表达式`18 – 7 * 2`的计算结果是什么？

    这是一个优先级问题，以确保你的高中代数老师最终得到了一些肯定，经历了所有那些“但我什么时候会用到这个？”的质疑。首先进行 7 和 2 的乘法运算，然后进行减法运算。最终答案是 4。（你可能得到了 22，这是从左到右执行操作的结果。如果你确实需要那个结果，你可以将`18 – 7`部分用括号括起来……就像这个旁注一样。）

1.  如何创建一个包含一周的天数名称的数组？

    可以使用花括号创建和初始化数组。对于一周的天数，我们需要一个`String`数组，就像这样：

    ```java
        String[] dayNames = {
          "Sunday", "Monday", "Tuesday", "Wednesday",
          "Thursday", "Friday", "Saturday"
        };
    ```

    列表中名称的间距是可选的。你可以将所有内容列在一行上，将每个名称列在单独的行上，或者像我们在这里做的那样，采用一些组合。

## 代码练习

1.  有多种方法可以将原始的`a`和`b`以及计算得到的最大公约数打印到屏幕上。你可以在开始计算之前使用`print()`语句（*而不是* `println()`），然后在计算结束时使用`println()`打印答案。或者你可以在开始计算之前将`a`和`b`复制到第二组变量中。找到答案后，你可以打印出复制的值以及结果。

1.  要以简单的行形式输出三角形数据，可以使用填充三角形的相同嵌套循环：

    ```java
        for (int i; i < triangle.length; i++)
          for (int j; j < triangle[i].length; j++)
            System.out.println(triangle[i][j]);
    ```

## 高级练习

1.  要在视觉三角形中呈现输出，可以在内部`j`循环中使用`print()`语句。（确保在每个数字后打印一个空格。）内部循环完成后，可以使用空的`println()`来结束该行并准备下一行。

# 第五章：Java 中的对象

1.  Java 中的主要组织单元是什么？

    Java 中的主要“单元”是一个类。当然，许多其他结构也起着重要作用，但是你至少需要一个类才能使用其他任何东西。

1.  你使用什么运算符来从类创建一个对象（或实例）？

    `new` 操作符实例化一个类的对象并调用适当的构造函数。

1.  Java 不支持经典的多重继承。Java 提供哪些机制作为替代？

    Java 使用接口来完成许多其他面向对象语言中多重继承的目标。

1.  如何组织多个相关的类？

    你将相关的类放在一个包中。在你的文件系统中，包显示为嵌套文件夹。在你的代码中，包使用点分隔的名称。

1.  如何将其他包中的类包含到你自己的代码中以供使用？

    你可以 `import` 其他单独的类或整个包供你自己使用。

1.  如何称呼定义在另一个类范围内的类？在某些情况下，使这样的类变得有用的一些特性是什么？

    在另一个类的大括号内部（不仅仅在同一个文件中）定义的简单类称为内部类。内部类具有对外部类的所有变量和方法的访问权限，包括私有成员。它们可以帮助将代码分割成可管理和可重用的片段，同时提供对谁可以使用它们的良好控制。

1.  如何称呼旨在被重写的方法，它具有名称、返回类型和参数列表，但没有主体？

    只有它们的签名定义的方法称为抽象方法。在类中包含抽象方法也使得该类成为抽象类。抽象类不能被实例化。你必须创建一个子类，然后为抽象方法提供一个真实的体来使用它。

1.  什么是重载方法？

    Java 允许你使用相同的方法名以不同类型或数量的参数。如果两个方法共享相同的名称，它们被称为重载。重载使得在不同参数上执行相同逻辑工作的方法批量创建成为可能。Java 中重载方法的经典示例是 `System.out.println()`，它可以接受多种类型的参数并将它们全部转换为字符串以打印到终端。

1.  如果你希望确保没有其他类可以使用你定义的变量，你应该使用什么访问修饰符？

    `private` 访问修饰符用于变量（或方法，或者整个内部类），将其使用限制在它定义的类中。

## 代码练习

1.  对于我们的 `Zoo` 中的第一个问题，你只需在内部 `Gibbon` 类的空 `speak()` 方法中添加一个 `print()` 语句。希望 `Lion` 的示例容易跟随。

1.  添加另一个动物也应该很简单；你可以复制整个`Lion`类。重命名类并在`speak()`方法中打印适当的声音。你还需要复制`listen()`方法中的几行代码，以便将你的动物声音添加到输出中。

1.  为了重构`listen()`方法，我们注意到每个动物的输出非常相似，但显然每个动物的名称都会改变。如果我们将这个名称移到动物各自的类中，我们就可以创建一个循环，其主体打印出一个动物的细节（名称和声音）。然后我们迭代我们的动物数组。如果你再创建另一个动物，你只需要将你的新内部类的实例添加到数组中。

1.  此练习的`AppleToss`类是`exercises.ch05`包的一部分。（*game*文件夹包含了已完成的游戏，其中包含了我们将在本书中构建的所有功能。该文件夹中的类属于`game`包。您可以编译和运行该版本，但它有一些我们尚未讨论的功能。）要从终端编译游戏，你可以进入*ch05/exercises*目录并在那里编译 Java 类，或者留在你解压源代码的顶层文件夹中，并在编译时给出路径：

    ```java
    % javac ch05/exercises/AppleToss.java
    ```

    要运行游戏，你需要在顶层文件夹中。从那里，你可以使用*java*命令运行`exercises.ch05.AppleToss`类。

## 高级练习

1.  希望添加一个`Hedge`看起来很简单。你可以从`Tree`类的副本开始。将文件重命名为*Hedge.java*。编辑类以反映我们的新`Hedge`障碍，并更新其`paintComponent()`方法。在`Field`类内部，你需要为`Hedge`添加一个成员变量。创建一个类似于`setupTree()`的`setupHedge()`方法，并确保在`Field`的`paintComponent()`方法中包含你的树篱。

    最后，但肯定不是最不重要的，更新`setupFieldForOnePlayer()`方法以调用我们的`setupHedge()`方法。编译并运行游戏，就像你在前面的练习中做的那样。你的新树篱应该会出现！

# 第六章：错误处理

1.  在你的代码中，你使用什么语句来处理潜在的异常？

    你可以在可能生成异常的任何语句或语句组周围使用`try/catch`块。

1.  编译器要求你处理或抛出哪些异常？

    在 Java 中，术语*checked exception*指的是编译器理解并要求程序员承认的异常类别。你可以在可能发生 checked exceptions 的方法中使用`try/catch`块，或者你可以在方法的签名中的`throws`子句中添加异常。

1.  在`try`块中使用完资源后，你会把任何“清理”代码放在哪里？

    `finally`子句将在`try`块结束时无论发生什么都会运行。如果没有问题，`finally`子句中的代码将运行。如果有异常并且`catch`块处理了它，`finally`仍然会运行。如果发生未处理的异常，`finally`子句在控制转移回调用方法之前仍会运行。

1.  禁用断言会对性能造成很大的惩罚吗？

    不是的。这是设计如此。断言通常更多用于开发或调试。当你关闭它们时，它们会被跳过。即使在生产应用中，你可能也会在代码中保留断言。如果用户报告了问题，可以临时打开断言以允许用户收集任何输出并帮助你找到原因。

## 代码练习

1.  要使*Pause.java*文件编译通过，你需要在调用`Thread.sleep()`周围添加一个`try/catch`块。对于这个简单的练习，你只需要封装`Thread.sleep()`行。

1.  我们需要的断言语句将具有以下形式：

    ```java
        assert x > 0 : "X is too small";
        assert y > 0 : "Y is too small";
    ```

    更重要的问题是：我们应该把它们放在哪里？我们只需要检查消息的起始位置，所以我们不希望断言在`paintComponent()`方法内部。更好的地方可能是在`HelloComponent0()`构造函数中，在我们存储提供的`message`参数之后。

    要测试断言，你需要编辑源文件以更改`x`或`y`的值并重新编译。

## 高级练习

1.  你的`GCDException`类可能看起来像这样：

    ```java
    package ch06.exercises;

    public class GCDException extends Exception {
      private int badA;
      private int badB;

      GCDException(int a, int b) {
        super("No common factors for " + a + ", " + b);
        badA = a;
        badB = b;
      }

      public int getA() { return badA; }
      public int getB() { return badB; }
    }
    ```

    你可以用一个简单的`if`语句测试你的 GCD 计算结果。如果结果是 1，你可以使用我们原来的`a`和`b`作为参数调用你的新`GCDException`构造函数，像这样：

    ```java
        if (a == 1) {
          throw new GCDException(a1, b1);
        }
        // ...
    ```

# 第七章：集合和泛型

1.  如果你想存储一个包含姓名和电话号码的联系人列表，哪种集合类型最适合？

    `Map`是个好办法。键可以是简单的字符串，包含联系人的姓名，值可以是一个简单（尽管包装过的）长数字。或者你可以创建一个`Person`类和一个`PhoneNumber`类，然后地图可以使用你的自定义类。

1.  你用什么方法为`Set`中的项目获取迭代器？

    `Collection`接口中富有创意的`iterator()`方法将为你获取迭代器。

1.  如何将`List`转换为数组？

    你可以使用`toArray()`方法将`List`转换为`Object`类型的数组或列表的参数化类型的数组。

1.  如何将数组转换为`List`？

    `Arrays`辅助类包括方便的`asList()`方法，接受一个数组并返回相同类型的参数化列表。

1.  要使用`Collections.sort()`方法对列表进行排序，你应该实现什么接口？

    尽管有许多方法可以对集合进行排序，但`Comparable`对象列表（表示其类实现了`Comparable`接口的对象）可以使用`Collections`辅助类提供的标准`sort()`方法。

## 代码练习

1.  正如我们在本章中提到的，您不能像对列表或数组进行排序一样直接对简单映射进行排序。甚至`Set`通常也不可排序。²不过，您可以对列表进行排序，因此使用`keySet()`方法填充列表应该可以满足您的需求：

    ```java
        List<Integer> ids = new ArrayList<>(employees.keySet());
        ids.sort();
        for (Integer id : ids) {
          System.out.println(id + ": " + employees.get(id));
        }
    ```

1.  希望你对于支持多种对冲的扩展感觉很直观。我们主要是复制已有的树代码。使用`List`允许我们使用增强的`for`循环快速遍历所有对冲：

    ```java
    // File: ch07/exercises/Field.java
      List<Hedge> hedges = new ArrayList<>();
      // ...

      protected void paintComponent(Graphics g) {
        // ...
        for (Hedge h : hedges) {
          h.draw(g);
        }
        // ...
      }
    ```

## 高级练习

1.  您可以使用`values()`输出创建并排序类似于代码练习 1 解决方案的列表。这个练习的有趣部分是使用`Employee`类实现`Comparable`接口。（实际上，在*ch07/solutions*文件夹中，可排序的员工类是`Employee2`。我们希望将原始的`Employee`类保留为第一个练习的有效解决方案。）这是一个使用员工姓名进行字符串比较的示例：

    ```java
    public class Employee2 implements Comparable<Employee2> {
      // ...
      public int compareTo(Employee2 other) {
        // Let's be a little fancy and sort on a constructed name
        String myName = last + ", " + first;
        String otherName = other.last + ", " + other.first;
        return myName.compareToIgnoreCase(otherName);
      }
      // ...
    }
    ```

    当然，您可以使用其他`Employee`属性进行其他比较。尝试玩弄一些其他排序，并查看是否得到您预期的结果。如果想进一步深入，请查看`java.util.TreeMap`类，以一种无需列表转换的方式将员工存储为排序方式。

# 第八章：文本和核心工具

1.  哪个类包含常量π？需要导入该类来使用π吗？

    `java.lang.Math`类包含常量`PI`。`java.lang`包中的所有类都会默认导入；使用它们无需显式`import`。

1.  哪个包含原始`java.util.Date`类的新的、更好的替代品？

    `java.time`包包含各种质量类，用于处理日期、时间、时间戳（或由日期和时间组成的“时刻”）以及时间跨度或持续时间。

1.  你使用哪个类来为用户友好输出格式化日期？

    `java.text`包中的`DateFormat`类具有非常灵活（有时不透明）的格式化引擎，用于呈现日期和时间。

1.  正则表达式中使用什么符号来帮助匹配单词“yes”和“yup”？

    您可以使用交替运算符`|`（竖线）创建表达式，例如`yes|yup`，用作模式。

1.  如何将字符串“42”转换为整数 42？

    各种数值包装类都有字符串转换方法。对于像 42 这样的整数，`Integer.parseInt()`方法是适用的。包装类都属于`java.lang`包。

1.  如何比较两个字符串以查看它们是否匹配，忽略大小写，例如“yes”和“YES”？

    `String`类有两个主要的比较方法：`equals()`和`equalsIgnoreCase()`。后者会忽略大小写，顾名思义。

1.  哪个运算符允许简单的字符串连接？

    Java 通常不支持运算符重载，但加号（`+`）在与数字基本类型一起使用时执行加法，在与`String`对象一起使用时执行连接。如果你使用`+`来“添加”一个字符串和一个数字，结果将是一个字符串。（因此，`7 + "is lucky"`将得到字符串“7is lucky”。注意，连接不会插入任何空格。如果你要组装一个典型的句子，你必须在部分之间添加自己的空格。）

## 代码练习

有很多方法可以完成这个练习的目标。测试参数数量应该很简单。然后，你可以使用`String`类的一些特性来判断你是否有随机关键字或一对坐标。你可以使用`split()`方法分割坐标，或者编写一个正则表达式来分离数值。在创建随机坐标时，你可以使用`Math.random()`，类似于我们在“数学实践中”中为游戏定位树木的方式。

# 第九章：线程

1.  什么是线程？

    线程代表程序内的“执行线程”。线程有自己的状态，并且可以独立于其他线程运行。通常，你使用线程来处理可以放在后台运行的长时间任务，而更重要的任务可以继续进行。Java 既有平台线程（与操作系统提供的本地线程一一对应）又有虚拟线程（纯 Java 构造，保留了本地线程的语义和好处，但没有操作系统的开销）。

1.  如果你希望线程在调用方法时“轮流”执行（即不希望两个线程同时执行该方法以避免破坏共享数据），你可以为该方法添加哪个关键字？

    你可以在任何读取或写入共享数据的方法上使用`synchronized`修饰符。如果两个线程需要使用同一个方法，第一个线程设置一个锁，阻止第二个线程调用该方法。一旦第一个线程完成，锁将被清除，第二个线程可以继续。

1.  哪些标志允许你编译包含预览特性代码的 Java 程序？

    在编译依赖于预览特性的 Java 类时，你必须提供`--enable-preview`以及`-source`或`--release`标志给*javac*。

1.  哪些标志允许你运行包含预览特性代码的 Java 程序？

    运行包含预览特性的编译类时，你只需要提供`--enable-preview`标志。

1.  一个本地线程能支持多少平台线程？

    只有一个。使用`Thread`类创建一个平台线程，并带有`Runnable`目标或使用`java.util.concurrent`包中的`ExecutorService`类，都需要操作系统提供一个线程。

1.  单个本机线程可以支持多少个虚拟线程？

    一个单独的本机线程可以支持许多虚拟线程。Project Loom 旨在将 Java 程序中使用的线程与操作系统管理的线程分开。对于某些场景，轻量级虚拟线程在 Java 负责其调度时表现更佳。虚拟线程与本机线程的数量之间没有固定的比率，但虚拟线程的关键洞见是，其数量不与本机线程的数量挂钩。

1.  对于`int`变量`x`，语句`x = x + 1;`是否是原子操作？

    尽管这看起来是一个小操作，但涉及到几个低级步骤。这些低级步骤中的任何一个都可能被中断，并且`x`的值可能会受到不利影响。如果需要保证线程安全的增量，可以使用`AtomicInteger`或将语句包装在同步块中。

1.  哪个包含了像`Queue`和`Map`这样的流行集合类的线程安全版本？

    `java.util.concurrent`包含几个 Java 定义为“并发”的集合类，例如`ConcurrentLinkedQueue`和`ConcurrentHashMap`。并发除了纯线程安全的读写之外，还涉及几个其他行为，但线程安全是得到保证的。

## 代码练习

1.  你将会在`startClock()`方法内完成大部分工作。（当然，除了使用的 AWT 和 Swing 包外，你仍需导入其他内容。）你可以创建一个单独的类、内部类或匿名内部类来处理时钟更新循环。记住，可以通过调用其`repaint()`方法请求 GUI 元素的刷新。Java 支持几种“无限”循环的机制。你可以使用像`while (true) { …​ }`这样的结构，或者巧妙命名的“forever”循环：`for (;;) { …​ }`。一切就绪后，别忘了启动你的线程！

1.  希望对你来说，这个练习相对简单。作为 Java 现有代码库与虚拟线程的整体兼容性的证明，你只需更改演示苹果投掷动画开始的几行代码。在这个游戏的这一轮中，所有设置和启动代码都发生在`Field`类中。查找类似`new Thread()`或`new Runnable()`的代码。你应该能够在不做任何修改的情况下重用实际的动画逻辑。

# 第十章：文件输入和输出

1.  如何检查给定的文件是否已经存在？

    有几种方法可以检查文件是否存在，但其中两种最简单的方法依赖于`java.io`或`java.nio`包中的辅助方法。`java.io.File`的实例可以使用`exists()`方法。静态的`java.nio.file.Files.exists()`方法可以测试`Path`对象以查看所表示的文件是否存在。

1.  如果必须使用旧的编码方案（如 ISO 8859）处理遗留文本文件，如何设置读取器以正确将内容转换为 UTF-8？

    你可以向`FileReader`的构造函数提供适当的字符集（`java.nio.charset.Charset`），安全地将文件转换为 Java 字符串。

1.  哪个包中有最适合非阻塞文件 I/O 的类？

    `java.nio`包及其子包的主要特性之一是支持非阻塞 I/O。

1.  你可能会使用哪种类型的输入流来解析二进制文件，比如 JPEG 压缩的图片？

    从`java.io`，你可以使用`DataInputStream`类。对于 NIO，通道和缓冲区（如`ByteBuffer`）自然地支持二进制数据。

1.  `System`类内置了哪三个标准文本流？

    `System`类提供了两个输出流，`System.out`和`System.err`，以及一个输入流`System.in`。这些流分别连接到操作系统的`stdout`、`stderr`和`stdin`句柄。

1.  绝对路径从根目录开始（例如`/`或`C:\`）。相对路径从哪里开始？更具体地说，相对路径相对于什么？

    相对路径是相对于“工作目录”的，通常这是你启动程序的地方，如果你使用命令行启动你的应用程序。在大多数 IDE 中，工作目录是可以配置的。

1.  如何从现有的`FileInputStream`中检索一个 NIO 通道？

    如果你已经有一个`FileInputStream`实例，你可以使用它的`getChannel()`方法返回与输入流关联的`FileChannel`。

## 代码练习

1.  我们的第一个`Count`迭代只需使用本章讨论的其中一个工具。你可以使用作为命令行参数给定路径的`File`类。从那里，`exists()`方法将告诉你是否可以继续，或者是否应该打印友好的错误消息；`length()`方法将给出文件的大小，以字节为单位。（此示例的解决方案是位于*ch10/solutions*文件夹中的`Count1.java`。）

1.  对于第二次迭代，在给定文件中显示行数和单词数，你需要读取和解析文件的内容。其中一个`Reader`类会很好，但有多种读取文本文件的方式。无论如何打开文件，你可以计算每一行，然后用`String.split()`或正则表达式将该行分解为单词。（此练习的解决方案是`Count2.java`。）

1.  这第三个版本中没有任何新功能，但我们希望你能借此机会尝试一些 NIO 类和方法。看看`java.nio.file.Files`类的方法。你会惊讶于这个助手类有多大帮助！（此练习的解决方案是`Count3.java`。）

## 高级练习

1.  对于这次最终的升级，您可以写入文件或通道。根据您选择在版本 2 或 3 中读取内容的方式，这可能代表我们类中的相当重要的添加。您需要检查确保第二个参数（如果给定！）是可写的。然后使用允许追加的类之一，如`RandomAccessFile`，或为`FileChannel`包括`APPEND`选项。（此练习的解决方案是`Count4.java`。我们使用了之前的`Count3`与 NIO，但您可以从`Count2`开始并使用标准 I/O 类。）

# 第十一章：Java 中的函数式方法

1.  哪个包含大多数功能接口？

    虽然函数式接口分散在整个 JDK 中，但您会发现大多数“官方”接口定义在`java.util.function`包中。我们在“官方”上加了引号，因为任何具有单个抽象方法（SAM）的接口都可以视为函数式接口。

1.  在编译或运行使用像 lambda 这样的函数特性的 Java 应用程序时，是否需要使用任何特殊标志？

    是的。目前 Java 中的许多函数式编程功能都是 JDK 的完整成员。编译或执行使用它们的 Java 代码时不需要预览或功能标志。

1.  如何创建具有多个语句的 lambda 表达式主体？

    lambda 表达式的主体遵循与诸如`while`循环之类的主体相同的规则：单个语句不需要括号括起来，但多个语句需要。如果您的 lambda 返回一个值，您也可以使用标准的`return`语句。

1.  lambda 表达式可以是 void 吗？它们可以返回值吗？

    在这两个方面都是可以的。Lambda 表达式可以运行与方法相同的各种选项。您可以有不接受参数且不返回值的 lambda 表达式。您可以有消耗参数但不产生结果的 lambda 表达式。您可以有不接受参数但返回值的 lambda 生成器。最后，您可以有接受一个或多个参数并返回值的 lambda 表达式。

1.  在处理完流后能否重用它？

    不行。一旦您开始处理流，那就是它了。试图重用流将导致异常。如果需要，您通常可以重用原始源来创建一个全新但完全相同的流。

1.  如何将对象流转换为整数流？

    您可以使用`Stream`类的`mapToInt()`变体之一：`mapToInt()`，`flatMapToInt()`或`mapMultiToInt()`。反过来，`IntStream`类有一个`mapToObj()`方法以在相反方向进行转换。

1.  如果您有一个从文件中过滤空行的流，您可能会使用什么操作来告诉您有多少行包含内容？

    计算剩余行数的最简单方法是使用`count()`终端操作。你也可以创建自己的规约器，或者使用一个收集器然后查询结果列表的长度。

## 代码练习

1.  希望我们对调整使用更有趣的公式感到顺利。我们不需要任何替代语法或额外方法；我们只需将摄氏度转换公式 C =（F - 32）* 5 / 9 放入我们的 lambda 体中，如下所示：

    ```java
        System.out.print("Converting to Celsius: ");
        System.out.println(adjust(sample, s -> (s - 32) * 5 / 9));
    ```

    这并不是非常引人注目，但我们想指出，Lambda 表达式可以开启一些非常聪明的可能性，这些可能性超出了最初的计划。

1.  对于这种平均值任务，你有多种选择可用。你可以编写一个平均值规约器。你可以将工资收集到一个更简单的容器中，然后编写自己的平均代码。但是，如果你查看不同流的文档，你会注意到数值流已经有了完美的操作：`average()`。它返回一个`OptionalDouble`对象。你仍然需要启动流，然后使用类似`mapToInt()`的东西来获取你的数值流。

## 高级练习

1.  `groupingBy()`收集器需要一个从流的每个元素中提取键并返回键与所有具有匹配键的元素列表的映射的函数。对于我们的`PaidEmployee`示例，你可能会有类似这样的内容：

    ```java
        Map<String, List<PaidEmployee>> byRoles =
          employees.stream().collect(
          Collectors.groupingBy(PaidEmployee::getRole));
    ```

    我们映射中键的类型必须与我们在`groupingBy()`操作中提取的对象类型相匹配。我们在这里使用了一个方法引用，但是任何返回员工角色的 lambda 也将起作用。

    我们不想使先前的解决方案复杂化，所以我们复制了报告和员工类，并分别命名为`Report2`和`PaidEmployee2`。

# 第十二章：桌面应用程序

1.  你会用哪个组件向用户显示一些文本？

    虽然你可以使用多种基于文本的组件，但`JLabel`是向用户展示一些（只读）文本信息的最简单方式。

1.  你会用哪些组件来允许用户输入文本？

    根据用户期望获得的信息量，你可以使用`JTextField`或`JTextArea`。 （还有其他文本组件存在，但它们提供更专业化的用途。）

1.  单击按钮会生成什么事件？

    单击按钮或类似按钮的组件（如`JMenuItem`）会生成`ActionEvent`。

1.  如果你想知道用户何时更改所选项目，应该将监听器附加到`JList`。

    你可以实现来自`javax.swing.event`包的`ListSelectionListener`来接收`JList`对象的列表选择（和取消选择）事件。

1.  `JPanel`的默认布局管理器是什么？

    默认情况下，`JPanel`使用`FlowLayout`管理器。这个默认的一个显著例外是`JFrame`的内容窗格。该窗格是一个`JPanel`，但是框架会自动将窗格的管理器更改为`BorderLayout`。

1.  在 Java 中，哪个线程负责处理事件？

    事件分发线程，有时被称为事件分发队列，管理着事件的传递和屏幕上组件的更新。

1.  在后台任务完成后，你会用什么方法来更新像`JLabel`这样的组件？

    如果你希望在处理其他事件之前等待标签更新完成，可以使用`SwingUtilities.invokeAndWait()`。如果不在乎标签何时更新完成，可以使用`Swing` `Utilities.invokeLater()`。

1.  什么容器持有`JMenuItem`对象？

    `JMenu`对象可以持有`JMenuItem`对象以及嵌套的`JMenu`对象。菜单本身则包含在`JMenuBar`中。

## 代码练习

1.  你可以以两种方式处理计算器布局：使用嵌套面板或使用`GridBagLayout`。（我们在 *ch12/solutions/Calculator.java* 中的解决方案使用了嵌套面板来布置按钮。）从简单开始。将文本字段添加到框架顶部。然后在中心添加一个按钮。现在决定如何处理添加剩余按钮。如果你的按钮使用了`Calculator`实例（使用我们在 “Shadowing” 中讨论的关键字`this`作为它们的监听器），你应该看到任何点击的按钮标签都会打印到终端上。

1.  这个练习并不需要太多新的图形代码。但是你需要在 UI 事件线程中安全地修改场地上显示的障碍物。你可以从简单的打印消息或使用`JOptionPane`来显示警告开始慢慢进行，只要苹果碰到树或篱笆就触发。在你对距离测量有信心后，回顾一下如何从列表中移除对象。移除障碍物后，记得重新绘制场地。

## 高级练习

1.  计算器的逻辑相对简单，但绝对不是琐碎的。首先让各种数字按钮（1、2、3 等）与显示器连接起来。你需要追加数字来创建完整的数字。当用户点击“–”等操作按钮时，将当前显示的数字及将来使用的操作存储起来。让用户输入第二个数字。点击“=”应该存储这第二个数字，然后执行实际的计算。将结果显示在显示器上，然后让用户重新开始。

    在完整的专业计算器应用程序中有许多（许多！）微妙之处。如果你的早期尝试限制为仅处理单个数字，不要担心。这个练习的重点是练习响应事件。即使只是在用户点击按钮后让一个数字显示在显示字段中，也值得庆祝！

# 第十三章：Java 网络编程

1.  `URL`类默认支持哪些网络协议？

    Java 的`URL`类包含对 HTTP、HTTPS 和 FTP 协议的支持。这三种协议涵盖了在线资源的大部分内容，但如果你处理除 Web 或文件服务器之外的系统，你可以创建自己的协议处理程序。

1.  你可以使用 Java 从在线源下载二进制数据吗？

    可以。字节流是 Java 中所有网络数据的核心。你可以读取原始字节，或者可以链接其他更高级别的流。例如，`InputStreamReader`和`BufferedReader`非常适合文本。`DataInputStream`可以处理二进制数据。

1.  如何使用 Java 将表单数据发送到 Web 服务器？（不需要完整功能的应用程序，我们只想让你考虑涉及的高级步骤和涉及的 Java 类。）

    你可以使用`URL`类打开到 Web 服务器的连接。在发出任何请求之前，你可以配置双向通信的连接。HTTP `POST`命令允许你在请求的正文中向服务器发送数据。

1.  你用什么类来监听传入的网络连接？

    你使用`java.net.ServerSocket`类来创建一个网络监听器。

1.  当像我们的游戏一样创建自己的服务器时，有关于选择端口号的规则吗？

    可以。端口号必须在 0 到 65,535 之间，通常为小于 1,024 的端口保留给常见服务，通常需要特殊权限才能使用。

1.  用 Java 编写的服务器应用程序可以支持多个同时客户端吗？

    是的。虽然你只能在给定端口上创建一个`ServerSocket`，但你可以接受数百甚至数千个客户端并在线程中处理他们的请求。

1.  给定的客户端`Socket`实例可以与多少个同时服务器通信？

    一个。客户端套接字与一个主机在一个端口上通信。客户端应用程序可以允许使用多个独立的套接字与多个服务器同时通信，但每个套接字仍然只能与一个服务器通信。

## 代码练习

1.  向我们的游戏协议添加一个功能需要更新服务器和客户端代码。幸运的是，我们可以在两端重复使用`TREE`条目的逻辑。更幸运的是，我们所有的网络通信代码都在`Multiplayer`类中。

    客户端和服务器是内部类，分别被创造性地命名为`Client`和`Server`。对于服务器，在`run()`方法中添加一个循环来在发送树数据后立即发送树篱数据。对于客户端，在`run()`方法中添加一个`HEDGE`段，接受树篱的位置并将其添加到场地上。

    一旦为两名玩家设置了字段，在游戏内部分协议中只报告分数和断开连接。我们不必修改任何此代码。每个玩家都将面对相同的树篱障碍，并有机会用扔苹果来移除它们。

1.  一个人类可读的日期/时间服务器应该相当简单，但是我们希望您练习从头设置自己的套接字。您需要为服务器选择一个端口号。3283 在电话键盘上拼写“DATE”，如果您需要一些灵感。我们建议在接受连接后立即处理客户端请求。（高级练习为您提供了尝试使用线程更复杂方法的机会。）

    对于客户端，唯一的可配置数据是服务器的名称。如果您计划在本地机器上的两个终端窗口上测试您的解决方案，则可以随意硬编码`“localhost”`。我们的解决方案还接受一个可选的命令行参数，默认为`“localhost”`，如果您不提供参数。

## 高级练习

1.  要处理使用线程的客户端，您需要隔离与客户端通信的代码。可以使用帮助类（内部类、匿名类或独立类都可以），或者使用 lambda 表达式。您仍然需要让`ServerSocket`完成其工作并`accept()`新连接，但一旦接收到，您可以立即将接受的`Socket`对象移交给您的帮助类。

    真的很难测试这个类，因为您需要同时有许多客户端在同一时刻请求当前日期。但至少，您的当前`FDClient`类应该可以在不更改的情况下工作，并且您应该仍然收到正确的日期。

1.  使用在线 API 可以很有趣，但也需要注意细节。通常在开始创建客户端时，您需要回答一些问题：

    +   API 的基本 URL 是什么？

    +   API 使用标准的 Web 表单编码或 JSON 吗？如果不是，是否有支持编码和解码的库？

    +   您能够发起多少请求或下载多少数据有限制吗？

    +   网站是否有良好的文档，包含发送和检索数据的常见示例？

随着实践，您将会形成自己对开始使用新 API 所需信息的感觉。但是，您确实需要练习。在构建第一个客户端之后，查找另一个服务。为该 API 编写客户端，并查看您是否已经能够识别常见问题或更好地重用第一个客户端的代码。

¹ 您可以使用`jar tvf <jarfile>`查看任何 JAR 或 ZIP 文件。

² 不过，您可以使用`SortedSet`或`TreeMap`，它们都会保持其条目排序。对于`TreeMap`，键保持有序。
