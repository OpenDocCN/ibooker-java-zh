# 前置材料

## 前言

当卡塔林请我写这篇前言时，我意识到在我的 17 年职业生涯中，我大部分时间都在处理这本书中讨论和解决的问题。从历史上看，我们已经到了一个大多数数据都保存在关系型数据库管理系统中的状态。这个任务听起来可能相当简单；将数据保存在数据库中，读取它，如果需要就修改它，最后删除它。许多（甚至高级）开发者没有意识到在这些几个操作中有多少计算机科学。用 Java 这样的面向对象语言与关系型数据库交谈，就像与来自另一个世界、完全遵循不同规则的人交谈一样。

在我的职业生涯早期，我大部分时间都在做将“结果集”映射到 Java 对象的工作，没有任何复杂的逻辑。这并不难，但确实很耗时。我只是在做梦，想着我们的架构师不会突然改变对象结构，这样我就不得不从头开始重写一切。而且，我并不是唯一一个这样想的人！

为了节省手动工作并自动化这些翻译任务，创建了像 Hibernate 这样的框架，后来还有 Spring Data。它们确实为你做了很多工作。你只需要将它们作为依赖项添加，在你的代码中添加一些注解，魔法就会发生！这在小型项目中工作得很好，但在现实生活中，项目要大得多，有很多边缘情况！

Hibernate 和 Spring Data 有着相当长的历史，投入了巨大的努力来实现这一魔法。在这本书中，你会发现每个框架的功能、它们的边缘情况、建议的优化和最佳实践的定义性描述。

这本书的流程设计得如此之好，以至于你首先理解关系型数据库的基本理论和对象/关系映射（ORM）的主要问题。然后，你会看到如何用 Hibernate 解决这个问题，以及如何在 Spring Data 为 Spring 框架宇宙扩展功能。最后，你将了解 ORM 在 NoSQL 解决方案中的应用。

我可以说，这些技术无处不在！字面意义上无处不在！无论是开设银行账户、购买机票、向政府发送请求，还是在博客文章上留言，幕布后面，有很大概率 Hibernate 和/或 Spring Data 正在处理这些应用程序的持久化！这些技术很重要，这本书提供了关于它们各种应用的信息。

了解你的工具对于正确完成工作至关重要。在这本书中，你将找到所有你需要的工作，有效地使用 Hibernate 和 Spring Data，这些工作都基于计算机科学的理论。对于所有 Java 开发者来说，这是一本必读之书，尤其是那些在企业技术领域工作的开发者。

——德米特里·亚历山德罗夫

Oracle 软件工程师，Java 冠军

保加利亚 Java 用户组联合负责人，《Helidon in Action》作者

数据持久性是任何应用程序的关键部分，数据库无疑是现代企业的核心。虽然像 Java 这样的编程语言提供了面向对象的业务实体视图，但这些实体背后的数据通常是关系型的。正是这个挑战——连接关系型数据和 Java 对象——Hibernate 和 Spring Data 通过对象/关系映射（ORM）来承担。

正如 Cătălin 在这本书中展示的那样，在除了最简单的企业环境之外，ORM 技术的有效使用需要理解和配置关系型数据和对象之间的中介。这要求开发者对应用程序及其数据需求、SQL 查询语言、关系型存储结构以及优化潜力有深入了解。

本书全面概述了使用行业领先的工具 Spring Data 和 Hibernate 进行 Java 持久性的方法。它涵盖了如何使用它们的类型映射能力和建模关联和继承的设施；如何通过 Querydsl 查询 JPA 来高效检索对象；如何使用 Spring Data 和 Hibernate 处理和管理事务；如何创建获取计划、策略和配置文件；如何过滤数据；如何配置 Hibernate 以在受管理和不受管理环境中使用；以及如何使用它们的命令。此外，你还将了解如何构建 Spring Data REST 项目，使用非关系型数据库进行 Java 持久性，以及测试 Java 持久性应用程序。在整本书中，作者提供了对 ORM 底层问题的见解以及 Hibernate 背后的设计选择。这些见解将使读者对 ORM 作为企业技术的有效使用有深入的理解。

*使用 Spring Data 和 Hibernate 进行 Java 持久性*是这些流行工具持久性的权威指南。你将受益于对 Spring Data JPA、Spring Data JDBC、Spring Data REST、JPA 和 Hibernate 的详细覆盖，比较和对比替代方案，以便能够为当今的企业计算选择最适合你代码的方案。

