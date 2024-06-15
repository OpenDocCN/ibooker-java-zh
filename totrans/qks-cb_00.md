# 序言

我们很高兴你能加入我们一起学习和使用 Quarkus 的旅程！与传统的 Java 框架不同，它可能庞大、笨重、复杂，需要数月才能掌握，Quarkus 则建立在你已有的知识基础之上！它使用了 JPA、JAX-RS、Eclipse Vert.x、Eclipse MicroProfile 和 CDI 等多种你已经熟悉的技术。然后，Quarkus 将你的知识整合成一个紧凑、易于部署、完全优化于 Kubernetes 的容器，可选择使用 OpenJDK Hotspot 或 GraalVM。这使得你能够尽可能紧密地打包你的 Kubernetes 集群，充分利用每台机器上的资源，以应对需求的扩展。无论你在迁移到 Kubernetes 的哪个阶段，你都会在 Quarkus 中找到有用的东西，而本书将为你提供成功所需的工具和资源。

# 谁应该阅读本书

显然，我们希望每个人都能阅读这本书！但是，我们对读者做了一些假设：

+   你已经熟悉 Java 和在该领域内的应用程序开发。

+   你理解传统软件开发实践。

+   你经常将服务部署到一组机器的集群或云中。

# 为什么要写这本书

Quarkus 是一个相对较新的框架，处于一个新的领域（本地 Java 和 GraalVM）。我们希望深入一些示例和操作指南，超出互联网上的内容。此外，我们希望本书尽可能地充实内容。这本书中没有需要理解或记住的大型应用程序。本书中的所有示例都是独立的，可以随时使用。我们希望你将其作为你所有 Quarkus 开发的参考！

# 导航本书

章节的组织比较松散，但基本上如下：

+   第一章和第二章将带你了解 Quarkus，并设置你的基本项目。

+   第三章至第六章介绍了 Quarkus 的主要部分：使用 CDI 和 Eclipse MicroProfile 构建的 RESTful 应用程序。这些章节还向你展示了如何打包你的应用程序。

+   第七章至第十四章涉及更难，但同样重要的概念，如容错性、持久化、安全性以及与其他服务的交互。你还将了解 Quarkus 与 Kubernetes 的其他集成。

+   第十五章和第十六章讨论了使用 Quarkus 进行响应式编程以及框架的一些额外功能，如模板化、调度和 OpenAPI。

# 本书中使用的约定

本书中使用以下排印约定：

*Italic*

表示新术语、网址、电子邮件地址、文件名和文件扩展名。

`Constant width`

用于程序清单，以及在段落内引用程序元素，例如变量或函数名、数据库、数据类型、环境变量、语句和关键字。

**`Constant width bold`**

显示用户应该按字面意义键入的命令或其他文本。

*`Constant width italic`*

显示应由用户提供的值或由上下文确定的值的文本。

###### Tip

此元素表示提示或建议。

###### Note

此元素表示一般注释。

###### Warning

此元素指示警告或注意事项。

###### Important

此元素表示需要记住的重要事项。

# 使用代码示例

可以下载补充材料（代码示例、练习等）[*https://oreil.ly/quarkus-cookbook-code*](https://oreil.ly/quarkus-cookbook-code)。

如果您有技术问题或在使用代码示例时遇到问题，请发送电子邮件至*bookquestions@oreilly.com*。

本书旨在帮助您完成工作。一般情况下，如果本书提供示例代码，您可以在自己的程序和文档中使用它。除非您复制了大量代码，否则无需征得我们的许可。例如，编写使用本书中几个代码块的程序不需要许可。出售或分发来自 O’Reilly 书籍的示例则需要许可。引用本书并引用示例代码来回答问题不需要许可。将本书的大量示例代码合并到产品文档中则需要许可。

我们感谢您的使用，但通常不需要署名。署名通常包括标题、作者、出版商和 ISBN。例如：“*Quarkus Cookbook* by Alex Soto Bueno and Jason Porter (O’Reilly). Copyright 2020 Alex Soto Bueno and Jason Porter, 978-1-492-06265-3.”

如果您认为您使用的代码示例超出了合理使用范围或以上所给予的许可，请随时通过*permissions@oreilly.com*联系我们。

# O’Reilly Online Learning

###### Note

超过 40 年来，[*O’Reilly Media*](http://oreilly.com)提供技术和商业培训、知识和洞察，帮助企业取得成功。

我们独特的专家和创新者网络通过书籍、文章和我们的在线学习平台分享他们的知识和专业知识。O’Reilly 的在线学习平台为您提供按需访问的现场培训课程、深入学习路径、交互式编码环境以及来自 O’Reilly 和 200 多个其他出版商的大量文本和视频。有关更多信息，请访问[*http://oreilly.com*](http://oreilly.com)。

# 如何联系我们

请将关于本书的评论和问题发送至出版商：

+   O’Reilly Media, Inc.

+   1005 Gravenstein Highway North

+   Sebastopol, CA 95472

+   800-998-9938（美国或加拿大）

+   707-829-0515（国际或本地）

+   707-829-0104（传真）

我们为这本书创建了一个网页，列出勘误、示例和任何额外信息。您可以访问这个页面：[*https://oreil.ly/quarkus-cookbook*](https://oreil.ly/quarkus-cookbook)。

如果您想评论或询问关于这本书的技术问题，请发送电子邮件至*bookquestions@oreilly.com*。

有关我们的图书和课程的新闻和信息，请访问[*http://oreilly.com*](http://oreilly.com)。

在 Facebook 上找到我们：[*http://facebook.com/oreilly*](http://facebook.com/oreilly)

在 Twitter 上关注我们：[*http://twitter.com/oreillymedia*](http://twitter.com/oreillymedia)

观看我们的 YouTube 频道：[*http://youtube.com/oreillymedia*](http://youtube.com/oreillymedia)

# 致谢

**Jason Porter**：在隔离期间你会做什么？当然是写一本书！感谢所有在医疗前线勇敢的人们。我要感谢 Quarkus 和 GraalVM 团队，为我们提供了一个令人惊叹的工具和有趣的开发体验。我从事软件开发已有 20 多年，而 Quarkus 让我重新感受到了当初学习软件开发时的乐趣。特别感谢 Georgios Andrianakis 和 Daniel Hinojosa 为本书提供的技术审查！你们的工作帮助我们创造了一些易于理解、有用且希望能给学习 Quarkus 的人带来乐趣的东西。我还要感谢 Red Hat 让我有机会撰写这本书。Alex，再次感谢你邀请我与你共同撰写书籍！最后，感谢我的五个孩子（Kaili、Emily、Zackary、Nicolas 和 Rebecca）和妻子 Tessie，尽管我说过不再写书，但你们仍然支持我。爱你们！

**Alex Soto Bueno**：这本书是在 COVID-19 大流行期间完成的，因此首先，我要感谢所有为我们所有人提供医疗服务的医护人员。我还要感谢 Red Hat Developers 团队，特别是 Burr Sutter，让我有机会撰写这本书。Jason，一如既往地与你共同撰写书籍是一种乐趣。最后，感谢我的父母；我的妻子 Jessica；以及我的女儿 Ada 和 Alexandra，在我写书的时候对我的耐心，因为没有两全其美的事情。非常感谢你们一切。
