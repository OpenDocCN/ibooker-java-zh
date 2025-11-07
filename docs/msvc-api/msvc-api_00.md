# 前置内容

## 前言

API 和微服务已经席卷了软件行业。在软件复杂性不断增加和需要扩展的压力下，越来越多的组织正在从单体架构迁移到微服务架构。O'Reilly 的“2020 年微服务采用”报告发现，77%的受访者已经采用了微服务，预计这一趋势在未来几年将继续增长。

使用微服务带来了通过 API 驱动服务集成的挑战。根据 Nordic APIs 的数据，90%的开发者使用 API，他们花费 30%的时间在构建 API 上。¹ API 经济的增长已经改变了我们构建应用程序的方式。如今，越来越多地构建完全通过 API 交付的产品和服务，例如 Twilio 和 Stripe。即使是传统行业如银行和保险业，也在通过开放 API 和集成到 Open Banking 生态系统中找到新的业务线。API-first 产品的广泛可用性意味着我们可以在构建自己的应用程序时专注于核心业务能力，同时使用外部 API 来处理诸如用户认证和发送电子邮件等常见任务。

成为这个不断发展的生态系统的成员令人兴奋。然而，在我们拥抱微服务和 API 之前，我们需要知道如何架构微服务，如何设计 API，如何制定 API 策略，如何确保我们提供可靠的集成，如何选择部署模式，以及如何保护我们的系统。根据我的经验，大多数组织在这些问题上都会遇到一个或多个挑战，而 IBM 最近的一份报告发现，31%的企业没有采用微服务，原因是缺乏内部专业知识。² 同样，Postman 的 2022 年 API 状态报告发现，14%的受访者有 11%–25%的时间遇到 API 集成失败，根据 Salt Security 的数据，2022 年 94%的组织经历了 API 安全事件。³

许多书籍都讨论了上一段提到的那些问题，但它们通常从非常具体的视角来处理：有些专注于架构，有些关注 API，还有些关注安全。我感觉缺少一本能够将这些疑问汇集在一起，并以实用方法来解答的书：本质上，一本能让普通开发者快速上手，掌握设计构建微服务 API 的最佳实践、原则和模式的书。我正是带着这个目标来写这本书的。

在过去的几年里，我有机会与不同的客户合作，帮助他们设计微服务并交付 API 集成。在这些项目中工作让我对开发团队在处理微服务和 API 时面临的主要挑战有了更深入的了解。事实证明，这两种技术都看似简单。一个设计良好的 API 易于导航和消费，而良好的微服务架构可以提升开发者的生产力，并且易于扩展。另一方面，设计不良的 API 容易出错且难以使用，而设计不良的微服务会导致所谓的分布式单体。

显而易见的问题随之而来：你如何设计好的 API？你如何设计松散耦合的微服务？这本书将帮助你回答这些问题以及更多。你还将有机会亲手构建 API 和服务，并学习如何确保它们的安全、测试和部署。我在这本书中教授的方法、模式和原则是多年试验和实验的结果，我非常期待与你们分享它们。我希望你们能在这本书中找到成为更好的软件开发者和架构师旅程中的宝贵资源。

## 致谢

写这本书是我职业生涯中最迷人的旅程之一，没有家人和一支了不起的同事团队的帮助和支持，我无法完成它。这本书献给我的妻子 Jiwon，没有她不断的鼓励和理解，我无法完成这本书；也献给我们的小女儿 Ivy，她确保我在日程中从未有过无聊的时刻。

我从为这本书贡献想法的人中受益匪浅，他们帮助我更好地理解了我在其中使用的工具和协议，并对各个章节和草稿提供了反馈。特别感谢 Dmitry Dygalo、Kelvin Meeks、Sebastián Ramírez Montaño、Chris Richardson、Jean Yang、Gajendra Deshpande、Oscar Islas、Mehdi Medjaoui、Ben Hutton、Andrej Baranovskij、Alex Mystridis、Roope Hakulinen、Steve Ardagh-Walter、Kathrin Björkelund、Thomas Dean、Marco Antonio Sanz、Vincent Vandenborne 以及 Mirumee 的 Ariadne 项目的出色维护者。

