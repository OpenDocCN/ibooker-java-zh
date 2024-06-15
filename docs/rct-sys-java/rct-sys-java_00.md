# 前言

在 IT 世界中，今天的限制是明天的门户。在过去的 50 年中，IT 世界不知疲倦地不断发展，始终在突破界限。这些变化不仅源于技术进步，还因为我们——消费者。作为消费者，我们每天与之互动的软件要求越来越多。此外，我们与软件互动的方式已完全改变。我们无法没有移动应用和设备，现在也接受全天候接收通知。物联网（IoT）是一个新兴市场，承诺更多创新，不断增加处理的事件和数据量。云计算和 Kubernetes 不仅改变了我们的使用方式，还彻底改变了我们设计、开发、部署和维护应用程序的方式。

但不要误解；所有这些革命都是有代价的。虽然它们使新的用途和应用成为可能，但也引入了巨大的复杂性。今天大多数软件系统都是分布式系统。而分布式系统很难设计、构建和操作，尤其是在我们需要实现这些新的现代应用的规模上。我们需要处理故障、异步通信、不断变化的拓扑结构、动态可用性的资源等等。虽然云计算承诺无限资源，但资金是一个限制因素，增加部署密度，即在较少资源上运行更多内容，成为一个严峻的问题。

那么，什么是*Reactive*？它不是你在代码中使用的库，也不是魔法框架。*Reactive*是一套原则、工具、方法和框架，有助于构建更好的分布式系统。究竟*多*好？这取决于系统，但是遵循 Reactive 原则的应用程序可以应对分布式系统的挑战，并专注于弹性、韧性和响应性，正如在[响应式宣言](https://oreil.ly/fO6n0)中所解释的那样。

在本书中，我们使用大写*R*的名词*Reactive*来涵盖响应式领域的各种方面，例如响应式编程、响应式系统、响应式流等。通过本书，您将了解 Reactive 如何帮助我们应对这些新的关注点，并在云环境中的适应性。阅读完本书后，您将能够构建响应式系统——弹性、适应性、事件驱动的分布式系统。

# 谁应该阅读本书？

本书的目标读者是中级和高级 Java 开发人员。您应该对 Java 相当熟悉；但不需要有响应式编程或响应式的先验知识。本书中的许多概念涉及分布式系统，但您也不需要对其熟悉。

反应式系统通常依赖于诸如 Apache Kafka 或高级消息队列协议（AMQP）之类的消息代理。本书介绍了您理解这些代理如何帮助设计和实现反应式系统所需的基本知识。

本书可以使三个不同的群体受益：

+   正在构建云原生应用程序或分布式系统的开发人员

+   寻求理解反应式和事件驱动架构角色的架构师

+   对反应式感兴趣并希望更好地理解的好奇开发者

通过本书，您将开始了解、设计、构建和实施反应式架构的旅程。您不仅将学习如何帮助构建*更好*的分布式系统和云应用程序，还将了解如何使用反应式模式改进现有系统。

# Quarkus 到底是什么？

细心的读者可能已经注意到本书副标题中提到了 Quarkus。但是，到目前为止，我们还没有提到它。*Quarkus* 是专为云而设计的 Java 堆栈。它使用构建时技术来减少应用程序使用的内存量，并提供快速的启动时间。

但是 Quarkus 也是一个反应式堆栈。在其核心，反应式引擎使得创建并发和具有弹性的应用程序成为可能。Quarkus 还提供了构建分布式系统所需的所有功能，这些系统能够适应波动的负载和不可避免的故障。

在本书中，我们使用 Quarkus 来演示反应式方法的优势，并介绍各种模式和最佳实践。如果您对此不熟悉或缺乏经验，不要担心。我们将在旅程中陪伴您，指导您每一步。

本书专注于创建利用 Quarkus 能力的反应式应用程序和系统，并提供了构建此类系统所需的所有知识。我们不涵盖完整的 Quarkus 生态系统，因为本书专注于帮助构建反应式系统的 Quarkus 组件。

# 导读本书

如果您刚刚接触反应式并希望了解更多信息，全书阅读将带给您对反应式及其如何帮助您的理解。如果您是一位经验丰富的反应式开发人员，对 Quarkus 及其反应式特性感兴趣，您可能想跳过本书的第一部分，直接阅读最感兴趣的章节。

第一部分 是一个简短的介绍，设定了背景：

+   第一章 提供了反应式领域的简要概述，包括其优点和缺点。

+   第二章 介绍了 Quarkus 及其通过构建时方法减少启动时间和内存使用的方法。

第二部分 概述了一般的反应式内容：

+   第三章 解释了分布式系统的复杂性和对反应式的误解；这些是成为反应式的原因。

+   第四章 展示了响应式系统的特性。

+   第五章 讨论了各种形式的异步开发模型，重点介绍了响应式编程。

第 III 部分 解释了如何使用 Quarkus 构建响应式应用程序：

+   第六章 探讨了响应式引擎以及如何在命令式和响应式编程之间桥接。

+   第七章 深入讲解了 SmallRye Mutiny，即 Quarkus 中使用的响应式编程库。

+   第八章 解释了 HTTP 请求的特性以及我们如何在响应式中进行处理。

+   第九章 解释了如何使用 Quarkus 构建与数据库交互的高并发高效应用程序。

最后部分，第 IV 部分，*连接了各个点*，展示了如何使用 Quarkus 构建响应式系统：

+   第十章 深入探讨了 Quarkus 应用程序与消息传递技术的集成，这是响应式系统的重要组成部分。

+   第十一章 着重介绍了与 Apache Kafka 和 AMQP 的集成，以及如何利用它们构建响应式系统。

+   第十二章 探索了从 Quarkus 应用程序消费 HTTP 端点的各种方式，以及如何实施弹性和响应性。

+   第十三章 涵盖了响应式系统中的可观察性问题，如自愈、追踪和监控。

# 为您做好准备

在本书中，您将看到许多代码示例。这些示例说明了本书涵盖的概念。有些示例很基础，可以在 IDE 中运行，而其他一些则需要一些先决条件。

我们逐一覆盖这些示例，遍布整本书。也许你对悬念不感兴趣。或者更可能的是，也许你已经厌倦了我们啰嗦地长篇大论，只是想看到它如何运作。如果是这样，只需将您的浏览器指向[*https://github.com/cescoffier/reactive-systems-in-java*](https://github.com/cescoffier/reactive-systems-in-java)，随意试探一下。您可以使用`git clone https://github.com/cescoffier/reactive-systems-in-java.git`命令从 Git 获取代码。或者，您可以下载一个[ZIP 文件](https://oreil.ly/Ey74z)，并解压缩它。

本书的代码按章节组织。例如，与第二章相关的代码位于*chapter-2*目录（见表 P-1）。根据章节的不同，代码可能会分成多个模块。书中的代码示例在代码库中的位置可以从书中的代码片段标题找到。

表 P-1\. 各章节的代码位置

| 章节 | 标题 | 路径 |
| --- | --- | --- |
| 第二章 | Quarkus 简介 | [*https://github.com/cescoffier/reactive-systems-in-java/tree/master/chapter-2*](https://github.com/cescoffier/reactive-systems-in-java/tree/master/chapter-2) |
| 第三章 | 分布式系统的黑暗面 | [*https://github.com/cescoffier/reactive-systems-in-java/tree/master/chapter-3*](https://github.com/cescoffier/reactive-systems-in-java/tree/master/chapter-3) |
| 第四章 | 响应式系统的设计原则 | [*https://github.com/cescoffier/reactive-systems-in-java/tree/master/chapter-4*](https://github.com/cescoffier/reactive-systems-in-java/tree/master/chapter-4) |
| 第五章 | 响应式编程：驯服异步性 | [*https://github.com/cescoffier/reactive-systems-in-java/tree/master/chapter-5*](https://github.com/cescoffier/reactive-systems-in-java/tree/master/chapter-5) |
| 第七章 | Mutiny：基于事件驱动的响应式编程 API | [*https://github.com/cescoffier/reactive-systems-in-java/tree/master/chapter-7*](https://github.com/cescoffier/reactive-systems-in-java/tree/master/chapter-7) |
| 第八章 | 面向响应式的 HTTP | [*https://github.com/cescoffier/reactive-systems-in-java/tree/master/chapter-8*](https://github.com/cescoffier/reactive-systems-in-java/tree/master/chapter-8) |
| 第九章 | 响应式数据访问 | [*https://github.com/cescoffier/reactive-systems-in-java/tree/master/chapter-9*](https://github.com/cescoffier/reactive-systems-in-java/tree/master/chapter-9) |
| 第十章 | 响应式消息传递：连接组织的纽带 | [*https://github.com/cescoffier/reactive-systems-in-java/tree/master/chapter-10*](https://github.com/cescoffier/reactive-systems-in-java/tree/master/chapter-10) |
| 第十一章 | 事件总线：支撑系统的骨干 | [*https://github.com/cescoffier/reactive-systems-in-java/tree/master/chapter-11*](https://github.com/cescoffier/reactive-systems-in-java/tree/master/chapter-11) |
| 第十二章 | 响应式 REST 客户端：与 HTTP 端点连接 | [*https://github.com/cescoffier/reactive-systems-in-java/tree/master/chapter-12*](https://github.com/cescoffier/reactive-systems-in-java/tree/master/chapter-12) |
| 第十三章 | 观察响应式和事件驱动架构 | [*https://github.com/cescoffier/reactive-systems-in-java/tree/master/chapter-13*](https://github.com/cescoffier/reactive-systems-in-java/tree/master/chapter-13) |

代码存储库中的示例使用 Java 11，所以请确保你的机器上安装了合适的 Java 开发工具包（JDK）。它们还使用 Apache Maven 作为构建工具。你不需要安装 Maven，因为代码库使用了 [Maven Wrapper](https://oreil.ly/0oKc9)（自动提供 Maven）。但是，如果你更喜欢手动安装，可以从 [Apache Maven 项目网站](https://oreil.ly/XgiCr) 下载它，并按照 [安装 Apache Maven 页面](https://oreil.ly/nwJV9) 上的说明进行操作。

要构建代码，请从项目根目录运行`mvn verify`。 Maven 将下载一组构建工件，所以请确保有互联网连接。

本书介绍了 Quarkus，一种 Kubernetes 原生的 Java 堆栈。只要你有 Java 和 Maven，你就不需要安装任何东西来使用 Quarkus。它会自动下载其他所有内容。

你将需要 Docker。Docker 用于为我们的应用程序创建容器。请按照 [获取 Docker 页面](https://oreil.ly/DjBnj) 上的说明安装 Docker。

最后，本书的几章介绍了我们在 Kubernetes 中部署反应式应用程序的过程。要部署到 Kubernetes，你首先需要`kubectl`，这是一个与 Kubernetes 交互的命令行工具。按照 [Kubernetes 安装工具页面](https://oreil.ly/4SA4J) 的说明安装它。除非你有一个方便的 Kubernetes 集群，否则我们还建议在你的机器上安装 minikube，以提供 Kubernetes 环境。请按照 [minikube 网站](https://oreil.ly/vuCs1) 上的说明安装它。

为什么我们需要所有这些工具？本书中你会看到，反应式既会给你的应用程序带来约束，也会给你的基础设施带来约束。Kubernetes 提供了我们部署应用程序、创建副本并保持系统运行的原语。另一方面，Quarkus 提供了我们实现反应式应用程序所需的一系列功能，包括非阻塞 I/O、反应式编程、反应式 API 和消息传递能力。Quarkus 还提供了与 Kubernetes 的集成，以便轻松部署和配置应用程序。

表 P-2 列出了本书中要使用的工具。

表 P-2. 本书中使用的工具

| 工具 | 网站 | 描述 |
| --- | --- | --- |
| Java 11 | [*https://adoptopenjdk.net*](https://adoptopenjdk.net) | Java 虚拟机（JVM）和 Java 开发工具包（JDK） |
| Apache Maven | [*https://maven.apache.org/download.cgi*](https://maven.apache.org/download.cgi) | 基于项目对象模型（POM）的构建自动化工具 |
| Quarkus | [*https://quarkus.io*](https://quarkus.io) | 一种优化 Java 用于容器的 Kubernetes 原生堆栈 |
| Docker | [*https://www.docker.com/get-started*](https://www.docker.com/get-started) | 容器创建和执行 |
| Kubernetes | [*https://kubernetes.io*](https://kubernetes.io) | 一个容器编排平台，也被称为 K8s |
| minikube | [*https://minikube.sigs.k8s.io/docs/start*](https://minikube.sigs.k8s.io/docs/start) | Kubernetes 的本地分发 |
| GraalVM | [*https://www.graalvm.org*](https://www.graalvm.org) | 提供多种工具，包括从 Java 代码创建本机可执行文件的编译器 |
| Node.js | [*https://nodejs.org/en*](https://nodejs.org/en) | 一个 JavaScript 运行时引擎 |

# 本书中使用的约定

本书使用以下排版约定：

*斜体*

指示新术语、网址、电子邮件地址、文件名和文件扩展名。

`常量宽度`

用于程序列表，以及段落内引用程序元素，如变量或函数名称、数据库、数据类型、环境变量、语句和关键字。

**`常量宽度粗体`**

显示用户应该逐字输入的命令或其他文本。

*`常量宽度斜体`*

显示应由用户提供的值或由上下文确定的值替换的文本。

###### 提示

此元素表示提示或建议。

###### 注意

此元素表示一般注释。

###### 警告

此元素表示警告或注意事项。


# 致谢

写一本书从未是一件容易的事。这是一项漫长而艰巨的任务，耗费了大量精力，也消耗了大量家庭时间。因此，我们首先感谢在这场马拉松中支持我们的家人。

我们也非常感激能与红帽公司的出色人才合作。在这段旅程中，无数人给予了我们帮助；不可能一一列举。特别感谢 Georgios Andrianakis、Roberto Cortez、Stuart Douglas、Stéphane Epardaud、Jason Greene、Sanne Grinovero、Gavin King、Martin Kouba、Julien Ponge、Erin Schnabel、Guillaume Smet、Michal Szynkiewicz、Ladislav Thon 和 Julien Viet。他们的工作不仅非常出色，而且令人惊叹。能与这些顶尖开发者一同工作，对我们来说是一种特权。

最后，我们感谢所有提供了出色和建设性反馈的审阅者们：Mary Grygleski、Adam Bellemare、Antonio Goncalves、Mark Little、Scott Morrison、Nick Keune 和 Chris Mayfield。
