# 前置内容

## 前言

我多年来写了数十篇前言和序言，但这次可能是第一次我感到不得不写一篇前言感到烦恼。为什么？我是一个输不起的人！我感到难过，因为我没有写这本书。我感到难过，因为我怀疑我甚至*能否*写这本书。

本书*非常棒*。它的每一页都充满了基于明显、深刻经验的有价值想法。我一直想看到所有这些概念都放在一个地方，我将在可预见的未来向人们推荐这本书。

构建生产级别的应用程序，其中生产环境大多是 Kubernetes，*并且*构建生产环境本身？这无疑是一项艰巨的任务，就像这本书一样，篇幅超过 600 页！但别让我对这本书的篇幅描述让你却步。对于这样一个庞大的主题，这本书实在是大得惊人。

本书涵盖了常见的主题：如何构建服务和微服务，如何处理持久化、消息传递、为可观察性进行仪表化、配置和安全。此外，书中还专门用整章内容来介绍这些概念中的某些部分。

你的 Spring Boot 应用程序是本书雄心壮志的巅峰之作（一个店面系统，当然），但这绝不是本书关注的唯一事物。本书在关注点上既深入又广泛——这是一个惊人的成就！我想我应该列出一些具体内容，这样你就可以开始欣赏这本书的覆盖范围是多么细腻。在我这样做之前，请记住，你*必须*阅读这本书。以下列表绝非详尽无遗，但它包括了我发现自己对阅读内容感到震惊的东西。这些都是关于 Spring Boot 和 Spring Cloud 的书中应该包含的内容，但不幸的是，很少见。

+   对 Loki、FluentBit 和 Grafana 的日志处理非常出色。

+   本书不仅带你进入 Kubernetes 的“上手”阶段，而且带你深入到 Kubernetes 部署的各个方面。你将轻松使用 Knative 和 Spring Cloud Function 实现无服务器架构。接下来，你将使用 GitHub Actions、Kustomize 和 Kubeval 等工具构建管道。最后，你将使用 Tilt 和 Docker Compose 等工具在本地进行开发。

+   在单页应用（SPA）的背景下讨论安全可能是一本优秀的书籍。它是有能力的、迭代的、快速的，并且注重生产。*不要*错过这一章节。

+   所有的工作都是以测试为前提的。Spring 为各种项目提供了互补的测试模块，在这里它们被优雅地展示出来。

+   本书介绍了 GraalVM 的原生图像编译器和 Spring Native 项目。Spring Native 是生态系统中相对较新的成员，所以没有人会责怪托马斯没有包括它。但他确实包括了。真是个传奇人物！

+   托马斯介绍了本书中描述的许多做法背后的原因，这与新的技术概念相一致。我特别欣赏对敏捷和 GitOps 的处理。

Spring Boot 改变了世界，托马斯的这本书是探索这个勇敢的、“bootiful”新世界的最佳地图。买下它。阅读它。付诸行动。构建一些令人惊叹的东西，并享受你通往生产的旅程！

——Josh Long

Spring 开发者倡导者

VMware Tanzu

@starbuxman

## 前言

我清楚地记得第一次我参加实地考察，去了解我和我工作的公司开发的软件是如何在日常工作中被护士和从业者使用的。目睹我们的应用程序如何改善他们照顾患者的方式是一个难以置信的时刻。软件可以产生影响。这就是我们构建它的原因。我们通过技术解决问题，目标是向我们的用户、客户以及企业本身提供价值。

另一个我难以忘记的时刻是我了解到 Spring Boot。在此之前，我非常喜欢使用核心 Spring 框架。我特别喜爱我编写的用于管理安全、数据持久化、HTTP 通信和集成的代码。这是一项艰巨的工作，但值得，尤其是在当时 Java 生态系统的替代方案中。Spring Boot 改变了这一切。突然间，平台本身开始为我处理所有这些方面。所有处理基础设施问题和集成的代码都不再需要了。

但随后我意识到：所有处理基础设施问题和集成的代码都不再需要了！当我开始删除所有这些代码时，我意识到我在这上面花费了这么多时间，与应用程序的业务逻辑相比，这部分是产生价值的。我还意识到，与所有样板代码相比，真正属于业务逻辑的代码其实很少。这是一个转折点！

