# 第二章\. 与环境交互

# 2.0 引言

本章描述了你的 Java 程序如何处理其即时环境，即我们所称的运行时环境。在某种意义上，几乎使用任何 Java API 在 Java 程序中进行的所有操作都涉及环境。在这里，我们更专注于直接围绕你的程序的事物。在这个过程中，我们将介绍 `System` 类，它对你的特定系统了解甚深。

另外两个运行时类值得简要提及。第一个是 `java.lang.Runtime`，它在 `System` 类的许多方法背后起作用。例如，`System.exit()` 只是调用 `Runtime.exit()`。`Runtime` 在技术上属于环境的一部分，但我们直接使用它的唯一时机是运行其他程序，这在 Recipe 18.1 中有涵盖。

# 2.1 获取环境变量

## 问题

你希望从你的 Java 程序内部获取环境变量的值。

## 解决方案

使用 `System.getenv()`。

## 讨论

1979 年发布的 Unix 第七版引入了一个称为环境变量的新功能。环境变量在所有现代 Unix 系统（包括 macOS）和大多数后来的命令行系统（如 Windows 中的 DOS 或命令提示符）中都存在，但在一些旧平台或其他 Java 运行时中并不存在。环境变量通常用于自定义个人计算机用户的运行时环境，因此得名。举一个熟悉的例子，在 Unix 或 DOS 上，环境变量 `PATH` 决定系统查找可执行程序的位置。因此，人们想知道如何从他们的 Java 程序中访问环境变量。

答案是，在所有现代版本的 Java 中都可以这样做，但是应该谨慎依赖能够指定环境变量，因为一些罕见的操作系统可能不提供它们。尽管如此，你不太可能遇到这样的系统，因为所有“标准”桌面系统目前都提供它们。

在一些古老版本的 Java 中，`System.getenv()` 被弃用或者根本无效。如今，`getenv()` 方法不再被弃用，尽管仍然警告应该使用系统属性（参见 Recipe 2.2）。即使在支持环境变量的系统中，它们的名称在某些平台上是大小写敏感的，在其他平台上则是不敏感的。Example 2-1 中的代码是一个使用 `getenv()` 方法的简短程序。

##### 示例 2-1\. main/src/main/java/environ/GetEnv.java

```java
public class GetEnv {
    public static void main(String[] argv) {
        System.out.println("System.getenv(\"PATH\") = " + System.getenv("PATH"));
    }
}
```

运行此代码将产生类似以下的输出：

```java
C:\javasrc>java environ.GetEnv
System.getenv("PATH") = C:\windows\bin;c:\jdk1.8\bin;c:\documents
    and settings\ian\bin
C:\javasrc>
```

`System.getenv()` 方法的无参数形式以不可变的 `String Map` 形式返回*所有*环境变量。你可以遍历这个映射并访问所有用户的设置或检索多个环境设置。

`getenv()` 的两种形式都要求您具有访问环境的权限，因此它们通常不适用于受限制的环境，例如小程序。

# 2.2 从系统属性中获取信息

## 问题

您需要从系统属性中获取信息。

## 解决方案

使用 `System.getProperty()` 或 `System.getProperties()`。

## 讨论

什么是 *属性* 呢？属性只是存储在 `java.util.Properties` 对象中的名称和值对，我们在 Recipe 7.10 中会更详细地讨论它。

`System.Properties` 对象控制并描述了 Java 运行时。`System` 类有一个静态的 `Properties` 成员，其内容是操作系统特定部分（例如 `os.name`）、系统和用户定制部分（`java.class.path`）以及在命令行上定义的属性（我们稍后会看到）。请注意，这些名称中的句点（例如 `os.arch`、`os.version`、`java.class.path` 和 `java.lang.version`）使其看起来好像存在类似于包/类名称的层次关系。然而，`Properties` 类并不强制这样的关系：每个键只是一个字符串，句点并不特殊。

要查看所有定义的系统属性，可以通过调用 `System.getProperties()` 的输出进行迭代，就像在 Example 2-2 中一样。

##### 例子 2-2\. jshell System.getProperties()

```java
jshell> System.getProperties().forEach((k,v) -> System.out.println(k + "->" +v))
awt.toolkit->sun.awt.X11.XToolkit
java.specification.version->11
sun.cpu.isalist->
sun.jnu.encoding->UTF-8
java.class.path->.
java.vm.vendor->Oracle Corporation
sun.arch.data.model->64
java.vendor.url->http://java.oracle.com/
user.timezone->
os.name->OpenBSD
java.vm.specification.version->11
... many more ...
jshell>
```

