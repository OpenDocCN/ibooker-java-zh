# 第十三章。平台工具

本章讨论了 Java 平台的 OpenJDK 版本附带的工具。所涵盖的工具都是命令行工具。如果您使用的是其他版本的 Java，您可能会发现在您的分发版本中有不同的工具，但功能类似。

在本章后面，我们将专门为两个工具分配专门的部分：`jshell`，它将交互式开发引入了 Java 平台，以及用于深度分析 Java 应用程序的 Java Flight Recorder（JFR）工具。

# 命令行工具

我们涵盖的命令行工具是最常用的工具和最有用的工具，它们不是每个可用工具的完整描述。特别是与 CORBA 和 RMI 服务器部分有关的工具未涵盖，因为这些模块在 Java 11 发布时从平台中删除了。

###### 注意

在某些情况下，我们需要讨论接受文件系统路径的开关。与本书的其他地方一样，我们在这种情况下使用 Unix 约定。

下面我们将讨论以下工具，包括它们的基本用法、描述和常用开关：

+   `javac`

+   `java`

+   `jar`

+   `javadoc`

+   `jdeps`

+   `jps`

+   `jstat`

+   `jstatd`

+   `jinfo`

+   `jstack`

+   `jmap`

+   `javap`

+   `jlink`

+   `jmod`

+   `jcmd`

###### 注意

整个描述的选项都针对 Java 17，并且在较旧的 Java 版本中可能会有所不同。例如，`--class-path` 是在 `--module-path` 成为选项时引入的，但在 Java 8 及更早版本中不起作用（它们需要 `-cp` 或 `--classpath`）。

javac

###### 基本用法

`bjavac *some*/*package*/MyClass.java`

###### 描述

`javac` 是 Java 源代码编译器——它从 *.java* 源文件生成字节码（以 *.class* 文件的形式）。

对于现代 Java 项目，`javac` 不经常直接使用，因为它相当低级且笨重，特别是对于较大的代码库。相反，现代集成开发环境（IDE）要么自动为开发人员驱动 `javac`，要么在编写代码时具有内置编译器供使用。对于部署，大多数项目将使用单独的构建工具，最常见的是 Maven 或 Gradle。这些工具的讨论超出了本书的范围。

尽管如此，对于开发人员来说，了解如何使用 `javac` 是有用的，因为有些情况下，通过手动编译小型代码库比安装和管理 Maven 等生产级构建工具更可取。

###### 常用开关

`-cp`，`--class-path *<path>*`

为编译提供我们需要的类。

`-p`，`--module-path *<path>*`

为编译提供应用程序模块。请参阅第十二章以了解 Java 模块的全面讨论。

`-d *some*/*dir*`

告诉 `javac` 输出类文件的位置。

`@project.list`

从文件 *project.list* 加载选项和源文件。

`-help`

选项的帮助。

`-X`

非标准选项的帮助。

`-source *<version>*`

控制 `javac` 将接受的 Java 版本。

`-target *<version>*`

控制 `javac` 将输出的类文件的版本。

`-profile *<profile>*`

控制 `javac` 编译应用程序时将使用的配置文件。

`-Xlint`

启用有关警告的详细信息。

`-Xstdout *<path>*`

将编译运行的输出重定向到文件。

`-g`

向类文件添加调试信息。

###### 笔记

`javac` 传统上接受控制编译器接受的源语言版本和用于输出类文件的类文件格式版本的开关（`-source` 和 `-target`）。

此功能引入了额外的编译器复杂性（因为必须内部支持多种语言语法），以换取一些小型开发人员的利益。在 Java 8 中，这种能力稍作整理并放置在更加正式的基础上。

从 JDK 8 开始，`javac` 只接受来自三个版本之前的源和目标选项。也就是说，`javac` 版本 8 只接受 JDK 5、6、7 和 8 的格式。这不影响 `java` 解释器——任何 Java 版本的类文件仍将在 Java 8 附带的 JVM 上正常工作。

