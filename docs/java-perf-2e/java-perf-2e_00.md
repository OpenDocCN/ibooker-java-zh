# 前言

当 O'Reilly 首次找我写一本关于 Java 性能调优的书时，我有些犹豫。我想，Java 性能调优？这不是早就完成了吗？是的，我仍然每天都在努力改进 Java（以及其他）应用程序的性能，但我更愿意认为，我大部分时间都在处理算法效率低下和外部系统瓶颈，而不是直接与 Java 调优相关的内容。

一瞬间的反思让我确信，我又一次在自欺欺人。无疑，整体系统性能占据了我大部分时间，有时我会遇到使用**O(n²)** 算法而本可以使用*O(logN)* 性能的代码。尽管如此，事实证明，每天我都会思考垃圾收集（GC）性能，或者 JVM 编译器的性能，或者如何从 Java APIs 中获得最佳性能。

这并不是为了贬低过去 20 多年来在 Java 和 JVM 性能方面取得的巨大进展。当我在上世纪 90 年代末担任 Sun 的 Java 传道士时，唯一真正的“基准测试”是来自 Pendragon 软件的 CaffeineMark 2.0。由于各种原因，那个基准测试的设计很快就限制了其价值；然而在当时，我们喜欢告诉大家，基于那个基准测试，Java 1.1.8 的性能比 Java 1.0 的性能快了八倍。而这是真实的——Java 1.1.8 拥有一个真正的即时编译器，而 Java 1.0 基本完全是解释执行的。

然后，标准委员会开始制定更严格的基准测试，Java 性能开始围绕这些测试展开。结果是 JVM 的所有领域——垃圾收集、编译和 API 内部——都在持续改进。当然，这个过程至今仍在进行中，但关于性能工作的一个有趣事实是，它变得越来越困难。通过引入即时编译器实现性能增长八倍是一件直截了当的工程问题，尽管编译器继续改进，但我们不会再见到类似的改进了。并行化垃圾收集器是一个巨大的性能改进，但近年来的变化更多地是渐进的。

这是应用程序的典型过程（而 JVM 本身只是另一个应用程序）：在项目开始阶段，很容易找到可以带来巨大性能改进的架构变更（或代码缺陷）。在成熟的应用程序中，找到这样的性能改进是很少见的。

这个原则是我最初关心的基础，很大程度上，工程界可能已经完成了关于 Java 性能的讨论。有几件事让我改变了看法。首先是我每天看到关于 JVM 在特定情况下的表现如何的问题数量。新工程师们一直在学习 Java，并且在某些领域，JVM 的行为仍然足够复杂，因此操作指南仍然有益。其次是计算环境的变化似乎改变了工程师们今天面临的性能问题。

过去几年来，性能问题已经变得复杂多样。一方面，现在很普遍使用能够运行具有非常大堆内存的 JVM 的超大机器。JVM 已经采取措施来解决这些问题，引入了新的垃圾收集器（G1），作为一种新技术，需要比传统的收集器更多的手动调优。与此同时，云计算重新强调了小型、单 CPU 机器的重要性：你可以去 Oracle、Amazon 或其他许多公司，廉价租用一个单 CPU 机器来运行小型应用服务器。（实际上你并没有获得一个单 CPU 机器：你得到的是一个虚拟操作系统镜像在一个非常大的机器上运行，但虚拟操作系统限制了使用单 CPU。从 Java 的角度来看，这与单 CPU 机器是相同的。）在这些环境中，正确管理少量内存变得非常重要。

Java 平台也在不断发展。每个新版 Java 都提供新的语言特性和新的 API，以提高开发者的生产力——尽管不总是提高应用程序的性能。合理使用这些语言特性的最佳实践可以帮助区分一个优秀的应用程序和一个平庸的应用程序。平台的演进也带来了有趣的性能问题：使用 JSON 在两个程序之间交换信息要比设计高度优化的专有协议简单得多。为开发者节省时间是一个重大的胜利——但确保这种生产力的提升伴随着性能的提升（或至少不下降）才是真正的目标。

# 谁应该（和不应该）阅读本书

本书旨在帮助性能工程师和开发人员了解 JVM 和 Java API 各方面对性能的影响。

如果现在是星期天深夜，你的网站将在星期一上线，而你正在寻找快速解决性能问题的方法，那这本书不适合你。

如果您是性能分析新手，并在 Java 中开始分析，本书可以帮助您。当然，我的目标是提供足够的信息和背景，使初学者工程师能够理解如何将基本的调优和性能原则应用于 Java 应用程序。然而，系统分析是一个广泛的领域。有许多关于系统分析的优秀资源（当然，这些原则也适用于 Java），从这个意义上讲，本书理想情况下将是这些文本的有用伴侣。

然而，在根本层面上，使 Java 运行速度真正快需要深入理解 JVM（和 Java API）的实际工作方式。存在数百个 Java 调优标志，并且调优 JVM 不应该只是盲目尝试它们并查看哪个有效。相反，我的目标是提供关于 JVM 和 API 正在做什么的详细知识，希望如果你理解了这些东西是如何工作的，你就能够查看应用程序的具体行为并理解为什么它表现糟糕。理解了这一点，消除不良（性能不佳）行为变得简单（或者至少更简单）起来。