自 2020 年以来，我在各种会议上展示了这本书的草稿和想法，包括 EuroPython、PyCon India、API World、API 规格会议以及各种播客和聚会。我想感谢所有参加我的演讲并给予宝贵反馈的人。我还想感谢参加 microapis.io 上我研讨会的人，他们对这本书的深思熟虑的评论。

我想感谢我的收购编辑 Andy Waldron。Andy 在帮助我把我的书稿提案整理得很好，并确保这本书聚焦于相关主题方面做得非常出色。他还不知疲倦地支持我推广这本书，并帮助我触及更广泛的受众。

您现在手中的这本书之所以易于阅读和理解，多亏了我的编辑 Marina Michaels 的无价工作。她不遗余力地帮助我写出更好的书籍。她出色地帮助我改进写作风格，并保持我保持方向和动力。

我要感谢我的技术编辑 Nick Watts，他正确地指出了许多不准确之处，并总是挑战我提供更好的解释和插图，以及我的技术校对员 Al Krinker，他勤奋地检查了所有代码列表和这本书的 GitHub 仓库，确保代码正确无误且无执行问题。

我还要感谢参与这本书生产的 Manning 团队的其他成员，包括 Candace Gillhoolley、Gloria Lukos、Stjepan Jureković、Christopher Kaufmann、Radmila Ercegovac、Mihaela Batinić、Ana Romac、Aira Dučić、Melissa Ice、Eleonor Gardner、Breckyn Ely、Paul Wells、Andy Marinkovich、Katie Tennant、Michele Mitchell、Sam Wood、Paul Spratley、Nick Nason 和 Rebecca Rinehart。也要感谢 Marjan Bace，他对我下注，给了这本书一个机会。

在撰写这本书的过程中，我有机会从最令人惊叹的审稿人团队那里获得详细而出色的反馈，包括 Alain Lompo、Björn Neuhaus、Bryan Miller、Clifford Thurber、David Paccoud、Debmalya Jash、Gaurav Sood、George Haines、Glenn Leo Swonk、Hartmut Palm、Ikechukwu Okonkwo、Jan Pieter Herweijer、Joey Smith、Juan Jimenez、Justin Baur、Krzysztof Kamyczek、Manish Jain、Marcus Young、Mathijs Affourtit、Matthieu Evrin、Michael Bright、Michael Rybintsev、Michal Rutka、Miguel Montalvo、Ninoslav Cerkez、Pierre-Michel Ansel、Rafael Aiquel、Robert Kulagowski、Rodney Weis、Sambasiva Andaluri、Satej Kumar Sahu、Simeon Leyzerzon、Steven K Makunzva、Stuart Woodward、Stuti Verma 和 William Jamir Silva。我感谢他们为书中内容的质量做出了巨大贡献。

自从这本书进入 MEAP 以来，我收到了许多读者通过各种渠道（如 LinkedIn 和 Twitter）发送的鼓励和反馈，这让我感到非常幸运。我还很幸运能与一群才华横溢的读者进行交流，他们积极参与了 Manning 的 liveBook 平台上的书籍论坛。我对你们所有人都心怀感激。

没有数千名开源贡献者的不懈努力，这本书是不可能完成的。他们创建了并维护了我在这本书中使用的令人惊叹的库。我对你们所有人都非常感激，并希望我的书能帮助使你们的工作更加突出。

最后，感谢您，读者，购买了这本书的一本。我只希望您觉得这本书有用且信息丰富，并且您阅读这本书的乐趣能和我写作这本书的乐趣一样。我喜欢收到读者的来信，如果您愿意与我分享对这本书的看法，我将非常高兴。

## 关于这本书