经过多年，Spring Boot 仍然是 Java 领域中构建企业级软件产品的领先平台之一，其受欢迎的原因之一就是它对开发者生产力的关注。使每个应用程序变得特殊的是其业务逻辑，而不是它如何暴露数据或连接到数据库。正是这种业务逻辑最终为用户、客户和业务带来了价值。利用广泛的框架、库和集成生态系统，Spring Boot 使开发者能够专注于业务逻辑，同时处理管道和样板代码。

云计算以及 Kubernetes（它迅速成为云的“操作系统”）也是我们领域中的另一个颠覆者。利用云计算模型的功能，我们可以构建云原生应用程序，并为我们的项目实现更好的可扩展性、弹性、速度和成本优化。最终，我们有通过我们的软件增加我们产生价值的机会，并以一种以前不可能的方式解决新类型的问题。

这本书的想法源于我帮助软件工程师在创造价值的过程中取得成功的愿望。我很高兴您决定加入我从代码到生产的这次冒险。Spring Boot 以及 Spring 生态系统在整体上代表了这样的旅程的骨干。云原生原则和模式将指导我们实现各种应用。持续交付实践将支持我们安全、快速、可靠地交付高质量的软件。Kubernetes 及其生态系统将为将我们的应用部署和发布给用户提供一个平台。

在构建和撰写这本书时，我的指导原则是提供相关、真实的示例，您可以直接将其应用到日常工作中。书中涵盖的所有技术和模式都旨在在有限的空间内，在生产的限制范围内，交付高质量的软件。我希望我成功地达到了这个目标。

再次感谢您与我一起踏上从代码到生产的云原生之旅。希望您阅读这本书时能有一个愉快且富有教育性的体验，并希望它能帮助您通过软件创造更多价值，并产生积极影响。

## 致谢

写一本书很困难，没有许多人在整个开发过程中给予我支持，这是不可能完成的。首先，我要感谢我的家人和朋友，他们一直在鼓励我，支持我。特别感谢我的父母 Sabrina 和 Plinio，我的姐姐 Alissa，以及我的祖父 Antonio，他们始终如一的支持和对我充满信心。

我要感谢我的朋友和同行工程师 Filippo、Luciano、Luca 和 Marco，他们在从最初的建议阶段就一直在支持我，并始终愿意提供反馈和建议来改进这本书。我要感谢 Systematic 的同事和朋友，他们在整个过程中一直给予我鼓励。能与你们一起工作，我感到非常幸运。

我要感谢都灵理工大学 Giovanni Malnati 教授，是他首先向我介绍了 Spring 生态系统，并改变了我职业生涯的轨迹。对 Spring 团队创建如此高效且宝贵的生态系统表示衷心的感谢。特别感谢 Josh Long，他的杰出工作让我受益匪浅，并为这本书写了序言。这对我的意义非常重大！

我要感谢整个 Manning 团队在将这本书打造成宝贵资源方面提供的巨大帮助。我特别想感谢 Michael Stephens（收购编辑）、Susan Ethridge（开发编辑）、Jennifer Stout（开发编辑）、Nickie Buckner（技术开发编辑）和 Niek Palm（技术校对）。他们的反馈、建议和鼓励为这本书增添了巨大价值。还要感谢 Mihaela Batinic´（审稿编辑）、Andy Marinkovich（生产编辑）、Andy Carroll（校对）、Keri Hales（校对）和 Paul Wells（生产经理）。

致所有审稿人：Aaron Makin、Alexandros Dallas、Andres Sacco、Conor Redmond、Domingo Sebastian、Eddú Meléndez Gonzales、Fatih Mehmet Ucar、François-David Lessard、George Thomas、Gilberto Taccari、Gustavo Gomes、Harinath Kuntamukkala、Javid Asgarov、João Miguel Pires Dias、John Guthrie、Kerry E. Koitzsch、Michał Rutka、Mladen Knežić、Mohamed Sanaulla、Najeeb Arif、Nathan B. Crocker、Neil Croll、Özay Duman、Raffaella Ventaglio、Sani Sudhakaran Subhadra、Simeon Leyzerzon、Steve Rogers、Tan Wee、Tony Sweets、Yogesh Shetty 和 Zorodzayi Mukuya，你们的建议帮助使这本书变得更好。

