# 前言

像任何使用最多的编程语言一样，Java 有其反对者、支持者、问题、怪癖¹ 和一个学习曲线。*Java Cookbook* 的目标是帮助 Java 开发者迅速掌握 Java 开发的一些最重要的部分。我专注于标准 API 和一些第三方 API，但也不吝涉及语言问题。

这是本书的第四版，经过许多人的共同努力以及 Java 这两个十年中经历的无数变化而形成。对 Java 历史感兴趣的读者可以参考 附录 A。

Java 11 是当前的长期支持版本，但 Java 12 和 13 已经发布。Java 14 正在提前访问，并计划在本书第四版发布当天最终发布。每六个月发布一次的新发布节奏可能对 Oracle 的 Java SE 开发团队和以 Java 为关键词的新闻网站来说是个好消息，但对于 Java 书籍作者来说，“可能会增加一些额外的工作”，因为书籍的修订周期通常比 Java 现在的发布周期长！Java 9 是本书上一版之后发布的，是一个破坏性的发布，是很长时间以来第一个破坏向后兼容性的发布，主要是 Java 模块系统。本书中的所有内容都假定在任何仍在使用的 JVM 上都可以工作。任何人都不应该再使用 Java 7（或之前的版本！）做任何事情，并且任何人都不应该在 Java 8 上进行新的开发。如果你还在使用，现在是时候转变了！

此次修订的目标是保持本书与所有这些变化的同步。虽然剔除了大量旧材料，我还添加了诸如模块和交互式 JShell 等新特性的信息，并在此过程中更新了大量其他信息。

# 本书适合谁？