本书的目标是教你如何使用 API 构建微服务并推动它们的集成。你将学习如何设计微服务平台，构建 REST 和 GraphQL API 以实现微服务之间的通信。你还将学习测试和验证你的微服务 API，确保它们的安全，并在云中部署和运行它们。

### 适合阅读本书的人群？

本书对使用微服务和 API 的软件开发者很有帮助。本书采用了一种非常实用的方法，几乎每一章都通过完整的编码示例来阐述解释。因此，直接与微服务 API 工作的实践开发者会发现本书的内容很有价值。

编码示例使用 Python 编写；然而，了解该语言并不是能够跟随示例的必要条件。在引入新代码之前，每个概念都得到了彻底的解释。

本书对设计策略、最佳实践和开发工作流程给予了大量关注，因此它对需要决定微服务是否是合适的架构解决方案的 CTO、架构师和工程副总裁也很有用，或者需要在不同 API 策略之间进行选择以及如何使集成工作。

### 本书是如何组织的：一个路线图

本书分为四个部分，共 14 章。

第一部分介绍了微服务和 API 的概念，展示了如何构建一个简单的 API，并解释了如何设计微服务平台：

+   第一章介绍了本书的主要概念：微服务和 API。它解释了微服务与单体架构的区别，以及何时使用单体与微服务更合适。它还解释了 API 是什么以及它们如何帮助我们推动微服务之间的集成。

+   第二章提供了使用 Python 流行的 FastAPI 框架实现 API 的逐步指南。你将学习如何阅读 API 规范并理解其要求。你还将学习逐步构建 API，以及如何测试你的数据验证模型。

+   第三章解释了如何设计微服务平台。它介绍了三个基本的微服务设计原则，并解释了如何通过业务能力和子域分解来将系统分解为微服务。

第二部分解释了如何设计、记录和构建 REST API，以及如何构建微服务：

+   第四章解释了 REST API 的设计原则。它介绍了 REST 架构的六个约束和 Richardson 成熟度模型，然后继续解释我们如何利用 HTTP 协议来设计结构良好且高度表达的 REST API。

+   第五章解释了如何使用 OpenAPI 规范标准来记录 REST API。你将学习 JSON Schema 语法的基础知识，如何定义端点，如何建模你的数据，以及如何使用可重用模式重构你的文档。

+   第六章解释了如何使用两个流行的 Python 框架：FastAPI 和 Flask 来构建 REST API。您将了解这两个框架之间的区别，但您也将了解构建 API 的原则和模式保持相同，并且超越了任何技术栈的实现细节。

+   第七章解释了构建微服务的基本原则和模式。它介绍了六边形架构的概念，并解释了如何强制执行应用程序层之间的松耦合。它还解释了如何使用 SQLAlchemy 实现数据库模型以及如何使用 Alembic 管理数据库迁移。

第三部分解释了如何设计、消费和构建 GraphQL API：

+   第八章解释了如何设计 GraphQL API 以及 Schema 定义语言的工作原理。它介绍了 GraphQL 的内置类型，并解释了如何定义自定义类型。您将学习如何创建类型之间的关系，以及如何定义查询和突变。

+   第九章解释了如何消费 GraphQL API。您将学习如何运行模拟服务器以及如何使用 GraphiQL 探索 GraphQL 文档。您将学习如何对 GraphQL 服务器运行查询和突变以及如何参数化您的操作。

+   第十章解释了如何使用 Python 的 Ariadne 框架构建 GraphQL API。您将学习如何利用 API 文档自动加载数据验证模型，以及如何实现自定义类型、查询和突变的解析器。

第四部分解释了如何测试、安全和部署您的微服务 API：

+   第十一章解释了如何使用标准协议，如 OpenID Connect (OIDC)和 Open Authorization (OAuth) 2.1，向您的 API 添加身份验证和授权。您将学习如何生成和验证 JSON Web Tokens (JWTs)以及如何为您的 API 创建授权中间件。