请记住，以“sun”开头的属性是不受支持且可能会更改的。

要检索一个系统提供的属性，使用 `System.getProperty(propName)`。如果我只想知道 `System Properties` 是否有一个名为 `"pencil_color"` 的属性，我可以这样说：

```java
        String sysColor = System.getProperty("pencil_color");
```

那么它返回什么呢？当然，Java 并不聪明到知道每个人最喜欢的铅笔颜色吧？你是对的！但是，我们可以使用 `-D` 参数轻松地告诉 Java 关于我们的铅笔颜色（或任何我们想告诉它的东西）。

当启动 Java 运行时，可以使用 `-D` 选项在系统属性对象中定义一个值。它的参数必须包含名称、等号和值，这与属性文件中的解析方式相同（参见 Recipe 7.10）。在 `java` 命令和命令行上的类名之间可以有多个 `-D` 定义。在 Unix 或 Windows 命令行中，输入：

```java
java -D"pencil_color=Deep Sea Green" environ.SysPropDemo
```

在 IDE 下运行时，在适当的对话框中（例如，在 Eclipse 的运行配置对话框中的程序参数下），将变量的名称和值放入其中。您还可以使用构建工具（Maven、Gradle 等）设置环境变量和系统属性。

`SysPropDemo` 程序中的代码用于提取一个或多个属性，您可以像这样运行它：

```java
$ java environ.SysPropDemo os.arch
os.arch = x86
```

如果不带参数调用 `SysPropDemo` 程序，则输出与 Example 2-2 中的 `jshell` 片段相同的信息。

这提醒我——现在是提及依赖系统的代码的好时机。配方 2.3 讨论了依赖操作系统和发布依赖代码。

## 另请参阅

配方 7.10 列出了关于使用和命名自己的`Properties`文件的更多细节。`java.util.Properties`的 javadoc 页面列出了`load()`方法使用的确切规则，以及其他细节。

# 2.3 处理依赖于 Java 版本或操作系统的代码

## 问题

您需要编写适应底层操作系统的代码。

## 解决方案

您可以使用`System.Properties`来查找 Java 版本和操作系统，使用`File`类中的各种功能来查找一些平台相关的特性，以及`java.awt.TaskBar`来查看是否可以使用系统相关的任务栏或 Dock。

## 讨论

有些东西取决于您正在运行的 Java 版本。使用`System.getProperty()`参数为`java.specification.version`。

或者，更普遍地说，您可能想测试特定类的存在与否。一种方法是使用`Class.forName("class")`（参见第十七章），如果无法加载类，则会抛出异常——这表明它在运行时库中不存在。示例 2-3 展示了这种情况下的代码，来自希望查找常见 Swing UI 组件是否可用的应用程序。标准类的 javadoc 报告了此类首次出现的 JDK 版本，标题为“Since”。如果没有此标题，通常意味着该类从 Java 开始就存在：

##### 示例 2-3\. *main/src/main/java/starting/CheckForSwing.java*

```java
public class CheckForSwing {
    public static void main(String[] args) {
        try {
            Class.forName("javax.swing.JButton");
        } catch (ClassNotFoundException e) {
            String failure =
                "Sorry, but this version of MyApp needs \n" +
                "a Java Runtime with JFC/Swing components\n" +
                "having the final names (javax.swing.*)";
            // Better to make something appear in the GUI. Either a
            // JOptionPane, or: myPanel.add(new Label(failure));
            System.err.println(failure);
        }
        // No need to print anything here - the GUI should work...
    }
}
```

重要的是要区分在编译时和运行时测试此代码。在两种情况下，必须在包含您要测试的类的系统上编译它：JDK >= 1.1 和 Swing，分别。这些测试只是试图帮助那些试图运行您更新的应用程序的 Java 运行时用户。目标是为这些用户提供比运行时简单的“类未找到”错误更有意义的消息。还要注意，如果将此测试编写在依赖于您正在测试的代码的任何代码内部，该测试将变得无法到达。在应用程序的主流程中尽早放置测试，在构造任何 GUI 对象之前。否则，该代码只会浪费在更新的运行时上的空间，而在不包含 Swing 的 Java 系统上永远不会运行。显然，这只是一个非常早期的示例，但您可以使用相同的技术测试 Java 演变的任何阶段添加的任何运行时特性（请参阅附录 A 了解 Java 每个发布版本中添加的功能概述）。您还可以使用此技术确定是否已成功将所需的第三方库添加到您的 CLASSPATH 中。