最后，我想感谢 Java 社区以及这些年来我在那里遇到的所有了不起的人：开源贡献者、同行演讲者、会议组织者，以及为使这个社区如此特别而做出贡献的每一个人。

## 关于本书

《云原生 Spring 实战》旨在帮助你使用 Spring Boot 和 Kubernetes 设计和部署云原生应用。它定义了一条通往生产的精选路径，并教授你可以立即应用于企业级应用的有效技术。它还逐步引导你从最初的想法到生产，展示云原生开发如何在软件开发生命周期的每个阶段增加商业价值。当你开发在线书店系统时，你将学习如何使用 Spring 和 Java 生态系统中的强大库来构建和测试云原生应用。逐章阅读，你将使用 REST API、数据持久化、响应式编程、API 网关、函数、事件驱动架构、弹性、安全、测试和可观察性。然后，本书扩展了如何将应用程序打包为容器镜像，如何为 Kubernetes 等云环境配置部署，如何使你的应用程序准备好生产，以及如何使用持续交付和持续部署设计从代码到生产的路径。

本书提供了一本动手、项目驱动的指南，帮助你导航日益复杂的云环境，并学习如何将模式和科技结合起来构建一个真正的云原生系统并将其投入生产。

### 适合阅读本书的人群？

本书面向希望了解更多关于使用 Spring Boot 和 Kubernetes 设计和部署生产级云原生应用的的开发者和架构师。

为了从这本书中获得最大收益，你需要具备 Java 编程技能、构建 Web 应用的经验，以及 Spring 核心功能的基本知识。我将假设你熟悉 Git、面向对象编程、分布式系统、数据库和测试。不需要有 Docker 和 Kubernetes 的经验。

### 本书组织结构：路线图

本书分为 4 部分，共涵盖 16 章。第一部分为你的云原生之旅从代码到生产奠定基础，并帮助你更好地理解本书中涵盖的主题，并将它们正确地放置在整体云原生图中。

+   第一章是云原生景观的介绍。它定义了云原生的含义，云原生应用程序的基本特性以及支持它们的流程。

+   第二章涵盖了云原生开发的原则，并指导你通过首次动手实践构建一个最小化的 Spring Boot 应用程序，并将其作为容器部署到 Kubernetes。

第二部分介绍了使用 Spring Boot 和 Kubernetes 构建生产就绪云原生应用程序的主要实践和模式。

+   第三章涵盖了启动新云原生项目的基本知识，包括组织代码库、管理依赖项和定义部署管道的提交阶段策略。你将学习如何使用 Spring MVC 和 Spring Boot Test 实现和测试 REST API。

+   第四章讨论了外部化配置的重要性，并涵盖了 Spring Boot 应用程序可用的某些选项，包括属性文件、环境变量和 Spring Cloud Config 配置服务。

+   第五章介绍了云中数据服务的主要方面，并展示了如何使用 Spring Data JDBC 将数据持久性添加到 Spring Boot 应用程序中。你将了解使用 Flyway 管理数据的生产选项以及使用 Testcontainers 进行测试的策略。

+   第六章是关于容器的；你将了解更多关于 Docker 的知识，以及如何使用 Dockerfile 和 Cloud Native Buildpacks 将 Spring Boot 应用程序打包成容器镜像。

+   第七章讨论了 Kubernetes，涵盖了服务发现、负载均衡、可伸缩性和本地开发工作流程。你还将了解如何将 Spring Boot 应用程序部署到 Kubernetes 集群。

第三部分涵盖了云原生系统中分布式系统的基本特性和模式，包括弹性、安全性、可伸缩性和 API 网关。它还描述了响应式编程和事件驱动架构。

+   第八章介绍了响应式编程和 Spring 响应式堆栈的主要功能，包括 Spring WebFlux 和 Spring Data R2DBC。它还教你如何使用 Project Reactor 使应用程序更具弹性。

+   第九章涵盖了 API 网关模式以及如何使用 Spring Cloud Gateway 构建边缘服务。你将学习如何使用 Spring Cloud 和 Resilience4J 构建具有弹性的应用程序，使用诸如重试、超时、回退、断路器和速率限制器等模式。