由于两个原因，我有幸推荐这本书。首先，我与作者分享了一个希望，那就是它将帮助你生产出越来越高效、安全、可测试的软件，其质量是其他人可以自信依赖的。其次，我认识作者本人，他在个人和技术层面都非常出色。他在软件开发行业有丰富的经验，他的专业活动，包括视频、书籍和文章，都是面向全球开发者社区受益的。

——穆罕默德·塔曼

首席解决方案架构师，Nortal

Java 冠军，Oracle ACE，JCP 成员

## 前言

我很幸运，在 IT 行业已经工作了超过 25 年。我在学生时代和职业生涯的初期就开始了 C++和 Delphi 编程。我从青少年时期的数学背景转向计算机科学，并一直努力将这两方面都保持在心中。

2000 年，我的注意力第一次转向 Java 编程语言。当时它非常新，但许多人都在预测它有一个光明的未来。我是网络游戏开发团队的一员，我们当时使用的技术是 applets，这在那些年是非常流行的。在应用程序背后，程序需要访问数据库，我们的团队花了一些时间开发访问和与数据库交互的逻辑。当时还没有使用 ORM，但我们能够开发自己的库来与数据库交互，这塑造了 ORM 的初步想法。

自 2004 年以来，我超过 90%的时间都在使用 Java 进行工作。对我来说，这是一个新时代的开始，像代码重构、单元测试和对象/关系映射这样的东西在我们的专业生活中变得越来越正常。

目前，有很多 Java 程序访问数据库，并依赖于高级技术和服务，如 JPA、Hibernate 和 Spring Data。使用 JDBC 的旧方法几乎已经不被记住。作为 Luxoft 的 Java 和 Web 技术专家以及 Java 章节负责人，我的一个活动就是开展关于 Java 持久化主题的课程，并指导我的同事关于这个话题。

我在 2020 年为 Manning Publications 写了我的第一本书，《JUnit in Action》，并且我很幸运能继续与他们合作。这本书的前几版主要关注 Hibernate，而如今 Spring 和 Spring Data 在 Java 程序中扮演着越来越重要的角色。当然，我们也专门用一章来介绍测试 Java 持久化应用程序。

对象/关系映射和 Java 持久化技术自从它们早期以来以及我开始使用它们以来已经取得了长足的进步。这些概念需要仔细的考虑和规划，因为特定的应用程序和技术需要特定的知识和方法。这本书有效地提供了这些信息，并附带了大量的示例。你还会找到简洁且易于遵循的程序来解决一系列任务。我希望这里概述的方法能帮助你决定在面临工作中新的情况时应该做什么。

## 致谢

首先，我想感谢我的教授和同事多年来给予的所有支持，以及参加我面对面和在线课程的所有参与者——他们激励我追求高质量的工作，并始终寻求改进。

我还想感谢 Luxoft，我在那里活跃了近 20 年，并且目前作为 Java 和 Web 技术专家以及 Java 章节负责人在那里工作。

感谢 Christian Bauer、Gavin King 和 Gary Gregory，他们是《Java Persistence with Hibernate》的共同作者，这本书为本书奠定了坚实的基础。希望有一天能亲自见到你们所有人。

向 Luxoft 的同事们表示最诚挚的感谢，Vladimir Sonkin 和 Oleksii Kvitsynskyi，我们共同研究新技术，开发 Java 课程和 Java 章节内容。在历史的长河中，能够如此有效地与俄罗斯和乌克兰工程师合作，是一个难得的机会。

我还想感谢 Manning 的员工：收购编辑 Mike Stephens、开发编辑 Katie Sposato Johnson 和 Christina Taylor、技术校对员 Jean-François Morin、校对员 Andy Carroll 以及幕后的制作团队。Manning 团队帮助我创作了一本高水平的书籍，我期待着更多这样的机会。

我很高兴两位杰出的专家，Dmitry Aleksandrov 和 Mohamed Taman，欣赏这本书并为其撰写了序言——与技术问题共同分析总是令人愉快的。

致所有审稿人：Amrah Umudlu、Andres Sacco、Bernhard Schuhmann、Bhagvan Kommadi、Damián Mazzini、Daniel Carl、Abayomi Otebolaku、Fernando Bernardino、Greg Gendron、Hilde Van Gysel、Jan van Nimwegen、Javid Asgarov、Kim Gabrielsen、Kim Kjærsulf、Marcus Geselle、Matt Deimel、Mladen Knezic、Najeeb Arif、Nathan B. Crocker、Özay Duman、Piotr Gliźniewicz、Rajinder Yadav、Richard Meinsen、Sergiy Pylypets、Steve Prior、Yago Rubio、Yogesh Shetty 和 Zorodzayi Mukuy——你们的建议帮助使这本书变得更好。

## 关于本书