C 和 C++ 开发人员可能会发现，与这些其他语言相比，`-g` 开关对他们的帮助较少。这在很大程度上是因为 Java 生态系统中广泛使用的 IDE——集成调试比在类文件中添加额外的调试符号简单得多，也更加有用。

在开发人员中，对于 lint 功能的使用仍然存在一些争议。许多 Java 开发人员生成触发大量编译警告的代码，然后简单地忽略它们。但是，在更大的代码库（特别是 JDK 代码库本身）中的经验表明，在相当大的比例的情况下，触发警告的代码是隐藏着微妙错误的代码。强烈建议使用 lint 功能或静态分析工具（例如 SpotBugs）。

java

###### 基本用法

`java some.package.MyClass`

`java -jar my-packaged.jar`

###### 描述

`java` 是启动 Java 虚拟机的可执行文件。程序的初始入口点是存在于指定类上的 `main()` 方法，其签名为：

```java
public static void main(String[] args);
```

此方法在由 JVM 启动创建的单个应用程序线程上运行。一旦此方法返回（以及任何额外启动的非守护应用程序线程终止），JVM 进程将退出。

如果形式使用 JAR 文件而不是类（可执行的 JAR 形式），则 JAR 文件必须包含一段元数据，告诉 JVM 从哪个类开始启动。