+   第十二章解释了如何测试和验证您的 API。您将了解基于属性的测试是什么以及如何使用它来测试您的 API，您还将学习如何使用 Dredd 和 schemathesis 等 API 测试自动化框架。

+   第十三章解释了如何将您的微服务 API Docker 化，如何使用 Docker Compose 在本地运行它们，以及如何将您的 Docker 构建发布到 AWS 弹性容器注册库（ECR）。

+   第十四章解释了如何使用 Kubernetes 将您的微服务 API 部署到 AWS。您将学习如何使用 AWS EKS 创建和操作 Kubernetes 集群，如何将 Aurora 无服务器数据库启动到安全网络中，如何使用信封加密安全地注入应用程序配置，以及如何设置您的服务以进行大规模操作。

所有章节都有一个共同的主题：构建一个虚构的、按需咖啡配送平台 CoffeeMesh 的组件。我们在第一章介绍了 CoffeeMesh，在第三章，我们将平台分解为微服务。因此，我建议阅读第一章和第三章，以便更好地理解后续章节中介绍的示例。否则，本书的每一部分都是相当独立的，每个章节都相当自包含。例如，如果你想学习如何设计和构建 REST API，可以直接跳转到第二部分，如果你想关注 GraphQL API，可以专注于第三部分。同样，如果你想学习如何给你的 API 添加身份验证和授权，可以直接跳转到第十一章，或者如果你想学习如何测试 API，可以直接跳转到第十二章。

之间有一些章节之间的交叉引用：例如，第十二章引用了第二部分和第三部分的 API 实现，但如果你熟悉构建 API，你应该可以直接跳转到第十二章。第四部分的其他章节也是如此。

### 关于代码

这本书包含了大量的源代码示例，既有编号列表中的，也有与普通文本混排的。在两种情况下，源代码都以`固定宽度字体`的形式呈现，以便与普通文本区分开来。有时代码也会被**`加粗`**，以突出显示章节中从先前步骤中更改的代码，例如当新功能添加到现有代码行时。

在许多情况下，原始源代码已经被重新格式化；我们添加了换行并重新调整了缩进，以适应书籍中的可用页面空间。在某些情况下，即使这样也不够，列表中还包括了行续接标记（➥）。此外，当代码在文本中描述时，源代码中的注释通常也会从列表中移除。许多列表旁边都有代码注释，突出显示重要概念。

除了第一章、第三章和第四章外，本书的每一章都充满了代码示例，这些示例展示了向读者介绍的所有新概念和模式。大多数代码示例都是用 Python 编写的，但在第五章、第八章和第九章中，重点在于 API 设计，因此包含 OpenAPI/JSON Schema（第五章）和 Schema 定义语言（第八章和第九章）的示例。所有代码都进行了彻底的解释，因此应该对所有读者都易于理解，包括那些不了解 Python 的读者。