Java 性能工作的一个有趣方面是，开发人员通常与性能或质量保证组的工程师背景大不相同。我知道有些开发人员可以记住成千上万个不常用的 Java API 方法签名，但却不知道 `-Xmn` 标志的含义。我也知道测试工程师可以通过设置垃圾收集器的各种标志来获得最后一丝性能，但是在 Java 中几乎无法编写一个合适的“Hello, World”程序。

Java 的性能涵盖了这两个方面：调优编译器和垃圾收集器等标志，以及最佳实践中 API 的使用。因此，我假设你已经很好地理解了如何在 Java 中编写程序。即使你的主要兴趣不在 Java 的编程方面，我也花了大量时间讨论程序，包括用于示例中许多数据点的示例程序。

不过，如果你主要关注的是 JVM 本身的性能——即如何在不进行任何编码的情况下改变 JVM 的行为——那么本书的大部分内容仍然对你有益。请随意跳过编码部分，专注于你感兴趣的领域。也许在这个过程中，你会对 Java 应用程序如何影响 JVM 性能有所了解，并开始建议开发人员进行更改，以便让你的性能测试工作更轻松。

# 第二版的新内容

自第一版以来，Java 采用了每六个月发布一次的周期，定期进行长期支持发布；这意味着与出版同时支持的当前版本是 Java 8 和 Java 11。虽然第一版覆盖了 Java 8，在当时还是非常新的。本版侧重于更加成熟的 Java 8 和 Java 11，重点更新了 G1 垃圾收集器和 Java Flight Recorder。还关注 Java 在容器化环境中行为变化的变化。

本版涵盖了 Java 平台的新功能，包括新的微基准测试工具（`jmh`）、新的即时编译器、应用程序类数据共享和新的性能工具，以及对 Java 11 新功能（如紧凑字符串和字符串连接）的覆盖。

# 本书使用的约定

本书中使用了以下排版约定：

*斜体*

指示新术语、URL、电子邮件地址、文件名和文件扩展名。

`固定宽度`

用于程序清单，以及在段落内引用程序元素，如变量或函数名称、数据库、数据类型、环境变量、语句和关键字。

**`固定宽度粗体`**

显示用户应该按字面输入的命令或其他文本。

*`固定宽度斜体`*

显示应替换为用户提供的值或由上下文确定的值的文本。

此元素表示主要点的摘要。

# 使用代码示例

补充材料（代码示例、练习等）可从[*https://github.com/ScottOaks/JavaPerformanceTuning*](https://github.com/ScottOaks/JavaPerformanceTuning)下载。

本书旨在帮助您完成工作。通常情况下，如果本书提供示例代码，您可以在您的程序和文档中使用它。除非您复制了代码的大部分，否则无需联系我们以获得许可。例如，编写使用本书中几个代码片段的程序不需要许可。出售或分发 O’Reilly 书籍的示例代码需要许可。通过引用本书并引用示例代码回答问题不需要许可。将本书大量示例代码整合到产品文档中需要许可。

我们感谢，但不要求归属。归属通常包括标题、作者、出版商和 ISBN。例如：“*Java 性能* by Scott Oaks (O’Reilly)。版权 2020 Scott Oaks，978-1-492-05611-9。”

如果您觉得您使用的代码示例超出了合理使用范围或上述许可，请随时通过[*http://oreilly.com*](http://oreilly.com)与我们联系。

# 致谢

我要感谢在我写作这本书的过程中帮助过我的每一个人。在很多方面，这本书积累了我在 Java 性能组及其他 Sun Microsystems 和 Oracle 工程组工作的 20 年知识，因此提供积极反馈的人员名单相当广泛。感谢在这段时间内与我合作的所有工程师，特别是那些耐心回答我随机问题的人，谢谢你们！

我特别要感谢 Stanley Guan、Azeem Jiva、Kim LiChong、Deep Singh、Martijn Verburg 和 Edward Yue Shung Wong，他们花时间审阅草稿并提供宝贵的反馈。尽管这里的材料得到了他们的帮助而显著改进，但我确信他们无法找出所有我的错误。第二版得到了 Ben Evans、Rod Hilton 和 Michael Hunger 全面和周到的帮助，极大地改进了这本书。我的同事 Eric Caspole、Charlie Hunt 和 Robert Strout 在 Oracle HotSpot 性能组中耐心地帮助我解决了各种问题。

在 O’Reilly 的制作人员一如既往地非常乐于助人。我很荣幸能与编辑 Meg Blanchette 共同完成第一版，而 Amelia Blevins 则细致而仔细地指导了第二版。在整个过程中，感谢你们的鼓励！最后，我必须感谢我的丈夫 James，忍受了漫长的夜晚以及那些周末晚餐，而我总是心不在焉。