此外，尽管 Java 旨在实现可移植性，但有些情况并非如此。其中包括文件名分隔符等变量。Unix 上的每个人都知道文件名分隔符是斜杠字符（/），反斜杠或反斜杠（\）是转义字符。回到 20 世纪 70 年代末，Microsoft 的一个小组实际上正在研究 Unix——他们的版本被称为 Xenix，后来被 SCO 接管——并且 DOS 上的开发人员看到并喜欢了 Unix 的文件系统模型。最早期的 MS-DOS 没有目录；它只有像 Digital Research CP/M（它本身是各种其他系统的克隆）那样的用户编号。因此，Microsoft 的开发人员着手克隆 Unix 的文件系统组织。不幸的是，MS-DOS 已经将斜杠字符用于用作选项分隔符，而 Unix 则使用破折号（`-`）；并且`PATH`分隔符（:)也用作驱动器号分隔符，如`C`:或`A`:。因此，我们现在有了像在表 2-1 中所示的命令。

表 2-1\. 目录列表命令

| 系统 | 目录列表命令 | 含义 | 示例 PATH 设置 |
| --- | --- | --- | --- |
| Unix | *ls -R /* | /的递归列表，顶级目录 | *PATH=/bin:/usr/bin* |
| DOS | *dir/s \* | 目录及其子目录选项（即递归）的顶级目录（但仅限当前驱动器） | *PATH=C:\windows;D:\mybin* |

这对我们有何帮助？如果我们要在 Java 中生成文件名，我们可能需要知道是放置/还是\或其他字符。Java 有两种解决方案。首先，在移动到 Unix 和 Microsoft 系统之间时，至少是*宽容的*：可以使用/或\，^(1)，处理操作系统的代码会解决这个问题。其次，更普遍地，Java 以与平台无关的方式提供平台特定的信息。对于文件分隔符（以及`PATH`分隔符），`java.io.File`类提供了一些包含此信息的静态变量。由于`File`类管理着与平台相关的信息，因此将此信息锚定在这里是有意义的。变量在表 2-2 中显示。

表 2-2\. 文件属性表

| 名称 | 类型 | 含义 |
| --- | --- | --- |
| `separator` | `static String` | 系统相关的文件名分隔符字符（例如，/或\） |
| `separatorChar` | `static char` | 系统相关的文件名分隔符字符（例如，/或\） |
| `pathSeparator` | `static String` | 系统相关的路径分隔符字符，以字符串形式表示以方便使用 |
| `pathSeparatorChar` | `static char` | 系统相关的路径分隔符字符 |

文件名和路径分隔符通常是字符，但也以`String`形式提供以方便使用。

第二个更一般的机制是提到的`System Properties`对象，它在 Recipe 2.2 中提到。您可以使用它来确定您正在运行的操作系统。以下是一个简单列出系统属性的代码；在几个不同的实现上运行这个代码可能很有启发性：

```java
public class SysPropDemo {
    public static void main(String[] argv) throws IOException {
        if (argv.length == 0)
            // tag::sysprops[]
            System.getProperties().list(System.out);
            // end::sysprops[]
        else {
            for (String s : argv) {
                System.out.println(s + " = " +
                    System.getProperty(s));
            }
        }
    }
}
```

例如，某些操作系统提供了称为空设备的机制，可用于丢弃输出（通常用于计时目的）。以下是请求系统属性`os.name`的代码，并使用它来编制可以用于丢弃数据的名称（如果给定平台没有已知的空设备，我们返回名称`jnk`，这意味着在这些平台上，我们偶尔会创建，嗯，垃圾文件；我在偶然发现时删除这些文件）：

```java
package com.darwinsys.lang;

import java.io.File;

/** Some things that are system-dependent.
 * All methods are static.
 * @author Ian Darwin
 */
public class SysDep {

    final static String UNIX_NULL_DEV = "/dev/null";
    final static String WINDOWS_NULL_DEV = "NUL:";
    final static String FAKE_NULL_DEV = "jnk";