*Java Persistence with Spring Data and Hibernate* 探讨了使用最流行的可用工具进行持久化。你将受益于对 Spring Data JPA、Spring Data JDBC、Spring Data REST、JPA 和 Hibernate 的详细覆盖，比较和对比了各种替代方案，以便你可以选择最适合你代码的方案。

我们将从对对象/关系映射（ORM）的实战介绍开始，然后深入探讨如何将对象与数据库连接的映射策略。你将了解 Hibernate 和 Spring Data 中事务的不同处理方法，甚至如何使用非关系型数据库实现 Java 持久化。最后，我们将探讨持久化应用程序的测试策略，以保持代码的清洁和错误最少。

### 适合阅读本书的人群

本书面向已经熟练编写 Java 核心代码的应用程序开发者，他们有兴趣学习如何开发能够轻松有效地与数据库交互的应用程序。你应该熟悉面向对象编程，并至少具备 Java 的工作知识。你还需要具备 Maven 的工作知识，能够构建 Maven 项目，在 IntelliJ IDEA 中打开 Java 程序，编辑它，并在执行中启动它。一些章节需要了解 Spring 或 REST 等技术的基本知识。

### 本书组织结构概述：路线图

本书共有 6 部分，20 章。第一部分将帮助你开始 ORM 之旅。

+   第一章介绍了对象/关系范式不匹配以及处理它的几种策略，首先是对象/关系映射（ORM）。

+   第二章通过 Jakarta Persistence API、Hibernate 和 Spring Data 教程逐步指导你，你将实现并测试一个“Hello World”示例。

+   第三章教你如何在 Java 中设计和实现复杂的企业领域模型，以及你有哪些可用的映射元数据选项。

+   第四章将提供 Spring Data JPA 的第一印象，介绍如何使用它及其功能。

第二部分是关于 ORM，从类和属性到表和列。

+   第五章从常规类和属性映射开始，解释了如何映射细粒度的 Java 领域模型。

+   第六章演示了如何映射基本属性和可嵌入组件，以及如何控制 Java 和 SQL 类型之间的映射。

+   第七章演示了如何使用四种基本的继承映射策略将实体的继承层次映射到数据库中。

+   第八章是关于映射集合和实体关联。

+   第九章深入探讨了高级实体关联映射，如一对一实体关联映射、一对多映射选项以及多对多和三元实体关系。

+   第十章讨论了数据管理，检查对象的生命周期和状态，以及如何有效地使用 Jakarta Persistence API。

第三部分是关于使用 Hibernate 和 Java Persistence 加载数据和存储数据。它介绍了编程接口，如何编写事务性应用程序，以及 Hibernate 如何最有效地从数据库加载数据。

+   第十一章定义了数据库和系统事务的基本要素，并解释了如何使用 Hibernate、JPA 和 Spring 控制并发访问。

+   第十二章探讨了延迟加载和即时加载、获取计划、策略和配置文件，并以优化 SQL 执行讨论结束。

+   第十三章涵盖了级联状态转换、监听和拦截事件、使用 Hibernate Envers 进行审计和版本控制，以及动态过滤数据。

第四部分将 Java 持久化与目前最广泛使用的 Java 框架：Spring 连接起来。

+   第十四章教你创建 JPA 或 Hibernate 应用程序的最重要策略，以及如何将其与 Spring 集成。

+   第十五章介绍了使用大型 Spring Data 框架的另一部分：Spring Data JDBC 开发持久化应用程序的可能性，并进行了分析。

+   第十六章探讨了 Spring Data REST，你可以使用它来构建表示状态转换（REST）架构风格的应用程序。

第五部分将 Java 应用程序连接到常用的 NoSQL 数据库：MongoDB 和 Neo4j。

+   第十七章介绍了 Spring Data MongoDB 框架最重要的特性，并将其与已讨论的 Spring Data JPA 和 Spring Data JDBC 进行比较。

+   第十八章介绍了 Hibernate OGM 框架，并演示了如何使用 JPA 代码连接到不同的 NoSQL 数据库（MongoDB 和 Neo4j）。

第六部分教您如何编写查询以及如何测试 Java 持久化应用程序。

+   第十九章讨论了使用 Querydsl 进行工作，这是使用 Java 程序查询数据库的替代方案之一。

+   第二十章探讨了如何测试 Java 持久化应用程序，介绍了测试金字塔，并分析了在特定环境下的持久化测试。

### 关于代码

本书包含（主要是）大块代码，而不是短小的代码片段。因此，所有代码列表都有注释和解释。