+   第十章描述了事件驱动架构，并教你如何使用 Spring Cloud Function、Spring Cloud Stream 和 RabbitMQ 来实现它们。

+   第十一章全部关于安全性，展示了如何使用 Spring Security、OAuth2、OpenID Connect 和 Keycloak 在云原生系统中实现身份验证。它还描述了当单页应用程序是系统的一部分时，如何解决 CORS 和 CSRF 等安全担忧。

+   第十二章继续安全之旅，涵盖了如何使用 OAuth2 和 Spring Security 在分布式系统中委派访问、保护 API 和数据以及根据用户的角色授权用户。

第四部分指导您完成最后几步，使您的云原生应用程序准备好生产环境，解决可观察性、配置管理、秘密管理和部署策略等问题。它还涵盖了无服务器和原生镜像。

+   第十三章介绍了如何使用 Spring Boot Actuator、OpenTelemetry 和 Grafana 可观察性堆栈使您的云原生应用程序可观察。您将学习如何配置 Spring Boot 应用程序以生成相关的遥测数据，例如日志、健康、指标、跟踪等。

+   第十四章涵盖了高级配置和秘密管理策略，包括 Kubernetes 原生选项，如 ConfigMaps、Secrets 和 Kustomize。

+   第十五章指导您完成云原生之旅的最后几步，并教您如何为生产环境配置 Spring Boot。然后，您将为应用程序设置持续部署并将它们部署到公共云中的 Kubernetes 集群，采用 GitOps 策略。

+   第十六章涵盖了使用 Spring Native 和 Spring Cloud Function 的无服务器架构和函数。您还将了解 Knative 及其强大的功能，这些功能在 Kubernetes 之上提供了卓越的开发者体验。

通常，我建议从第一章开始，按顺序逐章阅读。如果您根据您的特定兴趣选择不同的阅读顺序，请确保您首先阅读第一章到第三章，以便更好地理解书中使用的术语、模式和策略。即便如此，每一章都是基于前一章的，所以如果您那样做，可能会缺少一些上下文。

### 关于代码

本书提供了一种动手和以项目为导向的体验。从第二章开始，您将为一个虚构的在线书店构建由几个云原生应用程序组成的系统。