这段元数据是 `Main-Class` 属性，包含在 *META-INF/* 目录中的 *MANIFEST.MF* 文件中。有关更多详细信息，请参阅 `jar` 工具的描述。

###### 常见开关

`-cp`, `--class-path *<path>*`

定义从中读取的类路径。

`-p`, `--module-path *<path>*`

定义查找模块的路径。

`--list-modules`

找到当前设置中的模块列表并退出。

`-X`, `-?`, `-help`

提供关于 `java` 可执行文件及其开关的帮助。

`-D*<property=value>*`

设置一个可以被 Java 程序检索的 Java 系统属性。可以通过这种方式指定任意数量的这种属性。

`-jar`

运行一个可执行的 JAR 文件（参见 `jar` 条目）。

`-Xbootclasspath(/a or /p)`

使用替代的系统类路径运行（极少使用）。

`-client`, `-server`

选择 HotSpot JIT 编译器（参见 “Notes” for this entry）。

`-Xint`, `-Xcomp`, `-Xmixed`

控制 JIT 编译（很少使用）。

`-Xms*<size>*`

设置 JVM 的最小已提交堆大小。

`-Xmx*<size>*`

为 JVM 设置最大的已提交堆大小。

`-agentlib:*<agent>*`, `-agentpath:*<path to agent>*`

指定一个 JVM 工具接口（JVMTI）代理程序附加到正在启动的进程。代理通常用于仪器化或监控。

`-verbose`

生成额外的输出，有时用于调试。

###### Notes

HotSpot VM 包含两个单独的 JIT 编译器——称为客户端（或 C1）编译器和服务器（或 C2）编译器。它们设计用于不同目的，客户端编译器提供更可预测的性能和更快的启动速度，但牺牲了不执行激进代码优化的性能。

传统上，Java 进程使用的 JIT 编译器是通过 `-client` 或 `-server` 开关在进程启动时选择的。然而，随着硬件的进步使得编译成本越来越低，出现了一种新的可能性——在 Java 进程热身时使用客户端编译器，然后在可用时切换到服务器编译器进行高性能优化。这种方案称为分层编译，它是 Java 8 的默认设置。大多数进程将不再需要显式的 `-client` 或 `-server` 开关。

在 Windows 平台上，通常会使用稍微不同版本的 `java` 可执行文件——`javaw`。此版本启动一个 Java 虚拟机，而不会强制出现 Windows 控制台窗口。

在较旧的 Java 版本中，支持多种不同的遗留解释器和虚拟机模式。现在这些大多数已经被移除，任何剩余的应被视为残留的。

以 `-X` 开头的开关原本是非标准的开关。然而，趋势已经向标准化了一些这些开关（特别是 `-Xms` 和 `-Xmx`）。与此同时，Java 版本引入了越来越多的 `-XX:` 开关。这些开关原本是实验性的，不适合生产使用。然而，随着实现的稳定，一些这些开关现在适合一些高级用户（甚至在生产部署中使用）。

总体来说，详细讨论开关超出了本书的范围。为了生产使用配置 JVM 是一个专业的主题，建议开发人员特别注意，尤其是在修改与垃圾收集子系统相关的任何开关时。

jar

###### 基本用法

`jar cvf my.jar *someDir/*`

###### 描述

`jar`实用程序用于创建和操作 Java 存档（*.jar*）文件。这些是包含 Java 类、额外资源和（通常）元数据的 ZIP 格式文件。该工具在一个 JAR 文件上有五种主要的操作模式——创建、更新、索引、列出和提取。

这些由传递给`jar`的命令选项字符（而不是开关）来控制。只能指定一个命令字符，但也可以使用可选的修饰符字符。

###### 命令选项

+   `c`: 创建新存档

+   `u`: 更新存档

+   `i`: 索引存档

+   `t`: 列出一个存档

+   `x`: 提取存档

###### 修饰符

+   `v`: 详细模式

+   `f`: 操作指定的文件，而不是标准输入

+   `0`: 存储但不压缩添加到存档中的文件

+   `m`: 将指定文件的内容添加到`jar`元数据清单中

+   `e`: 使此`jar`可执行，并将指定的类作为入口点

###### 注意

`jar`命令的语法故意与 Unix `tar`命令非常相似。这种相似性是`jar`使用命令选项而不是开关（其他 Java 平台命令所做的）的原因。更典型的显式开关（例如`--create`）也可用，并且可以通过`jar --help`找到它们的文档。

创建 JAR 文件时，`jar`工具将自动添加一个名为*META-INF*的目录，其中包含一个名为*MANIFEST.MF*的文件——这是以头部与值配对的形式的元数据。默认情况下，*MANIFEST.MF*只包含两个头部：

```java
Manifest-Version: 1.0
Created-By: 17.0.4 (Eclipse Adoptium)
```

使用`m`选项允许在 JAR 创建时将附加的元数据添加到*MANIFEST.MF*中。一个经常添加的片段是`Main-Class:`属性，它指示 JAR 中包含的应用程序的入口点。具有指定`Main-Class:`的 JAR 可以通过 JVM 直接执行，通过`java -jar`或在图形文件浏览器中双击 JAR 文件。

添加`Main-Class:`属性是如此常见，以至于`jar`具有`e`选项直接在*MANIFEST.MF*中创建它，而不必为此创建单独的文本文件。可以使用`--extract`选项轻松检查 jar 的内容，包括清单。

javadoc

###### 基本用法

`javadoc *some.package*`

###### 描述

`javadoc`从 Java 源文件生成文档。它通过阅读一种特殊的注释格式（称为 Javadoc 注释）并将其解析成标准文档格式来实现，然后可以将其输出到各种文档格式中（尽管 HTML 是最常见的）。

有关 Javadoc 语法的完整描述，请参阅第七章。

###### 常用开关

`-cp`，`--class-path *<path>*`

定义要使用的类路径。

`-p`，`--module-path *<path>*`

定义要查找模块的路径。

`-D *<directory>*`

告诉`javadoc`生成文档的输出位置。

`-quiet`

除了错误和警告之外，抑制输出。

###### 注意

平台 API 文档都是用 Javadoc 编写的。

`javadoc` 建立在与 `javac` 相同的类之上，并使用一些源编译器基础设施来实现 Javadoc 的特性。

使用 `javadoc` 的典型方式是针对整个包运行，而不仅仅是一个类。

`javadoc` 有很多开关和选项，可以控制其行为的许多方面。详细讨论所有选项超出本书范围。

jdeps

`jdeps` 工具是一个静态分析工具，用于分析包或类的依赖关系。该工具有许多用途，从识别开发者代码调用内部未文档化的 JDK API（如 `sun.misc` 类）到帮助跟踪传递依赖关系。

`jdeps` 还可以用来确认一个 JAR 文件是否能在一个紧凑配置文件下运行（更多关于紧凑配置文件的详细信息请参见本章后面）。

###### 基本用法

`jdeps com.me.MyClass`

###### 描述

`jdeps` 报告请求分析的类的依赖信息。可以指定的类包括类路径上的任何类、文件路径、目录或者 JAR 文件。

###### 常见开关

`-cp`, `--class-path *<path>*`

定义要使用的类路径。

`-p`, `--module-path *<path>*`

定义查找模块的路径。

`-s`, `-summary`

仅打印依赖摘要。

`-m *<module-name>*`

针对一个模块进行分析

`-v`, `-verbose`

打印所有类级别的依赖关系。

`-verbose:package`

打印包级别的依赖关系，排除同一存档内的依赖关系。

`-verbose:class`

打印类级别的依赖关系，排除同一存档内的依赖关系。

`-p *<pkg name>*, -package *<pkg name>*`

在指定的包中查找依赖项。可以多次指定此选项以获取不同的包。`-p` 和 `-e` 选项是互斥的。

`-e *<regex>*, -regex *<regex>*`

查找与指定正则表达式模式匹配的包中的依赖项。`-p` 和 `-e` 选项是互斥的。

`-include *<regex>*`

限制分析到匹配模式的类。此选项过滤要分析的类列表。可以与 `-p` 和 `-e` 一起使用。

`-jdkinternals`

在 JDK 内部 API 中查找类级别的依赖关系（这些 API 可能在即使是次要平台发布中也会更改或消失）。

`-apionly`

限制分析到 API —— 例如，从公共类的签名中的公共和受保护成员的依赖，包括字段类型、方法参数类型、返回类型和已检查的异常类型。

`-R`, `-recursive`

递归遍历所有依赖关系。

`-h`, `-?`, `--help`

打印 `jdeps` 的帮助消息。

###### 注意事项

`jdeps` 是一个有用的工具，可以让开发者意识到他们对 JRE 的依赖不是作为一个单一的环境，而是作为一个更加模块化的东西。

jps

###### 基本用法

`jps`

`jps *<remote URL>*`

###### 描述

`jps`提供了本地机器（或者如果远程端运行了适当的`jstatd`实例，则是远程机器）上所有活动 JVM 进程的列表。远程 URL 支持需要 RMI；这种配置在`jstatd`部分有更详细的解释。

###### 常见开关

`-m`

输出传递给主方法的参数。

`-l`

输出应用程序主类的完整包名称（或应用程序的 JAR 文件的完整路径名称）。

`-v`

输出传递给 JVM 的参数。

###### 笔记

这个命令并不是严格必需的，因为标准的 Unix `ps`命令可能已经足够了。但它不使用标准的 Unix 进程查询机制，因此在某些情况下，Java 进程停止响应（并且在`jps`中看起来已经死掉），但仍然被操作系统列为活动状态。

jstat

###### 基本用法

`jstat -options`

`jstat *<report type such as -class>* *<PID>*`

###### 描述

这个命令显示了关于给定 Java 进程的一些基本统计信息。通常这是一个本地进程，但可以位于远程机器上，只要远程端运行了合适的`jstatd`进程。

###### 常见开关

`-options`

列出`jstat`可以生成的报告类型。最常见的选项包括：

`-class`

报告迄今为止的类加载活动。

`-compiler`

到目前为止的 JIT 编译过程。

`-gcutil`

详细的垃圾回收报告。

`-printcompilation`

更详细的编译信息。

###### 笔记

`jstat`用于识别进程（可能是远程的）的一般语法是：

```java
[*`<``protocol``>`*://]<vmid>[@hostname][:port][/servername]
```

此语法用于指定一个远程进程（通常通过 JMX 通过 RMI 连接），但实际上，更常见的本地语法仅仅使用 VM ID，即主流平台（Linux、Windows、Unix、macOS 等）上的操作系统进程 ID（PID）。

jstatd

###### 基本用法

`jstatd *<options>*`

###### 描述

`jstatd`通过网络公开了本地 JVM 的信息。它使用 RMI 实现，可以使这些本地能力对 JMX 客户端可访问。这需要特殊的安全设置，与 JVM 默认设置不同。要启动`jstatd`，首先需要创建以下文件并将其命名为*jstatd.policy*：

```java
grant codebase "jrt:/jdk.jstatd" {
   permission java.security.AllPermission;
};

grant codebase "jrt:/jdk.internal.jvmstat" {
   permission java.security.AllPermission;
};
```

该策略文件授予加载自实现`jstatd`的 JDK 模块的任何类所有安全权限。引入 JDK 9 中的模块后，精确的策略要求已经改变，并且可能会在未来的 JDK 版本中有所不同。

要使用此策略启动`jstatd`，请使用以下命令行：

```java
jstatd -J-Djava.security.policy=*`<``path` `to` `jstat``.``policy``>`*
```

###### 常见开关

`-p *<port>*`

在该端口上查找现有的 RMI 注册表，如果找不到则创建一个。

###### 笔记

建议在生产环境中始终启用`jstatd`，但不要在公共互联网上使用。对于大多数公司和企业环境来说，这并不容易实现，需要运维和网络工程人员的合作。然而，从生产 JVM 中获取遥测数据的好处，特别是在故障期间，难以言表。

本书不涵盖完整的 JMX 和监控技术讨论。

jinfo

###### 基本用法

`jinfo *<PID>*`

`jinfo *<core file>*`

###### 描述

此工具显示运行中 Java 进程（或核心文件）的系统属性和 JVM 选项。

###### 常见开关

`-flags`

仅显示 JVM 标志。

`-sysprops`

仅显示系统属性。

###### 注意

实际上，这很少被使用，尽管偶尔作为预期程序实际执行的健全性检查可能会有所帮助。

jstack

###### 基本用法

`jstack *<PID>*`

###### 描述

`jstack` 实用程序为进程中的每个 Java 线程生成堆栈跟踪。

###### 常见开关

`-e`

扩展模式（包含关于线程的额外信息）。

`-l`

长模式（包含关于锁的额外信息）。

###### 注意

生成堆栈跟踪不会停止或终止 Java 进程。`jstack` 生成的文件可能非常大，通常需要对文件进行后处理。

jmap

###### 基本用法

`jmap *<output option>* *<process>*`

###### 描述

`jmap` 提供了运行中 Java 进程的内存分配视图。

###### 常见开关

*-dump:<option>,file=<location;>*

生成运行进程的堆转储。

*-histo*

生成当前分配内存状态的直方图。

*-histo:live*

此版本的直方图仅显示活动对象的信息。

###### 注意

直方图形式遍历 JVM 分配列表。这包括活动对象和未收集的（但尚未收集）对象。直方图按使用内存的对象类型组织，并按特定类型使用的字节数量从多到少排序。标准形式不会暂停 JVM。

在执行之前，通过进行完整的停顿式（STW）垃圾回收来确保准确性。因此，在生产系统中，不应在垃圾回收可能显著影响用户的时间使用此工具。

对于 `-dump` 形式，请注意生成堆转储可能是一个耗时的过程，并且是 STW 的。由于当前分配的堆的大小成比例，因此对于某些进程可能非常大。

javap

###### 基本用法

`javap *<classname>*`

`javap *<path/to/ClassFile.class>*`

###### 描述

`javap` 是 Java 类反汇编器，实际上是查看类文件内部的工具。它可以显示 Java 方法编译成的字节码，以及常量池信息（类似于 Unix 进程的符号表）。

默认情况下，`javap` 显示 `public`、`protected` 和默认方法的签名。`-p` 开关还将显示 `private` 方法。

###### 常见开关

`-c`

反编译字节码

`-v`

冗长模式（包括常量池信息）

`-p`

包括 `private` 方法

`-cp`, `--class-path`

类名加载位置

`-p`, `--module-path`

模块加载的位置（如果按类名加载）

###### 注意

`javap` 工具将与任何类文件一起工作，前提是 `javap` 来自于与生成文件相同或更高版本的 JDK。

###### 注意

一些 Java 语言特性在字节码中可能有令人惊讶的实现。例如，正如我们在第九章中所见，Java 的 `String` 类具有有效不可变实例，并且 JVM 在 Java 8 版本后通过不同的方式实现字符串连接运算符 `+`。这种差异在`javap`显示的反汇编字节码中清晰可见。

jlink

###### 基本用法

`jlink *[options]* --module-path modulepath --add-modules module`

###### 描述

`jlink` 是 Java 平台的自定义运行时映像链接器工具，用于将 Java 类、模块及其依赖项打包成自定义运行时映像。`jlink` 工具创建的映像将包括一组链接的模块及其传递的依赖关系。

###### 常见开关

`--add-modules *<module>* [, *module1*]`

将模块添加到要链接的模块的根集合中

`--endian {little|big}`

指定目标体系结构的字节顺序

`--module-path *<path>*`

指定链接模块的路径

`--save-opts *<file>*`

将选项保存到指定文件的链接器中

`--help`

打印帮助信息

`@filename`

从文件名而不是命令行读取选项

###### 注释

`jlink` 工具将与任何类文件或模块一起工作，并且链接将需要代码的传递依赖项被链接。

###### 注意

默认情况下，自定义运行时映像不支持自动更新。这意味着开发人员在必要时需要负责重新构建和更新其应用程序。一些 Java 语言特性可能会有限制，因为运行时映像可能不包含完整的 JDK；因此，反射和其他动态技术可能不完全受支持。

jmod

###### 基本用法

`jmod create *[options]* my-new.jmod`

###### 描述

`jmod` 为自定义链接器（`jlink`）准备 Java 软件组件。结果是一个 *.jmod* 文件。这应被视为中间文件，而不是主要的分发工件。

###### 基本模式

`create`

创建新的 JMOD 文件

`extract`

从 JMOD 文件中提取所有文件（展开它）

`list`

列出来自 JMOD 文件的所有文件

`describe`

打印有关 JMOD 文件的详细信息

###### 常见开关

`--module-path path`

指定模块路径，其中可以找到模块的核心内容。

`--libs path`

指定用于包含的本地库的路径。

`--help`

打印帮助信息。

`@filename`

从文件名而不是命令行读取选项。

###### 注释

`jmod` 读取和写入 JMOD 格式，但请注意，这与模块化 JAR 格式不同，不打算立即替换它。

###### 注意

`jmod`工具目前仅用于要链接到运行时映像（使用`jlink`工具）的模块。另一个可能的用例是打包具有必须随模块一起分发的本地库或其他配置文件的模块。

jcmd

###### 基本用法

`jcmd *<PID>*`

`jcmd *<PID>* *<command>*`

###### 描述

`jcmd`向正在运行的 Java 进程发出命令。精确的命令可能因 Java 版本而异，并且可以通过使用进程 ID 和无命令运行`jcmd`来列出。

###### 常见开关

`-f *<path>*`

从文件而不是命令行参数读取命令

`-l`

列出 Java 进程（类似于`jps`）

`--help`

打印帮助信息

###### 常见命令

`GC.heap_dump *<path>*`

生成类似于`jmap`的堆转储。注意路径是相对于 Java 进程的，*而不是* `jcmd` 运行的位置！

`GC.heap_info`

显示有关 Java 进程堆的统计和大小信息。

`JFR.start`

开始 Java Flight Recorder（JFR）会话。JFR 是 JVM 内置的性能监控和分析工具。

`JFR.stop name=*<name from start>* filename=*<path>*`

停止命名的 JFR 会话并记录到文件中。

`VM.system_properties`

输出 Java 进程的系统属性。

###### 注意事项

###### 注意

`jcmd`的命令按其与之交互的子系统分组，例如`GC`或`JFR`。除了我们提供的示例之外，还有许多其他命令。值得探索您的 Java 安装中可用的内容，以帮助在生产环境中操作 JVM。

# JShell 介绍

Java 传统上被理解为一种类导向的语言，并具有明确的编译-解释-评估执行模型。然而，在本节中，我们将讨论一种通过提供一种交互/脚本能力来扩展此编程范例的新技术。

随着 Java 9 的到来，Java 运行时和 JDK 捆绑了一个新工具，JShell。这是一个用于 Java 的交互式 Shell，类似于 Python、Scala 或 Lisp 中的 REPL。该 Shell 旨在用于教学和探索使用，并且由于 Java 语言的性质，不像其他语言中类似的 Shell 对于工作程序员来说那么有用。

特别是，预计 Java 不会成为一个以 REPL 驱动的语言。相反，这打开了一个机会，可以使用 JShell 进行一种不同风格的编程，这种风格既补充了传统用例，又提供了新的视角，尤其是用于使用新 API 的情况。

使用 JShell 非常容易探索简单的语言特性，例如：

+   原始数据类型

+   简单的数值操作

+   字符串操作基础

+   对象类型

+   定义新类

+   创建新对象

+   调用方法

要启动 JShell，我们只需从命令行调用它：

```java
$ jshell
|  Welcome to JShell -- Version 17.0.4
|  For an introduction type: /help intro

jshell>
```

从这里，我们可以输入小段的 Java 代码，称为*片段*：

```java
jshell> 2 * 3
$1 ==> 6

jshell> var i = 2 * 3
i ==> 6
```

Shell 被设计为一个简单的工作环境，因此放宽了一些工作中的 Java 程序员可能期望的规则。JShell 代码片段与常规 Java 之间的一些差异包括：

+   在 JShell 中分号是可选的

+   JShell 支持详细模式

+   JShell 具有比常规 Java 程序更广泛的默认导入集合

+   方法可以在顶层声明（不在类内部）

+   方法可以在代码片段内重新定义

+   代码片段不能声明包或模块——所有内容都放在由 shell 控制的无名称包中

+   只有公共类可以从 JShell 访问

+   由于包限制，定义类和在 JShell 中工作时忽略访问控制是明智的选择

创建简单的类层次结构很简单（例如，用于探索 Java 的继承和泛型）：

```java
jshell> class Pet {}
|  created class Pet

jshell> class Cat extends Pet {}
|  created class Cat

jshell> var c = new Cat()
c ==> Cat@2ac273d3
```

在 shell 内进行的 tab 自动补全也是可能的，例如用于可能方法的自动补全：

```java
jshell> c.<TAB>
equals(       getClass()    hashCode()    notify()      notifyAll()
toString()    wait(
```

按下 tab 键两次并输入某些内容将显示方法的文档：

```java
jshell> c.hashCode(<TAB>
Signatures:
int Object.hashCode()

<press tab again to see documentation>
jshell> c.hashCode(<TAB TAB>
int Object.hashCode()
Returns a hash code value for the object. (Full Javadoc follows...)
```

我们还可以创建顶层方法，比如：

```java
jshell> int div(int x, int y) {
   ...> return x / y;
   ...> }
|  created method div(int,int)
```

也支持简单的异常回溯：

```java
jshell> div(3,0)
|  Exception java.lang.ArithmeticException: / by zero
|        at div (#2:2)
|        at (#3:1)
```

我们可以访问来自 JDK 的类：

```java
jshell> var ls = List.of("Alpha", "Beta", "Gamma", "Delta", "Epsilon")
ls ==> [Alpha, Beta, Gamma, Delta, Epsilon]

jshell> ls.get(3)
$11 ==> "Delta"

jshell> ls.forEach(s -> System.out.println(s.charAt(1)))
l
e
a
e
p
```

或者如有必要，显式导入类：

```java
jshell> import java.time.LocalDateTime

jshell> var now = LocalDateTime.now()
now ==> 2018-10-02T14:48:28.139422

jshell> now.plusWeeks(3)
$9 ==> 2018-10-23T14:48:28.139422
```

环境还允许以`/`开头的 JShell 命令。熟悉一些最常见的基本命令非常有用：

+   `/help intro` 是介绍性的帮助文本

+   `/help` 是进入帮助系统的更全面入口

+   `/vars` 显示当前作用域中的变量

+   `/list` 显示 shell 历史记录

+   `/save` 将接受的代码片段源输出到文件

+   `/open` 读取保存的文件并将其引入环境

+   `/exit` 退出 jshell 界面

例如，在 JShell 中可用的导入不仅限于`java.lang`。整个列表在 JShell 启动时由 JShell 加载，可以通过`/list -all`命令看到*特殊导入*：

```java
jshell> /list -all

  s1 : import java.io.*;
  s2 : import java.math.*;
  s3 : import java.net.*;
  s4 : import java.nio.file.*;
  s5 : import java.util.*;
  s6 : import java.util.concurrent.*;
  s7 : import java.util.function.*;
  s8 : import java.util.prefs.*;
  s9 : import java.util.regex.*;
 s10 : import java.util.stream.*;
```

JShell 环境支持 tab 自动补全，这极大地增加了工具的可用性。详细模式特别适用于了解 JShell 时——它可以通过在启动时传递`-v`开关或通过 shell 命令激活。

# Java Flight Recorder (JFR) 简介

Java Flight Record (JFR) 是一个强大的、低延迟的分析系统，直接内嵌在 JVM 中。在 Java 11 之前，它存在多年，但只能通过商业许可证获得。现在，这个丰富的信息源已经可以通过 OpenJDK 访问，值得探索。

典型的 JFR 工作流程涉及对运行中的 JVM 启动分析，将结果下载为文件，然后通过 JDK Mission Control (JMC) GUI 应用程序离线检查该文件。虽然 JFR 直接嵌入在 OpenJDK 中，但 JMC 并不随 JDK 分发，可以从[*https://oreil.ly/eq4cg*](https://oreil.ly/eq4cg)下载。

JFR（Java Flight Recorder）的录制可以通过 JVM 启动时的选项或本章前面展示的`jcmd`工具进行交互式启动。以下`java`调用会在两分钟内开始 JFR 录制，并在完成后将结果写入文件：

```java
java -XX:StartFlightRecording=duration=120s,filename=flight.jfr \
   Application
```

选项允许紧密控制 JFR 在内存中保存的数据量，可以通过指定生成录制的持续时间或生成文件的大小来实现。结合其低开销，可以合理地在生产环境中持续运行 JFR，以便随时捕获数据（有时称为“环形缓冲”模式）。这为使用 JFR 分析在问题发生几分钟到几小时后的详细调试开启了可能性。

除了大小限制外，JFR 录制还可以配置为仅收集感兴趣的特定信息。典型领域（但只是 JFR 测量的一小部分）包括：

+   对象分配

+   垃圾收集

+   线程和锁

+   方法分析

随着 Java 17，API 已经提供在进程中以流式方式消费 JFR 事件，逐步摆脱基于文件的分析方法。这为监控工具无需登录服务器请求转储性能分析数据开辟了新的可能性。

未来，我们可以期待 JFR 作为 Java 生态系统采用的新一代可观察性工具的数据源。

# 概要

在过去的 15 年中，Java 发生了巨大变化，然而平台和社区仍然充满活力。能够在保留可识别的语言和平台的同时取得这一成就，绝非小事。

最终，Java 的持续存在和可行性取决于个别开发者。基于这一点，未来看起来光明，我们期待着下一个浪潮及其之后的发展。