    /** Return the name of the null device on platforms which support it,
     * or "jnk" (to create an obviously well-named temp file) otherwise.
     * @return The name to use for output.
     */
    public static String getDevNull() {

        if (new File(UNIX_NULL_DEV).exists()) {     ![1](img/1.png)
            return UNIX_NULL_DEV;
        }

        String sys = System.getProperty("os.name"); ![2](img/2.png)
        if (sys==null) {                            ![3](img/3.png)
            return FAKE_NULL_DEV;
        }
        if (sys.startsWith("Windows")) {            ![4](img/4.png)
            return WINDOWS_NULL_DEV;
        }
        return FAKE_NULL_DEV;                       ![5](img/5.png)
    }
}
```

![1](img/#co_interacting_with_the_environment_CO1-1)

如果`/dev/null`存在，请使用它。

![2](img/#co_interacting_with_the_environment_CO1-2)

如果没有，请询问`System properties`是否知道操作系统名称。

![3](img/#co_interacting_with_the_environment_CO1-3)

不行，所以放弃，返回`jnk`。

![4](img/#co_interacting_with_the_environment_CO1-4)

我们知道这是 Microsoft Windows，所以使用`NUL:`。

![5](img/#co_interacting_with_the_environment_CO1-5)

除非所有其他方法都失败，否则使用`jnk`。

尽管 Java 的 Swing GUI 旨在具有可移植性，但苹果在 macOS 上的实现并不会自动为每个人都正确处理。例如，`JMenuBar`菜单容器默认出现在应用程序窗口的顶部。这在 Windows 和大多数 Unix 平台上是正常的，但 Mac 用户希望活动应用程序的菜单栏出现在屏幕顶部。为了启用正常行为，您必须在 Swing GUI 启动之前将`System`属性`apple.laf.useScreenMenuBar`设置为`true`。您可能还想设置其他一些属性，例如在菜单栏中显示应用程序的短名称（默认为主应用程序类的完整类名）。

书中源代码的示例，在*src/main/java/gui/MacOsUiHints.java*中可以找到。

设置这些属性可能没有意义，除非实际上是在 macOS 下运行。如何判断？苹果推荐的方法是检查系统属性`mrj.runtime`，如果是，则假定您正在使用 macOS：

```java
boolean isMacOS = System.getProperty("mrj.version") != null;
if (isMacOS) {
  System.setProperty("apple.laf.useScreenMenuBar",  "true");
  System.setProperty("com.apple.mrj.application.apple.menu.about.name",
  "My Super App");
}
```

另一方面，在非 Mac 系统上这些属性可能无害，因此您可以跳过测试并无条件设置这两个属性。

最后，Mac 的 Dock 或大多数其他系统上的任务栏可以使用在 Java 9 中添加的`java.awt.Taskbar`类访问。这里没有讨论，但在`main/gui`子目录中有一个名为`TaskbarDemo`的示例。

# 2.4 使用扩展或其他打包的 API

## 问题

你有一个你想使用的类的 JAR 文件。

## 解决方案

只需将 JAR 文件添加到您的`CLASSPATH`中。

## 讨论

随着您构建更复杂的应用程序，您将需要使用越来越多的第三方库。您可以将它们添加到您的 `CLASSPATH` 中。

以前建议将这些 JAR 文件放入 Java 扩展机制目录，通常类似于 *\jdk1.x\jre\lib\ext*，而不是在 `CLASSPATH` 变量中列出每个 JAR 文件。但是，现在一般不再推荐这样做，最新的 JDK 中也不再支持。相反，您可能希望使用像 Maven（参见 Recipe 1.7）或 Gradle 这样的构建工具，以及集成开发环境（IDE），来自动添加 JAR 文件到您的 `CLASSPATH` 中。

我不喜欢使用扩展目录的一个原因是，它需要修改已安装的 JDK 或 JRE，这可能会导致维护问题，以及在安装新 JDK 或 JRE 时出现问题。

Java 9 引入了对 Java 的重大改变，即 Java 9 模块系统用于程序模块化，我们在 Recipe 2.5 中进行了讨论。

# 2.5 使用 Java 模块系统

## 问题

您正在使用 Java 9 或更高版本，并且需要处理模块机制。

## 解决方案

继续阅读。

## 讨论

Java 模块系统，以前称为项目 Jigsaw，旨在解决构建大型应用程序所需的许多小部分的问题。在某种程度上，像 Maven 和 Gradle 这样的工具已经解决了这个问题，但模块系统解决的问题与这些工具略有不同。Maven 或 Gradle 将找到依赖项，下载它们，在您的开发和测试运行时安装它们，并将它们打包成可运行的 JAR 文件。模块系统更关注于从一个应用程序代码块到另一个代码块的类的可见性，通常是由不同的开发人员提供，他们可能不了解或信任彼此。因此，这表明 Java 最初的访问修饰符集（如 `public`、`private`、`protected` 和默认可见性）对于构建大型应用程序是不够的。

接下来简要讨论使用 JPMS，即 Java 平台模块系统，将模块导入到您的应用程序中。在 第十五章 中介绍了创建自己的模块。如需更详细的介绍，您可以参考像 *Java 9 Modularity: Patterns and Practices for Developing Maintainable Applications* 这样的专著，作者是 Sander Mak 和 Paul Bakker（O’Reilly）。

Java 一直是大规模开发的语言。面向对象是其中的关键之一：类和对象组合方法，访问修饰符可以应用，使得公共和私有方法清晰分离。在开发大型应用程序时，仅具有类的单一扁平命名空间仍然不够。进入包：它们将类收集到其自己的命名空间内的逻辑组中。同样可以在包级别应用访问控制，以便某些类仅在包内可访问。模块是更高一级的下一步。模块将一些相关的包组合在一起，具有明确的名称，并且可以限制对某些包的访问，同时将其他包作为公共 API 暴露给不同的模块。

一开始要理解的一件事：JPMS 并不是你现有构建工具的替代品。无论你使用 Maven、Gradle、Ant，还是只是将所有需要的 JAR 文件倒入 lib 目录，你仍然需要这样做。同时，不要将 Maven 的模块与 JPMS 模块混淆；前者是将项目物理结构化为子项目，后者是 Java 平台（编译器、运行时）理解的内容。通常在使用 Java 模块时，每个 Java 模块将等同于一个单独的 Maven 模块。

当你处理一个小而自包含的程序时，你不需要担心模块。只需在编译时和运行时将所有必要的 JAR 文件放在你的`CLASSPATH`上，一切都会很好。可能。

在此过程中，你可能会看到类似以下的警告消息：

```java
Illegal reflective access by com.foo.Bar
    (file:/Users/ian/.m2/repository/com/foo/1.3.1/foo-1.3.1.jar)
    to field java.util.Properties.defaults