您可以从本书的 liveBook（在线）版本中获取可执行的代码片段，网址为 [`livebook.manning.com/book/cloud-native-spring-in-action`](https://livebook.manning.com/book/cloud-native-spring-in-action)。本书中开发的所有项目的所有源代码都可在 GitHub 上找到，并受 Apache License 2.0 许可 ([`github.com/ThomasVitale/cloud-native-spring-in-action`](https://github.com/ThomasVitale/cloud-native-spring-in-action))。对于每一章，您都会找到一个“开始”文件夹和一个“结束”文件夹。每一章都是基于前一章构建的，但您始终可以使用给定章节的“开始”文件夹作为起点，即使您没有跟随前面的章节。而“结束”文件夹包含完成该章节步骤后的最终结果，您可以将其与自己的解决方案进行比较。例如，您可以在 Chapter03 文件夹中找到第三章的源代码，其中包含 03-begin 和 03-end 文件夹。

本书开发的所有应用程序都是基于 Java 17 和 Spring Boot 2.7，并使用 Gradle 构建。这些项目可以导入任何支持 Java、Gradle 和 Spring Boot 的 IDE 中，例如 Visual Studio Code、IntelliJ IDEA 或 Eclipse。您还需要安装 Docker。第二章和附录 A 将提供更多信息，以帮助您设置本地环境。

这些示例已在 macOS、Ubuntu 和 Windows 上进行了测试。在 Windows 上，我建议使用 Windows Subsystem for Linux 来完成书中描述的部署和配置任务。在 macOS 上，如果您使用的是 Apple Silicon 计算机运行所有示例，但您可能会遇到一些工具的性能问题，因为当时它们没有为 ARM64 架构提供原生支持。当相关时，章节将包含额外的上下文信息。

之前提到的 GitHub 仓库 ([`github.com/ThomasVitale/cloud-native-spring-in-action`](https://github.com/ThomasVitale/cloud-native-spring-in-action)) 包含本书主分支上的所有源代码。除此之外，我计划维护一个 sb-2-main 分支，其中我将保持与 Spring Boot 2.x 未来版本发布的源代码同步更新，以及一个 sb-3-main 分支，其中我将根据 Spring Boot 3.x 的未来版本发布来更新源代码。

这本书包含了许多源代码示例，既有编号列表中的，也有与普通文本内联的。在两种情况下，源代码都以固定宽度字体格式化，如这样，以将其与普通文本区分开来。有时代码也会加粗，以突出显示章节中从先前步骤中更改的代码，例如，当新功能添加到现有代码行时。

在许多情况下，原始源代码已被重新格式化；我们添加了换行并重新调整了缩进，以适应书籍中的可用页面空间。在极少数情况下，即使这样也不够，列表中还包括了行续接标记（➥）。此外，当代码在文本中描述时，源代码中的注释通常已从列表中删除。许多列表旁边都有代码注释，突出显示重要概念。

### liveBook 讨论论坛

购买 *《云原生 Spring 实战》* 包括免费访问 liveBook，Manning 的在线阅读平台。使用 liveBook 的独家讨论功能，您可以在全球范围内或针对特定章节或段落附加评论。为自己做笔记、提问和回答技术问题，以及从作者和其他用户那里获得帮助都非常简单。要访问论坛，请访问 [`livebook.manning.com/book/cloud-native-spring-in-action/discussion`](https://livebook.manning.com/book/cloud-native-spring-in-action/discussion)。您还可以在 [`livebook.manning.com/discussion`](https://livebook.manning.com/discussion) 了解更多关于 Manning 的论坛和行为准则。

Manning 对我们读者的承诺是提供一个场所，在那里个人读者之间以及读者与作者之间可以进行有意义的对话。这不是对作者参与特定数量承诺的承诺，作者对论坛的贡献仍然是自愿的（且未付费）。我们建议您尝试向作者提出一些挑战性的问题，以免他的兴趣转移！只要书籍在印刷中，论坛和先前讨论的存档将从出版社的网站提供访问。

### 其他在线资源

您可以通过 Twitter (@vitalethomas)、LinkedIn ([www.linkedin.com/in/vitalethomas](http://www.linkedin.com/in/vitalethomas)) 或我的博客 [`thomasvitale.com`](https://thomasvitale.com) 在线找到我。

如果您想了解更多关于 Spring 生态系统的内容，我维护了一个包含书籍、视频、播客、课程和活动的教育资源列表，可在 [`github.com/ThomasVitale/awesome-spring`](https://github.com/ThomasVitale/awesome-spring) 找到。

## 关于作者

![Vitale_FM_Author-Photo](img/Vitale_FM_Author-Photo.png)

托马斯·维塔莱（Thomas Vitale）是一位专注于构建云原生、弹性且安全的企业应用的软件工程师和架构师。他在丹麦的 Systematic 公司设计和开发软件解决方案，在那里他一直在为云原生世界现代化平台和应用，重点关注开发体验和安全。

他的主要兴趣和关注领域包括 Java、Spring Boot、Kubernetes、Knative 以及云原生技术。托马斯支持持续交付实践，并相信一种旨在共同为用户、客户和业务创造价值的协作文化。他喜欢为 Spring Security 和 Spring Cloud 等开源项目做出贡献，并与社区分享知识。

托马斯拥有都灵理工大学（意大利）计算机工程硕士学位，专攻软件。他是 CNCF 认证的 Kubernetes 应用程序开发者、Pivotal 认证的 Spring 专业人员和 RedHat 认证的企业应用程序开发者。他的演讲活动包括 SpringOne、Spring I/O、KubeCon+CloudNativeCon、Devoxx、GOTO、JBCNConf、DevTalks 和 J4K。

## 关于封面插图

《云原生 Spring 实战》封面上的插图被标注为“Paisan Dequito”，或“基多农民”，取自雅克·格拉塞·德·圣索沃尔（Jacques Grasset de Saint-Sauveur）的作品集，该作品集于 1797 年出版。每一幅插图都是手工精细绘制和着色的。

在那些日子里，人们通过他们的服饰就能轻易地识别出他们居住的地方以及他们的职业或社会地位。曼宁通过基于几个世纪前丰富多样的地区文化的书封面来庆祝计算机行业的创新精神和主动性，这些文化通过像这一系列这样的图片被重新带回生活。