您可以从本书的 liveBook（在线）版本中获取可执行的代码片段，请访问[`livebook.manning.com/book/microservice-apis`](https://livebook.manning.com/book/microservice-apis)。本书中示例的完整代码可以从 Manning 网站[www.manning.com](https://manning.com/books/microservice-apis)下载，也可以从专门为此书建立的 GitHub 仓库中下载：[`github.com/abunuwas/microservice-apis`](https://github.com/abunuwas/microservice-apis)。GitHub 仓库中的每个章节都有一个对应的文件夹，例如 ch02 对应第二章。除非另有说明，每个章节中所有文件引用都是相对于 GitHub 中相应文件夹的。例如，在第二章中，orders/app.py 指的是 GitHub 中的 ch02/orders/app.py 文件。

本书 GitHub 仓库显示了每个章节代码的最终状态。有些章节展示了如何逐步构建功能，以迭代步骤进行。在这些情况下，您在 GitHub 上找到的代码版本与章节中的最终代码版本相匹配。

本书中的 Python 代码示例已在 Python 3.10 上进行了测试，尽管任何高于 3.7 版本的 Python 都应该可以正常工作。我在整个书中使用的代码和命令已在 Mac 机器上进行了测试，但它们也应该在 Windows 和 Linux 上没有问题。如果您在 Windows 上工作，我建议您使用 POSIX 兼容的终端，例如 Cygwin。

我在每个章节中使用了 Pipenv 来管理依赖项。在每个章节的文件夹中，您将找到描述我用来运行代码示例的精确依赖项的 Pipfile 和 Pipfile.lock 文件。为了避免运行代码时出现问题，我建议您在每个章节的开始时下载这些文件，并从它们中安装依赖项。

### liveBook 讨论论坛

购买 *微服务 API* 包含对 Manning 在线阅读平台 liveBook 的免费访问。使用 liveBook 的独家讨论功能，您可以在全球范围内或特定章节或段落中添加评论。为自己做笔记、提问和回答技术问题以及从作者和其他用户那里获得帮助都非常简单。要访问论坛，请访问[`livebook.manning.com/book/microservice-apis/discussion`](https://livebook.manning.com/book/microservice-apis/discussion)。您还可以在[`livebook.manning.com/discussion`](https://livebook.manning.com/discussion)了解更多关于 Manning 论坛和行为准则的信息。

曼宁对读者的承诺是提供一个场所，在那里个人读者之间以及读者与作者之间可以进行有意义的对话。这不是对作者参与特定数量活动的承诺，作者对论坛的贡献仍然是自愿的（且未付费）。我们建议你尝试向他提出一些挑战性的问题，以免他的兴趣转移！只要本书有售，论坛和先前讨论的存档将可通过出版社的网站访问。

### 其他在线资源

如果你想了解更多关于微服务 API 的信息，你可以查看我的博客，[`microapis.io/blog`](https://microapis.io/blog)，其中包含补充本书课程内容的额外资源。在同一网站上，我还保持着一个经常更新的、我组织的研讨会和研讨会的最新列表，这些活动也补充了本书的内容。

## 关于作者

![](img/fm_Peralta.png)

何塞·哈罗·佩拉尔塔（José Haro Peralta）是一位软件和架构顾问。拥有超过 10 年的经验，何塞帮助大小组织构建复杂系统，设计微服务平台，并交付 API 集成。他也是 microapis.io 公司的创始人，该公司提供软件咨询和培训服务。何塞在云计算、DevOps 和软件自动化领域被认为是思想领袖，他经常在国际会议上发表演讲，并经常组织公开研讨会和研讨会。

## 关于封面插图

《微服务 API》封面上的插图标题为“L’invalide”，或“残疾人”，描绘了一位受伤的法国士兵，他曾是国家残疾人之家（Hôtel national des Invalides）的居民。这幅图像取自雅克·格拉塞·德·圣索沃尔（Jacques Grasset de Saint-Sauveur）的收藏，该收藏于 1797 年出版。每一幅插图都是手工精心绘制和着色的。

在那些日子里，人们的生活地点和他们的职业或社会地位很容易通过他们的着装来识别。曼宁通过基于几个世纪前丰富多样的地区文化的封面，以及此类收藏中的图片，来庆祝计算机行业的创新精神和主动性。

* * *

¹ J. Simpson，"20 个令人印象深刻的 API 经济统计数据" ([`nordicapis.com/20-impressive-api-economy-statistics/`](https://nordicapis.com/20-impressive-api-economy-statistics/) [访问日期：2022 年 5 月 26 日])。

² "企业中的微服务，2021 年：实际效益，值得挑战"，([`www.ibm.com/downloads/cas/OQG4AJAM`](https://www.ibm.com/downloads/cas/OQG4AJAM) [访问日期：2022 年 5 月 26 日])。

³ Salt Security，"API 安全状态 Q3 2022"，第 4 页 ([`content.salt.security/state-api-report.html`](https://content.salt.security/state-api-report.html))。