Please consider reporting this to the maintainers of com.foo.Bar
Use --illegal-access=warn to enable warnings of further
 illegal reflective access operations
All illegal access operations will be denied in a future release
```

警告消息是由于 JPMS 在执行其作业时产生的，检查在模块内封装的包中是否访问了任何类型。随着所有公共 Java 库和正在开发的所有应用程序逐步模块化，此类消息将逐渐消失。

为什么仅“可能”一切都会很好？如果你使用了在过去几个版本中已被弃用的某些类，事情将无法编译。为此，你必须使相应的模块可用。在`javasrc`下的*unsafe*子目录（也是一个 Maven 模块）中，有一个名为`LoadAverage`的类。负载平均值是 Unix/Linux 系统的一个特性，通过报告等待运行的进程数，给出了系统负载或繁忙程度的粗略测量。几乎总是有更多的进程在运行，而 CPU 核心数却要少于它们，因此总有一些进程需要等待。更高的数字意味着一个更繁忙的系统，响应速度较慢。

Sun 不支持的`Unsafe`类具有一个方法来获取负载平均值，仅在支持它的系统上。代码必须使用反射 API（参见第十七章）来获取`Unsafe`对象；如果尝试直接实例化`Unsafe`，将会得到`SecurityException`（这在模块系统之前是这样）。一旦获取了实例并将其转换为`Unsafe`，就可以调用诸如`loadAverage()`（示例 2-4）之类的方法。

##### 示例 2-4\. unsafe/src/main/java/unsafe/LoadAverage.java（使用 Unsafe.java）

```java
public class LoadAverage {
    public static void main(String[] args) throws Exception {
        Field f = Unsafe.class.getDeclaredField("theUnsafe");
        f.setAccessible(true);
        Unsafe unsafe = (Unsafe) f.get(null);
        int nelem = 3;
        double loadAvg[] = new double[nelem];
        unsafe.getLoadAverage(loadAvg, nelem);
        for (double d : loadAvg) {
            System.out.printf("%4.2f ", d);
        }
        System.out.println();
    }
}
```

此代码曾经编译通过，现在却产生了警告。如果我们正在使用 Java 模块，则必须修改我们的*module-info.java*文件，告诉编译器和 VM 我们需要使用半明显命名的模块`jdk.unsupported`。

```java
module javasrc.unsafe {
    requires jdk.unsupported;
	// others...
}
```

我们将在 Recipe 15.9 中详细介绍模块文件格式。

现在我们已经编写了代码并将模块文件放在了源文件夹的顶层，我们可以构建项目、运行程序，并将其输出与系统级工具*uptime*显示的负载平均值进行比较，例如，`uptime`。尽管我们仍然会收到“内部专有 API”警告，但它确实有效：

```java
$ java -version
openjdk version "14-ea" 2020-03-17
OpenJDK Runtime Environment (build 14-ea+27-1339)
OpenJDK 64-Bit Server VM (build 14-ea+27-1339, mixed mode, sharing)
$ mvn clean package
[INFO] Scanning for projects...
[INFO]
[INFO] --------------------< com.darwinsys:javasrc-unsafe >--------------------
[INFO] Building javasrc - Unsafe 1.0.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ javasrc-unsafe ---
[INFO] Deleting /Users/ian/workspace/javasrc/unsafe/target
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ javasrc-unsafe ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /Users/ian/workspace/javasrc/unsafe/src/main/resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ javasrc-unsafe ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 2 source files to /Users/ian/workspace/javasrc/unsafe/target/classes
[WARNING] /Users/ian/workspace/javasrc/unsafe/src/main/java/unsafe/LoadAverage.java:[3,16] sun.misc.Unsafe is internal proprietary API and may be removed in a future release
[WARNING] /Users/ian/workspace/javasrc/unsafe/src/main/java/unsafe/LoadAverage.java:[12,27] sun.misc.Unsafe is internal proprietary API and may be removed in a future release
[WARNING] /Users/ian/workspace/javasrc/unsafe/src/main/java/unsafe/LoadAverage.java:[14,17] sun.misc.Unsafe is internal proprietary API and may be removed in a future release
[WARNING] /Users/ian/workspace/javasrc/unsafe/src/main/java/unsafe/LoadAverage.java:[14,34] sun.misc.Unsafe is internal proprietary API and may be removed in a future release
[INFO]
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ javasrc-unsafe ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /Users/ian/workspace/javasrc/unsafe/src/test/resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ javasrc-unsafe ---
[INFO] No sources to compile
[INFO]
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ javasrc-unsafe ---
[INFO] No tests to run.
[INFO]
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ javasrc-unsafe ---
[INFO] Building jar: /Users/ian/workspace/javasrc/unsafe/target/javasrc-unsafe-1.0.0-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  4.668 s
[INFO] Finished at: 2020-01-05T14:53:55-05:00
[INFO] ------------------------------------------------------------------------
$
$ java -cp target/classes unsafe/LoadAverage
3.54 1.94 1.62
$ uptime
14:54  up 1 day, 21:50, 5 users, load averages: 3.54 1.94 1.62
$
```

幸运的是，它确实有效，并且与标准 Unix *uptime*命令给出相同的数字。至少在 Java 11 上有效。正如警告所示，它*可能*（即很可能）会在以后的版本中删除。

如果您正在构建一个更复杂的应用程序，您可能需要编写一个更完整的*module-info.java*文件。但在这个阶段，主要是需要的模块。标准 Java API 分为几个模块，您可以使用*java*命令列出它们：

```java
$ java --list-modules
java.base
java.compiler
java.datatransfer
java.desktop
java.instrument
java.logging
java.management
java.management.rmi
java.naming
java.net.http
java.prefs
java.rmi
java.scripting
java.se
java.security.jgss
java.security.sasl
java.smartcardio
java.sql
java.sql.rowset
java.transaction.xa
java.xml
java.xml.crypto
... plus a bunch of JDK modules ...
```

其中，`java.base`始终可用且无需在模块文件中列出，`java.desktop`添加了用于图形的 AWT 和 Swing，而`java.se`则包含了 Java SDK 中以前的几乎所有公共 API。例如，如果我们的负载平均程序想要在 Swing 窗口中显示结果，就需要将其添加到模块文件中：

```java
requires java.desktop;
```

当您的应用程序足够大，可以划分为多个层次或层次时，您可能希望使用 JPMS 描述这些模块。由于这个主题属于打包的范畴，它在 Recipe 15.9 中有描述。

^(1) 在为在 Windows 上使用的字符串编译时，请记住在大多数地方（除了 MS-DOS 命令行）\ 是转义字符：`String rootDir = "C:\\";`.