我假设你已经掌握了 Java 的基础知识。我不会告诉你如何 `println` 一个字符串，也不会告诉你如何编写一个继承另一个类和/或实现接口的类。我假设你已经学过像 [Learning Tree 的介绍](https://learningtree.com/471) 或者像 [*Head First Java*](http://shop.oreilly.com/product/9780596009205.do)、[*Learning Java*](http://shop.oreilly.com/product/0636920023463.do) 或 [*Java in a Nutshell*](http://shop.oreilly.com/product/9780596007737.do)（O’Reilly）这样的入门书籍。不过，第一章 涵盖了一些你可能不太熟悉但理解后续内容所必需的技术。欢迎随意查阅！书籍的印刷版和电子版都有大量交叉引用。

# 本书内容包括什么？

Java 更适合于“大规模开发”或企业应用程序开发，而不是 Perl、Awk 或 Python 中的单行一次性脚本。这是因为它是一种编译型的面向对象语言。然而，随着 JShell 的出现（参见 Recipe 1.4），这种适用性在某种程度上有所改变。我用较短的 Java 类示例甚至是代码片段来说明许多技术；其中一些较简单的技术将使用 JShell 展示。所有的代码示例（除了一些一两行的代码）都在我的一个公共 GitHub 存储库中，因此你可以放心，这里所见到的每一段代码都已经编译过，而且大多数最近都运行过。

本书中一些较长的示例是我最初编写的工具，用于自动化一些单调的任务。例如，一个名为`MkIndex`（在*javasrc*存储库中）的工具读取我保存 Java 示例源代码的顶级目录，并构建一个适合浏览器的*index.html*文件。另一个例子是`XmlForm`，它用于将手稿的部分从 XML 转换为另一种出版软件所需的形式。`XmlForm`还通过另一个程序`GetMark`处理来自*javasrc*目录的完整和部分代码插入到书稿中。我提到的 Github 存储库中包含`XmlForm`，以及`GetMark`的后续版本，尽管这两者都没有用于第四版的构建。如今，O’Reilly 的 Atlas 出版软件使用[Asciidoctor](https://asciidoctor.org)，它提供了我们用于向书中插入文件和文件部分的机制。

## 本书的组织结构

让我们来看看本书的组织结构。每一章由几个食谱组成，描述了一个问题及其解决方案，以及一个代码示例。每个食谱中的代码都旨在基本上是自包含的；欢迎你在自己的项目中借用其中的一些片段。该代码以伯克利样式的版权分发，仅仅是为了阻止全面复制。

我从第一章，*入门：编译和运行 Java*开始，描述了在不同平台上编译程序的方法，以及在不同环境中运行它们（浏览器、命令行、窗口化桌面）和调试方法。

第二章，*与环境交互*，从编译和运行程序转向使其适应周围环境——计算机中存在的其他程序。

接下来的几章将涉及基本 API。第三章，*字符串和相关内容*集中讲解了 Java 中最基本但强大的数据类型之一，展示了如何组装、拆分、比较和重新排列你可能认为是普通文本的内容。本章还涵盖了国际化/本地化的话题，使得你的程序在阿克巴、阿富汗、阿尔及尔、阿姆斯特丹和法国的使用体验与在阿尔伯塔、阿肯色和阿拉巴马州的使用体验一样。

第四章，*使用正则表达式进行模式匹配*教你如何在许多字符串匹配和模式匹配问题领域中使用 Unix 的强大正则表达式技术。正则表达式处理在 Java 中已经标准化多年，但如果你不知道如何使用它，你可能会重新发明轮胎的平坦。

第五章，*数字*讨论了内置的数值类型，如`int`和`double`，以及相应的 API 类（`Integer`、`Double`等）和它们提供的转换和测试功能。还简要提到了“大数”类。由于 Java 程序员经常需要处理日期和时间，无论是本地的还是国际的，第六章，*日期和时间*涵盖了这个重要主题。

接   接下来的几章将涵盖数据处理。与大多数语言一样，Java 中的数组是线性的、索引的相似对象集合，如第七章，*使用 Java 构建数据结构*中讨论的那样。本章将继续介绍许多集合类：在`java.util`包中存储大量对象的强大方式，包括 Java 泛型的使用。

尽管在语法上与 C 等过程语言有些相似，Java 从根本上是一种面向对象编程（OOP）语言，并巧妙地融合了一些重要的函数式编程（FP）构造。第八章，*面向对象的技术*讨论了 OOP 的一些关键概念，包括常被重写的方法`java.lang.Object`以及设计模式的重要问题。Java 不是，也永远不会是，纯粹的 FP 语言。然而，使用一些 FP 的特性是可能的，随着 Java 8 及其对 lambda 表达式（即闭包）的支持，这一点变得越来越普遍。第九章，*函数式编程技术：函数式接口、流和并行集合*对此进行了讨论。

下一章涉及传统输入和输出的方面。第十章，“输入和输出：读取、写入和目录技巧”，详细说明了读取和写入文件的规则（如果您认为文件很无聊，请不要跳过这一章；您将在后面的章节中需要这些信息）。该章还向您展示了有关文件的其他内容——如查找其大小和上次修改时间——以及关于读取和修改目录、创建临时文件和重命名磁盘上的文件的所有其他信息。

大数据和数据科学已经成为一个事物，而 Java 正好适用于此。Apache Hadoop、Apache Spark 等大数据基础设施的大部分都是用 Java 编写的，并且可以通过 Java 进行扩展，如第十一章，“数据科学和 R”所述。R 编程语言在数据科学家、统计学家和其他科学家中很受欢迎。至少有两个用 Java 编写的 R 重新实现，并且 Java 也可以直接与标准 R 实现双向接口，因此本章也涵盖了 R。

因为 Java 最初被宣传为互联网的编程语言，花点时间在 Java 网络编程上是公平的。第十二章，“网络客户端”，介绍了从客户端角度讲解网络编程的基础知识，重点介绍了套接字。如今，许多应用程序需要访问 Web 服务，主要是 RESTful Web 服务，因此这似乎是必要的。然后我将在第十三章，“服务器端 Java”中转向服务器端，您将在其中学习一些服务器端编程技术。

用于数据交换的一种简单的基于文本的表示形式是 JSON，即 JavaScript 对象表示法。第十四章，“处理 JSON 数据”，描述了该格式以及一些出现的许多用于处理它的 API。

第十五章，“包和打包”，展示了如何创建一起工作的类包。本章还讨论了部署（又名分发和安装）软件的方法。

第十六章，“Java 线程”，告诉您如何编写看起来可以同时执行多个操作的类，并利用强大的多处理器硬件。

第十七章，“反射，或“一个名为 Class 的类””，让您了解一些秘密，例如如何机械地编写 API 交叉引用文档以及 Web 服务器如何加载任何旧的 Servlet——从未见过该特定类并运行它。

有时您已经编写并在另一种语言中运行的代码可以为您的部分工作，或者您希望将 Java 用作较大程序包的一部分。第十八章，“使用其他语言与 Java”，向您展示了如何运行外部程序（已编译或脚本），并直接与 C/C++或其他语言中的本机代码交互。

这本书的篇幅不足以包含我想告诉你的关于 Java 的一切。后记提供了一些结尾思考，并链接到我整理的每个 Java 开发人员都应了解的 Java API 的在线摘要。

最后，附录 A，*Java Then and Now*，以版本发布时间线的形式呈现了 Java 的传奇历史，因此无论你学习的是哪个版本的 Java，你都可以快速了解最新情况。

如此之多的主题，但页面有限！许多主题没有得到百分之百的覆盖；我试图包含每个 API 最重要或最有用的部分。要深入了解，请查看每个包的官方*javadoc*页面；许多这些页面上都有一些关于包如何使用的简短教程信息。

除了本书涵盖的 Java 部分外，还有两个其他平台版本已经标准化。Java Micro Edition (Java ME) 专注于诸如手持设备、手机和传真机等小型设备。在规模较大的服务器设备上，有[Eclipse Jakarta EE](https://projects.eclipse.org/projects/ee4j.jakartaee-platform)，它取代了以前的 Java EE，在上个世纪被称为 J2EE。Jakarta EE 专注于构建大型、可扩展的分布式应用程序。Jakarta EE 的 API 包括 Servlets、JavaServer Pages、JavaServer Faces、JavaMail、Enterprise JavaBeans (EJBs)、Container and Dependency Injection (CDI) 和 Transactions。Jakarta EE 的包通常以“javax”开头，因为它们不是核心包。本书仅涉及其中几个；还有一个[*Java EE 8 Cookbook*](https://www.oreilly.com/library/view/java-ee-8/9781788293037)由 Elder Moraes (O’Reilly) 编写，涵盖了一些 Jakarta EE 的 API，以及一个更早的[*Java Servlet & JSP Cookbook*](http://shop.oreilly.com/product/9780596005726.do)由 Bruce Perry (O’Reilly) 编写。

本书完全不涵盖 Java Micro Edition、Java ME。但说到手机和移动设备，你可能知道 Android 使用 Java 作为其编程语言。对 Java 开发人员而言，令人欣慰的是，Android 也使用了大部分核心 Java API，只是对于 Swing 和 AWT 提供了特定于 Android 的替代品。想要学习 Android 的 Java 开发人员可以考虑查看我的[*Android Cookbook*](http://shop.oreilly.com/product/0636920038092.do) (O’Reilly)，或者这本书的[网站](http://androidcookbook.com)。

# Java 图书

这本书包含了大量有用的信息。但由于涵盖的主题广泛，不可能对任何一个主题进行书籍长度的处理。因此，本书包含了许多网站和其他书籍的参考资料。在指出这些参考资料时，我希望能为我的目标读者提供帮助：那些想要更多了解 Java 的人。

在我看来，O'Reilly 出版的 Java 书籍市场上是最佳选择。随着 API 的不断扩展，它们的覆盖范围也在增加。查看完整的[O'Reilly Java 书籍系列](https://ssearch.oreilly.com/?i=1;m_Sort=searchDate;q=java+o%27reilly;q1=Books;x1=t1&act=sort)，你可以在大多数实体书店和虚拟书店购买它们。你也可以通过[O'Reilly 在线学习平台](http://oreilly.com)，一个付费订阅服务，在线阅读它们。当然，大多数书籍现在也提供电子书格式；O'Reilly 的电子书是无 DRM 的，因此你不必担心它们的拷贝保护方案将你锁定在特定的设备或系统上，就像某些其他出版商的做法一样。

尽管本书在适当的位置提到了许多书籍，但在这里还值得特别一提几本。

首先，David Flanagan 和 Benjamin Evan 的[*Java 程序员快速参考*](http://shop.oreilly.com/product/9780596007737.do)（O'Reilly）提供了语言和 API 的简要概述，以及对最重要的包的详细参考。这本书很方便放在你的电脑旁边。Bert Bates 和 Kathy Sierra 的[*Head First Java*](http://shop.oreilly.com/product/9780596009205.do)则是一本更为风趣的入门书，推荐给经验较少的开发者。

[*Java 8 Lambdas*](http://shop.oreilly.com/product/0636920030713.do)（Warburton，O'Reilly）介绍了 Java 8 引入的 Lambda 语法，支持函数式编程和更简洁的代码。

[*Java 9 模块化：开发可维护应用程序的模式与实践*](http://shop.oreilly.com/product/0636920049494.do)（Sander Mak 和 Paul Bakker，O'Reilly）涵盖了 Java 9 语言中关键的更改，专注于 Java 模块系统。

[*Java 虚拟机*](http://shop.oreilly.com/product/9781565921948.do)（Jon Meyer 和 Troy Downing，O'Reilly）将会吸引那些想深入了解底层技术的人。这本书已经停印，但可以在二手市场和图书馆找到。

一本权威（而且庞大）的 Swing GUI 编程书籍是[*Java Swing*](http://shop.oreilly.com/product/9780596004088.do)（Robert Eckstein 等人，O'Reilly）。

[*Java 网络编程*](http://shop.oreilly.com/product/0636920028420.do)和[*Java I/O*](http://shop.oreilly.com/product/9780596527501.do)，均由 Elliotte Harold（O'Reilly）编著，也是有用的参考资料。

对于 Java 数据库工作，推荐[*Java 数据库编程与 JDBC & Java*](http://shop.oreilly.com/product/9781565926165.do)（George Reese，O'Reilly）和*Pro JPA 2: 掌握 Java 持久化 API*（Mike Keith 和 Merrick Schincariol，Apress）。或者我的即将推出的[Java 数据库概述](https://darwinsys.com/db_in_java)。

虽然你现在读的这本书没有涵盖 Java EE 的内容，但我想提到两本相关的书：

+   阿伦·古普塔在《Java EE 7 Essentials》（O’Reilly）中详细介绍了企业版。

+   亚当·比恩的《真实世界 Java EE 模式：重新思考最佳实践》（http://realworldpatterns.com）提供了在设计和实现企业应用程序中的有用见解。

您可以在[O’Reilly 网站](https://shop.oreilly.com)找到更多信息。

最后，尽管它不是一本书，《Java 信息》（https://docs.oracle.com/en/java/javase/13/docs）在网络上有大量内容。该网页曾展示了一张显示 Java 所有组件的大图，这是一个“概念图”。早期版本显示在图 P-1 中；每个彩色框都是指向该特定技术详细信息的可点击链接。

![jcb4 0001](img/jcb4_0001.png)

###### 图 P-1\. Oracle 文档中的 Java 概念图

不管好坏，Java 的新版本已将此替换为文本页面；对于 Java 13，该页面位于[*https://docs.oracle.com/en/java/javase/13*](https://docs.oracle.com/en/java/javase/13)。

## 一般编程书籍

唐纳德·E·克努特的《计算机程序设计艺术》（Addison-Wesley）自 1968 年首次出版以来，一直是计算机学生们的灵感来源。第 1 卷涵盖了*基本算法*，第 2 卷是*半数值算法*，第 3 卷是*排序和搜索*，第 4A 卷是*组合算法，第一部分*。预计系列中的其余卷尚未完成。尽管他的示例与 Java 差距很大（他为示例发明了假想的汇编语言 MIX），但他对算法的讨论——关于计算机如何解决实际问题——与多年前一样，至今仍然相关。²

尽管它的代码示例现在看来有些过时，《编程风格的元素》一书由布赖恩·克尼甘和 P. J. 普劳格（麦格劳希尔）撰写，为一代程序员设定了风格基调。克尼甘和普劳格还合著了一对书籍，《软件工具》（Addison-Wesley）和《Pascal 语言中的软件工具》（Addison-Wesley），这两本书提供了大量关于编程的良好建议，以至于我曾经建议所有程序员都应该阅读。然而，这三本书现在已经过时；许多时候我想要用一种更现代的语言写一本后续书籍。现在我转而推荐克尼甘的后续之作《编程实践》—与罗布·派克（Addison-Wesley）合著—作为《软件工具》系列的延续。这本书延续了贝尔实验室在软件教科书中的卓越传统。在本书的早期版本中，我甚至从他们的书中借鉴了一小段代码，他们的 CSV 解析器。最后，克尼甘最近出版了《UNIX：历史与回忆》，他对 Unix 故事的诠释。

还请参阅安德鲁·亨特和大卫·托马斯的 *实用编程*（Addison-Wesley）。

## 设计书籍

彼得·科德的 *Java 设计*（PTR-PH/Yourdon Press）专门讨论了针对 Java 的面向对象分析和设计的问题。科德对 Java 的可观察者-观察者范式的实现持有一定的批评态度，并提供了自己的替代方案。

近年来关于面向对象设计最著名的书之一是 *设计模式*，作者是埃里希·伽玛、理查德·赫尔姆、拉尔夫·约翰逊和约翰·弗利西德斯（Addison-Wesley）。这些作者通常被称为“四人帮”，导致他们的书有时被称为 GoF 书。我的一位同事称它为“有史以来最好的面向对象设计书籍”，我同意；至少，它是最好之一。

马丁·福勒的 *重构*（Addison-Wesley）涵盖了很多可应用于代码的“清理”工作，以提高可读性和可维护性。正如 GoF 书籍引入了有助于开发人员和其他人沟通代码设计方式的新术语一样，福勒的书提供了一个讨论如何改进代码的词汇表。但这本书可能不如其他书有用；现在许多重构内容已经出现在 Eclipse IDE 的重构菜单中（参见 Recipe 1.3）。

目前流传的两个重要方法论流派。第一个被称为敏捷方法，最著名的成员是 [Scrum](https://en.wikipedia.org/wiki/Scrum_(software_development)) 和极限编程（XP）。XP（方法论，而不是那个真的很老的微软 OS 版本）由其设计者肯特·贝克领导，以一系列小型、简短、易读的文本呈现。XP 系列的第一本书是 *极限编程解析*（Addison-Wesley）。对所有敏捷方法的很好概述是吉姆·海斯密斯的 *敏捷软件开发生态系统*（Addison-Wesley）。

另一组重要的方法论书籍，涵盖了更传统的面向对象设计，是由“三位好友”（布奇、雅各布森和朗保）领导的 UML 系列。他们的主要作品是 *UML 用户指南*、*UML 过程* 等等。同一系列中一本更小、更易接近的书是马丁·福勒的 *UML 精要*。

# 本书中使用的约定

本书使用以下约定。

## 编程约定

本书中我使用以下术语。程序指任何可运行的代码单元：从五行主程序到 Servlet 或 Web 层组件、EJB 或完整的 GUI 应用程序。小程序是用于在 Web 浏览器中使用的 Java 程序；这些曾一度流行，但今天几乎不存在。Servlet 是使用 Jakarta EE API 构建的 Java 组件，用于通过 HTTP 在 Web 服务器中使用。EJB 是使用 Jakarta API 构建的业务层组件。应用程序指任何其他类型的程序。桌面应用程序（也称为客户端）与用户进行交互。服务器程序间接与客户端交互，通常通过网络连接（今天通常是 HTTP/HTTPS）。

所示示例有两种变体。以零个或多个导入语句、一个 javadoc 注释和一个`public class`语句开头的是完整示例。以声明或可执行语句开头的当然是摘录。但这些摘录的完整版本已经被编译和运行，在线资源包括完整版本。

配方按章节和编号编号，例如，Recipe 8.1 指的是第八章中的第一个配方。

## 排版约定

本书使用以下排版约定：

*斜体*

用于命令、文件名和示例网址。在文本中首次出现时，还用于强调和定义新术语。

`等宽字体`

在代码示例中用于显示部分或完整的 Java 源代码程序清单。还用于类名、方法名、变量名和其他 Java 代码片段。

**`等宽粗体字体`**

用于用户输入，例如在命令行上键入的命令。

*`等宽斜体字体`*

显示应该被用户提供的值或由上下文确定的值替换的文本。

###### 提示

此元素表示提示或建议。

###### 注意

此元素表示一般注释。

###### 警告

此图标表示警告或注意事项。

## 代码示例

本书的代码示例位于作者的 GitHub 上。大多数位于仓库[*javasrc*](https://github.com/IanDarwin/javasrc)，但少数从另一个仓库[*darwinsys-api*](https://github.com/IanDarwin/darwinsys-api)中拉取。有关下载这些内容的详细信息，请参见 Recipe 1.6。

许多程序都附有示例，展示它们在命令行中的运行方式。这些示例通常会显示以`$`为结尾的 Unix 提示符或以`>`为结尾的 Windows 提示符，具体取决于我编写示例时使用的计算机。如果在这个提示符字符之前有文本，可以忽略它。它可能是路径名或主机名，同样取决于系统。

当从命令行启动程序时，这些示例通常还会显示类的完整包名称。因为 Java 要求这样做，而且这将提醒您在源代码库的哪个子目录中找到源代码，所以我不会经常明确指出它。

我们感谢但通常不要求署名。署名通常包括标题、作者、出版商和 ISBN 号。例如：“*Java Cookbook* by Ian F. Darwin (O’Reilly). Copyright 2020 RejmiNet Group, Inc., 978-1-492-07258-4。”

如果您认为您对代码示例的使用超出了合理使用范围或以上授权，请随时通过*permissions@oreilly.com*与我们联系。

# 奥莱利在线学习

###### 注意

40 多年来，[*奥莱利媒体*](http://oreilly.com)已为技术和商业培训提供了知识和见解，以帮助公司取得成功。

我们独特的专家和创新者网络通过图书、文章、会议和我们的在线学习平台分享他们的知识和专业知识。奥莱利的在线学习平台为您提供按需访问的实时培训课程、深入学习路径、交互式编码环境以及奥莱利和其他 200 多家出版商的大量文本和视频。欲了解更多信息，请访问[*http://oreilly.com*](http://oreilly.com)。

# 评论和问题

正如前面提到的，我已经在至少一个参考平台上测试了所有代码，大多数情况下是在多个平台上测试过。尽管如此，我的代码或某些重要的 Java 实现中可能存在平台依赖性或甚至错误。请报告您发现的任何错误以及对未来版本的建议，写信至：

+   奥莱利媒体公司

+   1005 Gravenstein Highway North

+   CA 95472，Sebastopol

+   800-998-9938（美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

本书有一个网页，我们列出了勘误、示例和任何额外信息。您可以访问[*http://shop.oreilly.com/product/0636920304371.do*](http://shop.oreilly.com/product/0636920304371.do)。

电子邮件*bookquestions@oreilly.com*以评论或询问有关本书的技术问题。

欲了解更多关于我们的图书、课程、会议和新闻的信息，请访问我们的网站[*http://www.oreilly.com*](http://www.oreilly.com)。

在 Facebook 上找到我们：[*http://facebook.com/oreilly*](http://facebook.com/oreilly)

在 Twitter 上关注我们：[*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)

在 YouTube 上观看我们：[*http://www.youtube.com/oreillymedia*](http://www.youtube.com/oreillymedia)

奥莱利网站列出了勘误表。您也可以找到所有 Java 代码示例的源代码以供下载；*请*不要浪费时间重新输入它们！有关具体说明，请参见 Recipe 1.6。

# 致谢

我在第一版的后记中写道，“写这本书是一次令人印象深刻的经历。”我应该补充说，维护这本书也同样令人感到印象深刻。尽管许多人对此赞不绝口——一位非常友好的评论者称其为“有史以来关于 Java 编程语言写得最好的书”——但我被早期版本中的错误和遗漏数量所震惊。我努力进行了改正。

生活中多次被命运的流动触及，让我与正确的人在正确的时间接触到正确的事物。与我早已失去联系的史蒂夫·芒罗在我们高中同班时向我介绍了计算机，特别是多伦多教育委员会的一台比起客厅还大、有 32 或 64K（不是 M 或 G！）内存的 IBM 360/30。后来在多伦多大学，已故的赫伯特·库格尔在我学习后来的更大型 IBM 主机时关心着我。特里·伍德和丹尼斯·史密斯在我接触 IBM PC 之前介绍了我迷你和微型计算机。在多伦多商业俱乐部的 Toastmasters International 和阿尔·兰伯特的加拿大潜水学校，我得以发展我的演讲和教学能力。多伦多大学的几位人士，尤其是杰弗里·科利尔，教会了我 Unix 操作系统的特性和好处，正是在我准备学习它的时候。

感谢许多[Learning Tree](https://www.learningtree.com)的讲师和学生向我展示了如何改进我的演讲。我仍然为“The Tree”教书，并推荐他们的课程给那些希望在四天内深入了解一个主题的忙碌开发者。

离这个项目更近的是，当我的“小林书”只是一个提议的更长作品的样章时，蒂姆·奥莱利相信了它，使我得以早日进入奥莱利作者的尊贵圈子。多年后，迈克·劳基德斯鼓励我继续寻找一个我们两个都能合作的 Java 书籍创意。在我一次次未能按时完成截稿时，他一直支持着我。迈克还阅读了整个手稿，并提出了许多明智的意见，其中一些把不切实际的想法拉回了现实。杰萨敏·里德将许多传真和电子邮件上的难以辨认的涂鸦变成了你在这本书中看到的高质量插图。还有许多奥莱利的其他才华横溢的人帮助把这本书变成了你现在看到的形态。

现在代码示例已经动态包含（因此更新更快），而不是直接粘贴进来。我的儿子（功能编程开发者）Benjamin Darwin 通过将几乎整个代码库转换为 O'Reilly 最新的“包含”机制，并解决其他几个非 Java 展示问题，帮助满足了截止日期。他还帮助使第九章更清晰和更实用。

## 在 O'Reilly

在这本书的第四版中，Suzanne McQuade 担任编辑监督，Corbin Collins 担任主编。Corbin 在校对手稿方面尤为细致。Meghan Blanchette、Sarah Schneider、Adam Witwer、Melanie Yarbrough 以及在版权页列出的许多制作人员都为第三版的完成贡献了自己的力量。感谢 Mike Loukides、Deb Cameron 和 Marlowe Shaeffer 在第二版的编辑和制作工作中的贡献。

## 技术审阅员

对于第四版，我很荣幸能够拥有两位非常彻底的技术审阅员，Sander Mak 和 Daniel Hinojosa。这两位指出了我在主要修订期间未考虑到的许多问题，在 O'Reilly 制作团队接管前的最后几周进行了广泛的重写和修改。非常感谢你们两位！

我的第三版审稿人 Alex Stangl 阅读了第三版的手稿，并超出了职责范围，提出了无数有益的建议，甚至发现了之前版本中存在的错别字！Benjamin Darwin、Mark Finkov 和 Igor Savin 对特定章节提出了有益的建议。如果我遗漏了任何人，请接受我诚挚的感谢！

Bil Lewis 和 Mike Slinn 在第一版的多个草稿中提出了有益的评论。Ron Hitchens 和 Marc Loy 仔细阅读了第一版的整个最终草稿。我非常感谢 Mike Loukides 在整个过程中的鼓励和支持。编辑 Sue Miller 在生产的最后阶段积极推动手稿的完成。Sarah Slocombe 阅读了 XML 章节的全文，并提出了许多清晰的建议；不幸的是，时间不允许我在第一版中包含所有建议。

Jonathan Knudsen、Andy Oram 和 David Flanagan 在这本书的大纲还只是章节列表时提出了评论，他们能够看到这本书可能成为的类型，并提出了改进的方法。

每一位参与者都在很多方面使这本书变得更好，特别是通过提出额外的配方建议或修改现有的配方。感谢每一位！尚存的错误均属于我的疏忽。

## 读者

我衷心感谢所有找到勘误并提出改进建议的读者。每一版新书都因像你们这样花时间和精力来报告需要改进的地方的人而变得更好！

特别要提到本书的德文译者之一³，吉斯伯特·塞尔克（Gisbert Selke），他在翻译第一版时从头到尾阅读了整本书，并澄清了我的英语。吉斯伯特为第二版做了同样的工作，并提供了许多代码重构，使得这本书比其他情况下更好。吉斯伯特甚至超越了职责范围，为这本书做出了一份食谱（Recipe 18.5），并修改了同一章节中其他一些食谱。谢谢你，吉斯伯特！

第二版还受益于吉姆·伯吉斯的评论，他阅读了书的大部分内容。乔纳森·弗尔斯、已故的金·福勒、马克·洛伊和迈克尔·麦克洛斯基分别对各个章节提出了评论。我的妻子贝蒂和当时还是十几岁的孩子们也分别校对了几章内容。

以下人员提供了重要的错误报告或建议改进：雷克斯·博斯马、罗德·布坎南、约翰·张伯伦、基思·戈德曼、吉尔-菲利普·格雷戈瓦、B·S·休斯、杰夫·约翰斯顿、罗布·康尼斯伯格、汤姆·默塔格、乔纳森·奥康纳、马克·彼得罗维奇、史蒂夫·赖斯曼、布鲁斯·X·史密斯和帕特里克·沃尔温德。

## 等等。

亲爱的妻子贝蒂·赛拉依然对我在编程时喝的咖啡充满了了解，比我使用的编程语言更加了解，但是她对清晰表达和正确语法的热情在我们共同生活的岁月里给我的写作带来了很多好处。

没有一本关于 Java 的书能完整地没有感谢詹姆斯·高斯林发明 Java（他还发明了第一个 Unix Emacs、*sc*电子表格和 NeWS 窗口系统）。同时也感谢他的雇主 Sun Microsystems（在被 Oracle 收购之前），他们不仅发布了 Java 语言，还在互联网上免费提供了一系列令人难以置信的 Java 工具和 API 库。

在第一版的早期，苹果加拿大的威利·鲍威尔为我提供了 macOS 的访问权限。

对于每一位贡献者，我真诚地感谢你们。

## 书籍制作软件

在准备、编译和测试本书时，我使用了各种工具和操作系统。值得感谢 OpenBSD 的开发者们，他们创造了“主动安全的类 Unix 系统”，提供了一个稳定和安全的 Unix 克隆系统，比其他免费软件更接近传统 Unix。我在输入原始手稿的 XML 时使用了*vi*编辑器（在 OpenBSD 上是*vi*，在 Windows 上是*vim*），并使用 Adobe FrameMaker（一个精美的基于 GUI 的文档工具，后来被 Adobe 收购并毁掉）来格式化文档。我不知道我是否能原谅 Adobe 毁掉了可以说是世界上最好的文档系统，还通过保持充满 bug 的 Flash 活跃使互联网变得如此危险。但我知道我永远不会再使用 Adobe 的文档系统来做任何事情。

由于此原因，我编辑的众包 *[Android Cookbook](http://shop.oreilly.com/product/0636920038092.do)* 并不是用 FrameMaker 准备的，而是使用了 XML DocBook（从我为此目的编写的基于 Java 的维基标记生成）和 O'Reilly 工具组提供的一些定制工具。

*Java Cookbook* 的第三版和第四版是用 [Asciidoctor](http://asciidoctor.org) 格式化的，并在 O'Reilly 的 [Atlas](http://atlas.oreilly.com) 出版工具链上实现。

¹ 有关怪癖，请参见 Joshua Bloch 和 Neal Gafter 的 [*Java Puzzlers* books](http://javapuzzlers.com)（Addison-Wesley）。

² 在考虑到目前计算能力的巨大变化后，对于不再那么相关的算法决策可能会有例外情况。

³ 较早的版本有或者曾经有英文、德文、法文、波兰文、俄文、韩文、繁体中文和简体中文版本。感谢所有翻译人员为使这本书能够面向更广泛的读者群体所做的努力。