您可以通过从 GitHub 下载获取所有这些示例的完整源代码[`github.com/ctudose/java-persistence-spring-data-hibernate`](https://github.com/ctudose/java-persistence-spring-data-hibernate)。您还可以从本书的 liveBook（在线）版本获取可执行的代码片段[`livebook.manning.com/book/java-persistence-with-spring-data-and-hibernate`](https://livebook.manning.com/book/java-persistence-with-spring-data-and-hibernate)。本书中示例的完整代码可在 Manning 网站[`www.manning.com/books/java-persistence-with-spring-data-and-hibernate`](https://www.manning.com/books/java-persistence-with-spring-data-and-hibernate)上下载。

### liveBook 讨论论坛

购买*使用 Spring Data 和 Hibernate 进行 Java 持久化*包括免费访问 liveBook，Manning 的在线阅读平台。使用 liveBook 的独特讨论功能，您可以在全球范围内或针对特定章节或段落添加评论。为自己做笔记、提问和回答技术问题，以及从作者和其他用户那里获得帮助都非常简单。要访问论坛，请访问[`livebook.manning.com/book/java-persistence-with-spring-data-and-hibernate/discussion`](https://livebook.manning.com/book/java-persistence-with-spring-data-and-hibernate/discussion)。您还可以在[`livebook.manning.com/discussion`](https://livebook.manning.com/discussion)了解更多关于 Manning 论坛和行为准则的信息。

Manning 对我们读者的承诺是提供一个场所，让读者之间以及读者与作者之间可以进行有意义的对话。这不是对作者参与特定数量活动的承诺，作者对论坛的贡献仍然是自愿的（且未付费）。我们建议您尝试向作者提出一些挑战性的问题，以免他的兴趣转移！只要本书有售，论坛和以前讨论的存档将可通过出版社的网站访问。

## 关于作者

![Tudose](img/Tudose.png)

Cătălin Tudose 出生于罗马尼亚的 Piteşti，Argeş，1997 年在布加勒斯特获得计算机科学学位。他还拥有这个领域的博士学位。他在 Java 领域拥有超过 20 年的经验，目前担任 Luxoft Romania 的 Java 和网络技术专家。作为布加勒斯特自动化与计算机学院的助教和教授，他教授了超过 2,000 小时的课程和应用。Cătălin 还在公司内部教授了超过 3,000 小时的 Java 课程，包括企业初级项目，该项目在波兰培养了大约 50 名新的 Java 程序员。他在 UMUC（马里兰大学大学学院）教授了在线课程：使用 Java 的计算机图形学（CMSC 405）、Java 中级编程（CMIS 242）和 Java 高级编程（CMIS 440）。Cătălin 为 Pluralsight 开发了六个与 JUnit 5、Spring 和 Hibernate 相关的课程：“使用 JUnit 5 进行 TDD”、“Java BDD 基础”、“在 Java 中实现测试金字塔策略”、“Spring 框架：使用 Spring AOP 的面向方面编程”、“从 JUnit 4 迁移到 JUnit 5 测试平台”和“Java Persistence with Hibernate 5 基础”。除了 IT 领域和数学，Cătălin 对世界文化和足球也感兴趣。他是 FC Argeş Piteşti 的终身支持者。

## 《Java Persistence with Hibernate，第二版》的作者

Christian Bauer 是 Hibernate 开发团队的成员；他担任培训师和顾问。

Gavin King 是 Hibernate 的创造者，并在 Red Hat 担任杰出工程师。他帮助设计了 JPA 和 EJB 3，并担任了 CDI 规范的规范负责人和作者。他最近参与了 Hibernate 6 和 Hibernate Reactive 的工作，并参与了 Quarkus 设计的咨询。Gavin 在世界各地的数百个会议和 Java 用户组上进行了演讲。

Gary Gregory 是 Rocket Software 的首席软件工程师，负责应用程序服务器和遗留集成。他是 Manning 出版的 *JUnit in Action* 和 *Spring Batch in Action* 的合著者，并担任 Apache 软件基金会项目（Commons、HttpComponents、Logging Services 和 Xalan）的项目管理委员会成员。

## 关于封面插图

《Java Persistence with Spring Data and Hibernate》封面上的图像是“Homme Maltois”，或“马耳他人”，取自 Jacques Grasset de Saint-Sauveur 的作品集，该作品集于 1788 年出版。每一幅插图都是手工精心绘制和着色的。

在那些日子里，人们通过他们的服装很容易就能识别出他们住在哪里，以及他们的职业或社会地位。Manning 通过基于几个世纪前丰富的地方文化多样性的书封面，以及像这样的作品集中的图片，庆祝计算机行业的创新和主动性。
