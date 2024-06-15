# 第四章。解构单体架构

伊什切尔·鲁伊兹

> 终极目标应该是通过数字创新改善人类生活的质量。
> 
> 马化腾

通过历史，人类一直着迷于将想法和概念分解为简单或复合部分。通过分析和综合的结合，我们可以达到更高层次的理解。

亚里士多德称分析为“将每个复合物分解成组成这些合成物的要素。因为分析是综合的反面。*综合*是从原则到由原则派生的事物的道路，而分析是从终点返回到原则。”

软件开发遵循类似的方法：将系统分析为其组成部分，识别输入、期望输出和详细功能。在软件开发的分析过程中，我们意识到，非特定于业务的功能总是需要来处理输入，并通信或持久化输出。这使得明显的是，我们可以从可重复使用的、明确定义的、上下文绑定的原子功能中受益，这些功能可以被共享、消费或互连，以简化软件构建。

允许开发人员主要专注于实现业务逻辑，以满足客户/企业的明确定义的需求，满足一些潜在用户集的感知需求，或者使用功能来满足个人需求（自动化任务）一直是长期以来的愿望。每天都浪费太多时间在重新发明最常见的可靠样板代码。

近年来，微服务模式因承诺的优势而声名鹊起并获得动力。在实现这种架构模式的好处的同时，减少采用它的缺点，避免已知的反模式，采用最佳实践，并理解核心概念和定义至关重要。本章涵盖了微服务的反模式，并包含了使用 Spring Boot、Micronaut、Quarkus 和 Helidon 等流行微服务框架编写的代码示例。

传统上，单体架构提供或部署单个单元或系统，从单一源应用程序满足所有需求，可以识别出两个概念：*单体应用程序* 和 *单体架构*。

*单体应用程序* 有 *唯一的* 部署实例，负责执行特定功能所需的所有步骤。这种应用程序的一个特征是独特的执行接口点。

*单块架构*指的是所有需求均由单一来源处理，并且所有部分作为一个单元交付的应用程序。组件可能被设计为限制与外部客户的交互，以显式限制*私有*功能的访问。单块中的组件可能是相互连接或相互依赖的，而不是松散耦合的。换句话说，从外部或用户的视角来看，对其他独立组件的定义、接口、数据和服务知之甚少。

*粒度*是一个组件向软件的其他外部合作或协作部分公开的聚合级别。软件的粒度水平取决于几个因素，例如必须在一系列组件内保持的机密级别，不可暴露或对其他消费者可用。

现代软件架构越来越专注于通过捆绑或组合来自不同来源的软件组件来提供功能，这导致或强调了详细级别上的更细粒度。因此，向不同组件、客户或消费者公开的功能要比单块应用程序更多。

要确定一个模块有多独立或可互换，我们应该仔细看以下特征：

+   依赖的数量

+   这些依赖的强度

+   它所依赖的模块的稳定性

对前述特征赋予的任何高分应触发对模块建模和定义的第二次审查。

# 云计算

*云计算*有多个定义。彼得·梅尔（Peter Mell）和蒂姆·格兰斯（Tim Grance）将其定义为一种模型，用于实现对共享可配置计算资源池（如网络、服务器、存储、应用程序和服务）的无处不在、方便、按需网络访问，可以快速配置和释放，几乎不需要管理工作或与服务提供商的互动。

近年来，云计算有了显著增长。例如，2020 年第四季度云基础设施服务支出增长了 32%，达到了 399 亿美元。总支出比上一季度高出 30 亿美元，比 2019 年第四季度高出近 100 亿美元，据[Canalys 数据](https://oreil.ly/uZdZa)显示。

存在多家提供商，但市场份额并不均匀分布。三家领先的服务提供商是亚马逊网络服务（AWS）、微软 Azure 和谷歌云。AWS 是领先的云服务提供商，在 2020 年第四季度占据了总支出的 31%。Azure 的增长率加快，增长了 50%，市场份额接近 20%，而谷歌云占据了总市场的 7%。

云计算服务的利用一直存在滞后。Cinar Kilcioglu 和 Aadharsh Kannan 在 2017 年在“第 26 届国际万维网会议”上报告，数据中心中云资源的使用显示出租户配置和支付的资源（租用 VM）与实际资源利用（CPU、内存等）之间存在显著差距。也许客户只是将他们的 VM 保持开启，但实际上并没有使用它们。

云服务根据用于不同类型计算的类别进行划分：

**软件即服务（SaaS）**

客户可以使用提供者在云基础设施上运行的应用程序。这些应用程序可以通过薄客户端接口（如 Web 浏览器）或程序接口从各种客户端设备访问。客户不管理或控制底层云基础设施，包括网络、服务器、操作系统、存储甚至单个应用程序功能，但可能有限制的特定于用户的应用程序配置设置。

**平台即服务（PaaS）**

客户可以将使用由提供者支持的编程语言、库、服务和工具创建的客户制作或购买的应用程序部署到云基础设施。消费者不管理或控制底层云基础设施，包括网络、服务器、操作系统或存储，但可以控制已部署的应用程序，可能还可以配置应用程序托管环境的设置。

**基础设施即服务（IaaS）**

客户能够配置处理、存储、网络和其他基本计算资源。他们可以部署和运行任意软件，其中*包括操作系统和应用程序*。客户不管理或控制底层云基础设施，但可以控制操作系统、存储和部署的应用程序，并可能对某些网络组件有限的控制。

# **微服务**

*微服务* 这个术语并不是最近才出现的。彼得·罗杰斯在 2005 年提出了*微网络服务* 这个术语，同时倡导*软件即微网络服务* 这一概念。*微服务架构* ——作为面向服务架构（SOA）的一种演变——将应用程序组织为一组相对轻量级的模块化服务。从技术上讲，微服务是 SOA 实现方法的一种特殊化。

*微服务* 是小型且松散耦合的组件。与单体应用程序相比，它们可以独立部署、扩展和测试，具有单一职责，由上下文界定，是自治的和分散的。它们通常围绕业务能力构建，易于理解，并可以使用不同的技术栈进行开发。

一个微服务应该有多小？它应该足够*微小*，以允许小型、自包含和严格执行的功能原子共存、发展或替换前一个版本，以适应业务需求。

每个组件或服务几乎不了解其他独立组件的定义，与服务的所有交互都通过其 API 进行，该 API 封装了其实现细节。这些微服务之间的消息传递使用简单的协议，通常不需要大量数据。

## 反模式

微服务模式导致了显著的复杂性，并非在所有情况下都是理想的。该系统由许多独立工作的部分组成，其本质使其更难以预测在现实世界中的表现如何。

这种增加的复杂性主要是由于（潜在的）成千上万的微服务在分布式计算机网络中异步运行。请记住，难以理解的程序也难以编写、修改、测试和衡量。所有这些问题都将增加团队理解、讨论、跟踪和测试接口和消息格式所需的时间。

关于这个特定主题有几本书籍、文章和论文可供参考。我推荐访问[Microservices.io](https://microservices.io)、马克·理查兹（Mark Richards）的报告[*Microservices AntiPatterns and Pitfalls*](https://oreil.ly/KpzyW)（O’Reilly）以及 2018 年大卫德比（Davide Taibi）和瓦伦蒂娜·莱纳杜茨（Valentina Lenarduzz）在《IEEE 软件》上发表的“关于微服务坏味道定义”的论文。

一些最常见的反模式包括以下内容：

API 版本控制（*静态协议陷阱*）

API 需要进行语义版本控制，以允许服务知道它们是否正在与正确版本的服务通信，或者是否需要调整其通信以适应新的协议。

不当的服务隐私依赖性

微服务需要其他服务的私密数据而不是处理自己的数据，这通常与数据建模问题有关。可以考虑的解决方案之一是合并这些微服务。

多用途巨型服务

几个业务功能被实现在同一个服务中。

记录

错误和微服务信息被隐藏在每个微服务容器内。在软件生命周期的各个阶段发现问题时，应优先采用分布式日志记录系统。

复杂的服务间或循环依赖

*循环服务关系* 被定义为两个或多个相互依赖的服务之间的关系。循环依赖可能会损害服务扩展或独立部署的能力，并违反无环依赖原则（ADP）。

缺失的 API 网关

当微服务直接相互通信，或者当服务消费者直接与每个微服务通信时，系统的复杂性增加，维护减少。在这种情况下的最佳实践是使用 API 网关。

一个 *API 网关* 接收来自客户端的所有 API 调用，然后通过请求路由、组合和协议转换将它们引导到适当的微服务。网关通常通过调用多个微服务并聚合结果来处理请求，以确定最佳路由。它还能够在内部使用之间进行 Web 协议和 Web 友好协议之间的转换。

应用程序可以使用 API 网关为移动客户端提供一个单一的端点，通过单个请求查询所有产品数据。API 网关整合了各种服务，如产品信息和评论，并将结果合并和公开。

API 网关是应用程序访问数据、业务逻辑或功能（RESTful API 或 WebSocket API）的门卫，允许实时双向通信应用程序。API 网关通常处理接受和处理多达数十万个并发 API 调用的所有任务，包括流量管理、跨源资源共享（CORS）支持、授权和访问控制、阻塞、管理和 API 版本控制。

过度共享

在分享足够的功能以避免重复自己与创建依赖混乱的纠结之间，有一条薄线阻止了服务变更分离。如果需要更改过度共享的服务，评估接口的建议变更最终会导致一个涉及更多开发团队的组织任务。

在某些时候，需要分析是否将冗余或库提取到新的共享服务中，这些相关的微服务可以独立安装和开发。

## DevOps 和 微服务

微服务完美地符合 DevOps 理念，利用小团队逐步对企业服务进行功能更改——将大问题分解成小片段并系统化处理的理念。为了减少开发、测试和部署之间的摩擦，必须存在一系列持续交付管道，以保持这些阶段的稳定流动。

DevOps 是这种架构风格成功的关键因素，提供必要的组织变更，以最小化负责每个组件的团队之间的协调，并消除开发和运营团队之间有效互动的障碍。

###### 警告

我强烈反对任何团队在没有健全的 CI/CD 基础设施或对流水线基本概念没有广泛理解的情况下采用微服务模式。

## 微服务框架

JVM 生态系统庞大且提供了许多特定用例的替代方案。提供了几十种微服务框架和库，以至于在候选项中选择优胜者可能有些棘手。

话虽如此，由于几个原因，某些候选框架已经获得了流行：开发者体验、上市时间、可扩展性、资源（CPU、内存）消耗、启动速度、故障恢复、文档、第三方集成等等。这些框架——Spring Boot、Micronaut、Quarkus 和 Helidon——在接下来的章节中进行了介绍。请注意，一些说明可能需要根据更新版本进行额外调整，因为其中一些技术正在快速发展。我强烈建议查阅每个框架的文档。

此外，这些示例需要至少 Java 11，并且尝试使用本地镜像还需要安装 GraalVM。有许多方法可以在您的环境中安装这些版本。我建议使用[SDKMAN!](https://sdkman.io)来安装和管理它们。为简洁起见，我专注于生产代码——一个单一框架可以填写整本书！毫无疑问，您还应该关注测试。每个示例的目标是构建一个简单的“Hello World” REST 服务，该服务可以接受一个可选的名称参数并回复问候语。

如果您以前没有使用过 GraalVM，它是一个涵盖几种技术的综合项目，使以下功能成为可能：

+   一个用 Java 编写的即时编译器（JIT），可以在运行时编译代码，将解释代码转换为可执行代码。Java 平台已经有过几个 JIT，大多数是用 C 和 C++组合编写的。Graal 碰巧是最现代的一个，用 Java 编写。

+   *Substrate VM* 是一个虚拟机，能够在 JVM 之上运行托管语言，如 Python、JavaScript 和 R，使得托管语言能够更紧密地集成 JVM 的能力和特性。

+   [本地镜像](https://sdkman.io)是一种依赖预编译（AOT）的实用工具，将字节码转换为机器可执行代码。所得的转换产生一个特定于平台的二进制可执行文件。

这里介绍的四个候选框架都以某种方式支持 GraalVM，主要依赖于 GraalVM Native Image 来生成特定于平台的二进制文件，旨在减少部署大小和内存消耗。请注意，在使用 Java 模式和 GraalVM Native Image 模式之间存在权衡。后者可以生成具有较小内存占用和更快启动时间的二进制文件，但需要较长的编译时间；长时间运行的 Java 代码最终会变得更加优化（这是 JVM 的关键特性之一），而原生二进制文件在运行时无法进行优化。开发体验也各不相同，您可能需要使用额外的工具进行调试、监控、测量等。

## Spring Boot

*Spring Boot* 可能是这四个候选框架中最为人熟知的，因为它建立在 Spring Framework 所奠定的遗产之上。如果开发者调查结果可信，超过 60% 的 Java 开发者在与 Spring 相关的项目中有一定的经验，使 Spring Boot 成为最受欢迎的选择。

Spring 的方式允许您通过组合现有组件、定制其配置并承诺低成本的代码拥有权来组装应用程序（或在我们的情况下是微服务），因为您的自定义逻辑理论上应比框架提供的内容更小，对于大多数组织来说这是正确的。关键是找到一个可以在编写自己的组件之前进行调整和配置的现有组件。Spring Boot 团队着重于添加所需的多个有用集成，从数据库驱动程序到监控服务、日志记录、日志处理、批处理、报告生成等等。

启动 Spring Boot 项目的典型方式是浏览至 [Spring Initializr](https://start.spring.io)，选择您在应用程序中需要的功能，然后单击生成按钮。此操作将创建一个 ZIP 文件，您可以将其下载到本地环境以开始使用。在 图 4-1 中，我选择了 Web 和 Spring Native 功能。第一个功能添加了组件，使您可以通过 REST API 公开数据；第二个功能增强了构建，使用 Graal 可以创建额外的打包机制，生成原生镜像。

在项目的根目录解压 ZIP 文件并运行`./mvnw verify`命令，确保一个健康的起点。如果您之前没有在目标环境上构建过 Spring Boot 应用程序，您会注意到该命令会下载一组依赖。这是正常的 Apache Maven 行为。除非在 *pom.xml* 文件中更新了依赖版本，否则下次调用 Maven 命令时不会再次下载这些依赖。

![dtjd 0401](img/dtjd_0401.png)

###### 图 4-1\. Spring Initializr

项目结构应该是这样的：

```java
.
├── HELP.md
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── demo
    │   │               ├── DemoApplication.java
    │   │               ├── Greeting.java
    │   │               └── GreetingController.java
    │   └── resources
    │       ├── application.properties
    │       ├── static
    │       └── templates
    └── test
        └── java
```

我们当前的任务需要两个未由 Spring Initializr 网站创建的附加源：*Greeting.java* 和 *GreetingController.java*。这两个文件可以使用您选择的文本编辑器或 IDE 创建。首先，*Greeting.java* 定义了一个数据对象，将用于将内容呈现为 JavaScript 对象表示法（JSON），这是一种通过 REST 公开数据的典型格式。还支持其他格式，但是 JSON 支持无需任何额外的依赖项即可直接使用。此文件应如下所示：

```java
package com.example.demo;

public class Greeting {
    private final String content;

    public Greeting(String content) {
        this.content = content;
    }

    public String getContent() {
        return content;
    }
}
```

除了它是不可变的数据持有者之外，这个数据持有者没有什么特别之处；根据您的用例，您可能希望切换到可变的实现，但目前这样就足够了。接下来是 REST 端点本身，定义为 */greeting* 路径上的一个`GET`调用。Spring Boot 更偏爱 *controller* 的原型来创建这种组件，毫无疑问是在回顾 Spring MVC（是的，那就是模型-视图-控制器）作为首选选项来创建 Web 应用程序的日子里。可以随意使用不同的文件名，但是组件的注解必须保持不变：

```java
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GreetingController {
    private static final String template = "Hello, %s!";

    @GetMapping("/greeting")
    public Greeting greeting(@RequestParam(value = "name",
        defaultValue = "World") String name) {
        return new Greeting(String.format(template, name));
    }
}
```

控制器可以接受 `name` 参数作为输入，并在未提供此参数时使用值 `World`。请注意，映射方法的返回类型是一个普通的 Java 类型；这是我们在前一步中定义的数据类型。Spring Boot 将根据应用于控制器及其方法的注解和设置的合理默认值，自动将数据从 JSON 格式转换为 JSON 格式。如果我们保持代码不变，`greeting()` 方法的返回值将自动转换为 JSON 负载。这是 Spring Boot 开发经验的威力，依赖于可以根据需要进行微调的默认值和预定义的配置。

您可以通过调用 `/.mvnw spring-boot:run` 命令来运行应用程序，该命令将在构建过程中运行应用程序，也可以通过生成应用程序 JAR 并手动运行它来运行应用程序，即 `./mvnw package` 后跟 `java -jar target/demo-0.0.1.SNAPSHOT.jar`。无论哪种方式，都会启动一个内嵌的 Web 服务器，监听 8080 端口； */greeting* 路径将映射到 *GreetingController* 的一个实例。现在只剩下发出一些查询，比如以下内容：

```java
// using the default name parameter
$ curl http://localhost:8080/greeting
{"content":"Hello, World!"}

// using an explicit value for the name parameter
$ curl http://localhost:8080/greeting?name=Microservices
{"content":"Hello, Microservices!"}
```

在运行应用程序时，请注意应用程序生成的输出。在我的本地环境中，它显示（平均）JVM 启动需要 1.6 秒，而应用程序初始化需要 600 毫秒。生成的 JAR 文件大小大约为 17 MB。您可能还想记录这个微不足道的应用程序的 CPU 和内存消耗。有人建议使用 GraalVM Native Image 可以减少启动时间和二进制文件大小。让我们看看如何在 Spring Boot 中实现这一点。

记得当项目创建时我们选择了 Spring Native 特性吗？不幸的是，到了 2.5.0 版本，生成的项目在*pom.xml*文件中并未包含所有必需的指令。我们需要进行一些调整。首先，由`spring-boot-maven-plugin`创建的 JAR 文件需要一个分类器；否则，生成的本地镜像可能无法正确创建。这是因为应用程序 JAR 文件已经包含了所有依赖项，位于 Spring Boot 特定路径下，这个路径并不被`native-image-maven-plugin`处理，我们也需要进行配置。更新后的*pom.xml*文件应该如下所示：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.0</version>
    </parent>
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>11</java.version>
        <spring-native.version>0.10.0-SNAPSHOT</spring-native.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.experimental</groupId>
            <artifactId>spring-native</artifactId>
            <version>${spring-native.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <classifier>exec</classifier>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.experimental</groupId>
                <artifactId>spring-aot-maven-plugin</artifactId>
                <version>${spring-native.version}</version>
                <executions>
                    <execution>
                        <id>test-generate</id>
                        <goals>
                            <goal>test-generate</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>generate</id>
                        <goals>
                            <goal>generate</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
    <repositories>
        <repository>
            <id>spring-release</id>
            <name>Spring release</name>
            <url>https://repo.spring.io/release</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-release</id>
            <name>Spring release</name>
            <url>https://repo.spring.io/release</url>
        </pluginRepository>
    </pluginRepositories>

    <profiles>
        <profile>
            <id>native-image</id>
            <build>
                <plugins>
                    <plugin>
                        <groupId>org.graalvm.nativeimage</groupId>
                        <artifactId>native-image-maven-plugin</artifactId>
                        <version>21.1.0</version>
                        <configuration>
                            <mainClass>
                                com.example.demo.DemoApplication
                            </mainClass>
                        </configuration>
                        <executions>
                            <execution>
                                <goals>
                                    <goal>native-image</goal>
                                </goals>
                                <phase>package</phase>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>
</project>
```

在我们尝试之前还有一步：确保安装了与*pom.xml*文件中找到的`native-image-maven-plugin`版本接近的 GraalVM 版本作为您当前的 JDK。还必须在系统中安装`native-image`可执行文件；您可以通过调用`gu install native-image`来完成。`gu`命令由 GraalVM 安装提供。

当所有设置就绪后，我们可以通过调用`./mvnw -Pnative-image package`生成一个本地可执行文件。你会注意到屏幕上会有大量的文本输出，可能会下载新的依赖项，并且可能会有一些关于缺少类的警告——这是正常的。构建时间也比平时长，这就是这种打包解决方案的权衡所在：我们增加了开发时间以加快生产环境中的执行时间。一旦命令完成，您会注意到在*target*目录中出现了一个名为*com.example.demo.demoapplication*的新文件。这就是本地可执行文件。继续运行它吧。

你注意到启动速度有多快了吗？在我的环境中，平均启动时间为 0.06 秒，而应用程序初始化需要 30 毫秒。你可能还记得在 Java 模式下运行时，这些数字分别为 1.6 秒和 600 毫秒。这真是一个严重的速度提升！现在看看可执行文件的大小；在我的情况下，大约是 78 MB。哦，看起来有些事情变得更糟了，或者说没有？这个可执行文件是一个单一的二进制文件，包含了运行应用程序所需的一切，而我们之前使用的 JAR 文件则需要 Java 运行时才能运行。Java 运行时的大小通常在 200 MB 左右，并由多个文件和目录组成。当然，可以使用[jlink](https://oreil.ly/agfRB)创建较小的 Java 运行时，这样在构建过程中会增加另一个步骤。没有免费的午餐。

现在我们暂停使用 Spring Boot，记住，它的功能远不止这里展示的。接下来我们看看下一个框架。

## Micronaut

*Micronaut* 于 2017 年诞生，是对 Grails 框架的一次现代化重新构想。Grails 是少数成功的 Ruby on Rails（RoR）框架“克隆”之一，利用 Groovy 编程语言。Grails 在几年间引起了轰动，直到 Spring Boot 的兴起使其失去了关注，促使 Grails 团队寻找替代方案，最终推出了 Micronaut。从表面上看，Micronaut 提供了与 Spring Boot 类似的用户体验，因为它也允许开发人员基于现有组件和合理的默认设置来构建应用程序。

Micronaut 的一个关键区别在于使用编译时依赖注入来组装应用程序，而不是运行时依赖注入，这与目前使用 Spring Boot 组装应用程序的首选方式不同。这一看似微不足道的改变让 Micronaut 在运行时提升了速度，因为应用程序在启动时花费的时间更少；这也可以减少内存消耗，并且减少对 Java 反射的依赖，后者在直接方法调用之前的历史上速度较慢。

有几种启动 Micronaut 项目的方式，但首选的方法是浏览到 [Micronaut Launch](https://oreil.ly/QAdrG)，选择您想要添加到项目中的设置和功能。默认的应用程序类型定义了构建基于 REST 的应用程序所需的最小设置，例如我们将在几分钟内讲解的内容。一旦满意您的选择，请点击“生成项目”按钮，如 图 4-2 所示，这将生成一个 ZIP 文件，可以下载到您的本地开发环境中。

![dtjd 0402](img/dtjd_0402.png)

###### 图 4-2\. Micronaut 启动

与我们为 Spring Boot 所做的类似，解压 ZIP 文件并在项目根目录运行`./mvnw verify`命令会确保一个良好的起点。如果一切顺利，此命令会根据需要下载插件和依赖项；如果构建成功，几秒钟后应该看到这样的输出。添加一对额外的源文件后，项目结构应如下所示：

```java
.
├── README.md
├── micronaut-cli.yml
├── mvnw
├── mvnw.bat
├── pom.xml
└── src
    └── main
        ├── java
        │   └── com
        │       └── example
        │           └── demo
        │               ├── Application.java
        │               ├── Greeting.java
        │               └── GreetingController.java
        └── resources
            ├── application.yml
            └── logback.xml
```

*Application.java* 源文件定义了入口点，暂时不需要更新。同样，我们也不需要更改 *application.yml* 资源文件；该资源提供的配置属性目前不需要更改。

我们需要另外两个源文件：由*Greeting.java*定义的数据对象，其责任是包含发送给消费者的消息，以及由*GreetingController.java*定义的实际 REST 端点。控制器原型追溯到 Grails 规范所制定的约定，并几乎被所有 RoR 克隆框架所遵循。您可以根据您的领域需求更改文件名，但必须保留`@Controller`注解。数据对象的源代码应如下所示：

```java
package com.example.demo;

import io.micronaut.core.annotation.Introspected;

@Introspected
public class Greeting {
    private final String content;

    public Greeting(String content) {
        this.content = content;
    }

    public String getContent() {
        return content;
    }
}
```

对于这个类，我们再次依赖于不可变设计。请注意使用的`@Introspected`注解，这会告诉 Micronaut 在编译时检查类型并将其包含为依赖注入过程的一部分。通常情况下，可以省略此注解，因为 Micronaut 会自动识别出需要的类。但是，在使用 GraalVM Native Image 生成本地可执行文件时，它的使用是至关重要的；否则，可执行文件将无法完整生成。第二个文件应该是这样的：

```java
package com.example.demo;

import io.micronaut.http.annotation.Controller;
import io.micronaut.http.annotation.Get;
import io.micronaut.http.annotation.QueryValue;

@Controller("/")
public class GreetingController {
    private static final String template = "Hello, %s!";

    @Get(uri = "/greeting")
    public Greeting greeting(@QueryValue(value = "name",
        defaultValue = "World") String name) {
        return new Greeting(String.format(template, name));
    }
}
```

我们可以看到控制器定义了一个映射到`/greeting`的单个端点，接受一个名为`name`的可选参数，并返回数据对象的一个实例。默认情况下，Micronaut 会将返回值解析为 JSON，因此无需额外配置。应用程序的运行可以通过两种方式完成。您可以调用`./mvnw mn:run`，将其作为构建过程的一部分运行应用程序，或者调用`./mvnw package`，它会在*target*目录中创建一个名为*demo-0.1.jar*的文件，可以像传统方式一样启动它，即`java -jar target/demo-0.1.jar`。调用 REST 端点进行一些查询可能会产生类似于以下输出：

```java
// using the default name parameter
$ curl http://localhost:8080/greeting
{"content":"Hello, World!"}

// using an explicit value for the name parameter
$ curl http://localhost:8080/greeting?name=Microservices
{"content":"Hello, Microservices!"}
```

任何一个命令都能快速启动应用程序。在我的本地环境中，应用程序平均在 500 毫秒内准备好处理请求，这比 Spring Boot 的相同行为速度快三倍。JAR 文件的大小也小了一点，总共为 14 MB。尽管这些数字可能令人印象深刻，但如果使用 GraalVM Native Image 将应用程序转换为本地可执行文件，我们可以获得速度提升。幸运的是，Micronaut 的方式对这种设置更友好，生成的项目中已经配置了我们需要的一切。就这样。无需更新构建文件的其他设置，一切都已准备就绪。

您确实需要安装 GraalVM 及其`native-image`可执行文件，就像我们以前做的那样。创建本地可执行文件只需调用`./mvnw -Dpackaging=native-image package`，几分钟后，我们应该能得到一个名为`demo`的可执行文件（事实上，这是项目的`artifactId`，如果您想知道的话），位于*target*目录下。使用本地可执行文件启动应用程序平均启动时间为 20 毫秒，比 Spring Boot 快三分之一。可执行文件大小为 60 MB，与 JAR 文件的减小大小相对应。

让我们停止探索 Micronaut，并转向下一个框架：Quarkus。

## Quarkus

尽管*Quarkus*在 2019 年初宣布，但其工作开始得早得多。Quarkus 与我们迄今看到的两个候选者有很多相似之处。它提供基于组件、约定大于配置和生产力工具的优秀开发体验。更重要的是，Quarkus 决定也采用像 Micronaut 一样的编译时依赖注入，使其能够获得同样的好处，如更小的二进制文件、更快的启动时间和更少的运行时魔法。同时，Quarkus 还添加了自己的风格和独特性，对某些开发人员来说可能最重要的是，Quarkus 更多地依赖于标准而不是其他两个候选者。Quarkus 实现了 MicroProfile 规范，这些规范来自于 JakartaEE（以前称为 JavaEE），并在 MicroProfile 项目的保护伞下开发了其他标准。

您可以访问[Quarkus 配置您的应用程序页面](https://code.quarkus.io)开始使用 Quarkus，配置值并下载 ZIP 文件。此页面包含许多好东西，包括许多扩展选项，可用于配置特定的集成，如数据库、REST 能力、监控等。必须选择 RESTEasy Jackson 扩展，以便 Quarkus 能够无缝地将值编组为 JSON 并从 JSON 解组。单击“生成您的应用程序”按钮应提示您将 ZIP 文件保存到本地系统，其内容应与以下内容类似：

```java
.
├── README.md
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    ├── main
    │   ├── docker
    │   │   ├── Dockerfile.jvm
    │   │   ├── Dockerfile.legacy-jar
    │   │   ├── Dockerfile.native
    │   │   └── Dockerfile.native-distroless
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── demo
    │   │               ├── Greeting.java
    │   │               └── GreetingResource.java
    │   └── resources
    │       ├── META-INF
    │       │   └── resources
    │       │       └── index.html
    │       └── application.properties
    └── test
        └── java
```

我们可以看到，Quarkus 默认添加 Docker 配置文件，因为它旨在通过容器和 Kubernetes 解决云中的微服务架构问题。但随着时间的推移，它的范围已经扩展，通过支持额外的应用程序类型和架构。*GreetingResource.java*文件也是默认创建的，它是一个典型的 Jakarta RESTful Web Services（JAX-RS）资源。我们需要对该资源进行一些调整，以使其能够处理*Greeting.java*数据对象。以下是该资源的源代码：

```java
package com.example.demo;

public class Greeting {
    private final String content;

    public Greeting(String content) {
        this.content = content;
    }

    public String getContent() {
        return content;
    }
}
```

代码与本章前面所见的基本相同。关于这个不可变数据对象没有什么新鲜或意外的。现在，在 JAX-RS 资源的情况下，事情看起来相似但又有所不同，因为我们寻求的行为与以前相同，只是我们指示框架执行其魔术的方式是通过 JAX-RS 注解。因此，代码如下：

```java
package com.example.demo;

import javax.ws.rs.DefaultValue;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.QueryParam;

@Path("/greeting")
public class GreetingResource {
    private static final String template = "Hello, %s!";

    @GET
    public Greeting greeting(@QueryParam("name")
        @DefaultValue("World") String name) {
        return new Greeting(String.format(template, name));
    }
}
```

如果您熟悉 JAX-RS，这段代码对您来说应该不会感到意外。但是如果您不熟悉 JAX-RS 注解，我们在这里所做的是用我们想要响应的 REST 路径标记资源；我们还指示`greeting()`方法将处理`GET`调用，并且其`name`参数具有默认值。要指示 Quarkus 将返回值转换为 JSON，无需做任何其他操作，因为这将默认发生。

运行应用程序可以通过几种方式完成，其中一种是使用开发者模式作为构建的一部分。这是具有独特 Quarkus 风味的功能之一，它允许您在不手动重启应用程序的情况下自动运行应用程序并获取您所做的任何更改。您可以通过调用`/.mvnw compile quarkus:dev`来激活此模式。如果您对源文件进行任何更改，您会注意到构建将自动重新编译和加载应用程序。

您也可以像之前看到的那样使用`java`解释器运行应用程序，结果是一个命令，例如`java -jar target/quarkus-app/quarkus-run.jar`。请注意，我们使用了一个不同的 JAR，尽管*demo-1.0.0-SNAPSHOT.jar*确实存在于*target*目录中；之所以这样做是因为 Quarkus 应用了自定义逻辑以加速启动过程，即使在 Java 模式下也是如此。

运行应用程序应该会导致平均启动时间为 600 毫秒，这几乎接近 Micronaut 的情况。此外，完整应用程序的大小在 13 MB 左右。向应用程序发送一对`GET`请求，无论是否带有`name`参数，都会产生类似以下输出的结果：

```java
// using the default name parameter
$ curl http://localhost:8080/greeting
{"content":"Hello, World!"}

// using an explicit value for the name parameter
$ curl http://localhost:8080/greeting?name=Microservices
{"content":"Hello, Microservices!"}
```

毫无疑问，Quarkus 也支持通过 GraalVM Native Image 生成本机可执行文件，因为它面向推荐使用小二进制大小的云环境。由于这个原因，Quarkus 自带电池，就像 Micronaut 一样，并且从一开始就生成您所需的一切。无需更新构建配置即可开始使用本机可执行文件。与其他示例一样，您必须确保当前 JDK 指向 GraalVM 分发，并且`native-image`可执行文件位于您的路径中。完成这一步后，剩下的就是通过调用`./mvnw -Pnative package`将应用程序打包为本机可执行文件。这会激活`native`配置文件，该配置文件指示 Quarkus 构建工具生成本机可执行文件。

几分钟后，构建应该已经生成了一个名为*demo-1.0.0-SNAPSHOT-runner*的可执行文件，该文件位于*target*目录内。运行这个可执行文件显示应用程序平均启动时间为 15 毫秒。可执行文件的大小接近 47 MB，这使得 Quarkus 在与以前的候选框架相比的启动速度和最小可执行文件大小方面表现最佳。

目前我们告别了 Quarkus，留下了第四个候选框架：Helidon。

## Helidon

最后但并非最不重要的是，*Helidon* 是一个专门用于构建微服务的框架，有两种版本：SE 和 MP。MP 版本代表*MicroProfile*，通过利用标准的力量来构建应用程序；这个版本是 MicroProfile 规范的完整实现。另一方面，SE 版本虽然没有实现 MicroProfile，但使用不同的一组 API 提供了类似的功能。根据你想要与之交互的 API 和你对标准的偏好，选择一个版本；无论如何，Helidon 都能完成工作。

鉴于 Helidon 实现了 MicroProfile，我们可以使用另一个站点来启动 Helidon 项目。可以使用[MicroProfile Starter site](https://oreil.ly/3U7RG)（见图 4-3）来为 MicroProfile 规范的所有支持实现创建项目版本。

![dtjd 0403](img/dtjd_0403.png)

###### 图 4-3\. MicroProfile Starter

浏览到该站点，选择您感兴趣的 MP 版本，选择 MP 实现（在我们的例子中是 Helidon），并可能定制一些可用的功能。然后点击下载按钮以下载包含生成项目的 ZIP 文件。ZIP 文件包含类似于以下结构的项目结构，当然，我已经使用两个文件更新了源文件，以使应用程序按照我们想要的方式工作：

```java
.
├── pom.xml
├── readme.md
└── src
    └── main
        ├── java
        │   └── com
        │       └── example
        │           └── demo
        │               ├── Greeting.java
        │               └── GreetingResource.java
        └── resources
            ├── META-INF
            │   ├── beans.xml
            │   └── microprofile-config.properties
            ├── WEB
            │   └── index.html
            ├── logging.properties
            └── privateKey.pem
```

碰巧的是，源文件*Greeting.java*和*GreetingResource.java*与我们在 Quarkus 示例中看到的源文件完全相同。这是怎么可能的呢？首先因为代码显然是微不足道的，但更重要的是因为这两个框架都依赖于标准的力量。事实上，*Greeting.java*文件在所有框架中几乎完全相同，只有 Micronaut 需要额外的注解，但仅在您有兴趣生成本地可执行文件时才需要；否则，它是完全相同的。如果您决定在浏览其他内容之前直接跳到这部分，这是*Greeting.java*文件的样子：

```java
package com.example.demo;

import io.helidon.common.Reflected;

@Reflected
public class Greeting {
    private final String content;

    public Greeting(String content) {
        this.content = content;
    }

    public String getContent() {
        return content;
    }
}
```

它只是一个普通的不可变数据对象，只有一个访问器。定义应用程序所需的 REST 映射的*Greeting​Re⁠source.java*文件如下：

```java
package com.example.demo;

import javax.ws.rs.DefaultValue;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.QueryParam;

@Path("/greeting")
public class GreetingResource {
    private static final String template = "Hello, %s!";

    @GET
    public Greeting greeting(@QueryParam("name")
        @DefaultValue("World") String name) {
        return new Greeting(String.format(template, name));
    }
}
```

我们很欣赏 JAX-RS 注解的使用，因为我们可以看到此时无需特定于 Helidon 的 API。运行 Helidon 应用程序的首选方法是将二进制文件打包并使用`java`解释器运行它们。也就是说，虽然我们在构建工具集成方面稍有损失（目前如此），但我们仍然可以使用命令行进行迭代开发。因此，调用`mvn package`后接着是`java -jar/demo.jar`编译、打包并运行应用程序，嵌入式 Web 服务器监听 8080 端口。我们可以向其发送一些查询，例如：

```java
// using the default name parameter
$ curl http://localhost:8080/greeting
{"content":"Hello, World!"}

// using an explicit value for the name parameter
$ curl http://localhost:8080/greeting?name=Microservices
{"content":"Hello, Microservices!"}
```

如果你查看应用程序进程运行的输出，你会看到应用程序的平均启动时间为 2.3 秒，这使其成为迄今为止最慢的候选框架，而二进制文件的大小接近 15 MB，属于所有测量值的中间位置。但正如谚语所说，不能以貌取人。Helidon 提供了更多自动配置的开箱即用功能，这解释了额外的启动时间和更大的部署大小。

如果启动速度和部署大小是问题，你可以重新配置构建，移除可能不需要的功能，并切换到本地可执行模式。幸运的是，Helidon 团队也接受了 GraalVM Native Image，并且每个像我们自己一样启动的 Helidon 项目都配备了创建本地二进制文件所需的配置。如果遵循惯例，无需调整*pom.xml*文件。执行`mvn -Pnative-image package`命令，你会在*target*目录中找到一个名为*demo*的可执行二进制文件。这个可执行文件大约有 94 MB，迄今为止最大，平均启动时间为 50 毫秒，与之前的框架处于相同范围内。

到目前为止，我们已经初步了解了每个框架提供的功能，从基本特性到构建工具的集成。作为提醒，有几个理由可以选择一个候选框架而不是另一个。我鼓励你为每个影响你的开发需求的相关功能/方面编写一个矩阵，并对每个候选项进行评估。

# 无服务器

本章开始时讨论了通常由组件和层组合成的单片应用程序和架构。对特定部分的更改或更新需要更新和部署整个单体。某个特定位置的故障也可能导致整体崩溃。然后我们转向了微服务。将单体应用程序拆分为可以单独和独立更新和部署的更小块应该解决之前提到的问题，但微服务也带来了一系列其他问题。

以前，在大型服务器上托管的应用服务器内运行单体已经足够了，配备了少数副本和负载均衡器以作为补充。但这种设置存在可伸缩性问题。采用微服务的方法，我们可以根据负载的大小扩展或折叠服务的网格。这增强了弹性，但现在我们必须协调多个实例并提供运行时环境，负载均衡器变得必不可少，API 网关也是必需的，网络延迟也开始显现，我有提到分布式追踪吗？是的，这些都是需要注意和管理的许多事情。但如果不需要这些怎么办？如果有人可以处理基础设施、监控和其他运行大规模应用所需的“琐事”，那就是无服务器方法的用武之地：你可以专注于手头的业务逻辑，让无服务器提供者处理其他所有事务。

当将组件分解为较小的部分时，应该思考一件事：“我可以将这个组件转化为什么样的最小可重复使用的代码片段？” 如果你的答案是一个带有少量方法和一两个注入的协作者/服务的 Java 类，那么你已经接近了，但还不够。事实上，最小的可重复使用代码片段是一个单一的方法。想象一下，一个微服务定义为一个执行以下步骤的单一类：

1.  读取输入参数并按照下一步所需的格式转换为可消耗的格式

1.  执行服务所需的实际行为，例如向数据库发出查询、索引或记录日志

1.  将处理后的数据转换为输出格式

现在，这些步骤中的每一个可能会被组织成单独的方法。你可能很快意识到，其中一些方法是可重复使用的或者可以参数化的。解决这个问题的典型方法是为微服务提供一个公共*超类型*。这会在类型之间创建强依赖关系，在某些使用情况下是可以接受的。但对于其他情况，共同代码的更新必须尽快进行，以版本化的方式进行，而不会中断当前正在运行的代码，所以我恐怕我们可能需要另一种方法。

考虑到这种情况，如果通用代码被提供为一组可以独立调用的方法，它们的输入和输出以这样一种方式组合，即您建立了一个数据转换的管道，则我们现在所知道的称为*函数*。像*函数即服务*（FaaS）这样的服务是无服务器提供者的一个常见主题。

总之，FaaS 是一种精致的方式，即您根据可能的最小部署单元组合应用程序，并让提供者为您解决所有基础设施细节。在接下来的章节中，我们将构建并部署一个简单的函数到云端。

## 设置中

如今，每个主要的云提供商都有一个可供使用的 FaaS 提供，其附加组件可以连接其他工具，用于监控、日志记录、灾难恢复等等；只需选择满足您需求的一个。在本章中，我们将选择 AWS Lambda，毕竟它是 FaaS 理念的发起者。我们还会选择 Quarkus 作为实现框架，因为它目前提供了最小的部署大小。请注意，这里展示的配置可能需要一些调整或可能完全过时；始终审查构建和运行代码所需工具的最新版本。目前我们将使用 Quarkus 1.13.7。

使用 Quarkus 和 AWS Lambda 设置函数需要一个 [AWS 账号](https://aws.amazon.com)，在您的系统上安装 [AWS CLI](https://oreil.ly/0dYrb)，以及如果您想要运行本地测试，还需要安装 [AWS Serverless Application Model (SAM) CLI](https://oreil.ly/h7gdD)。

一旦你搞定了这些，下一步就是启动项目，我们会倾向于像以前一样使用 [Quarkus](https://code.quarkus.io)，但是函数项目需要不同的设置。所以最好切换到使用 Maven 原型：

```java
mvn archetype:generate \
    -DarchetypeGroupId=io.quarkus \
    -DarchetypeArtifactId=quarkus-amazon-lambda-archetype \
    -DarchetypeVersion=1.13.7.Final
```

在交互模式下调用此命令将询问您一些问题，例如项目的组、artifact、版本（GAV）坐标和基础包。对于这个演示，让我们使用以下配置：

+   `groupId`: com.example.demo

+   `artifactId`: demo

+   `version`: 1.0-SNAPSHOT（默认值）

+   `package`: com.example.demo（与 `groupId` 相同）

这将导致一个适合构建、测试和部署到 AWS Lambda 的 Quarkus 项目的项目结构。原型创建了 Maven 和 Gradle 的构建文件，但是我们现在不需要后者；它还创建了三个函数类，但我们只需要一个。我们的目标是拥有类似于这个的文件结构：

```java
.
├── payload.json
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── example
    │   │           └── demo
    │   │               ├── GreetingLambda.java
    │   │               ├── InputObject.java
    │   │               ├── OutputObject.java
    │   │               └── ProcessingService.java
    │   └── resources
    │       └── application.properties
    └── test
        ├── java
        │   └── com
        │       └── example
        │           └── demo
        │               └── LambdaHandlerTest.java
        └── resources
            └── application.properties
```

函数的要点是使用 `InputObject` 类型捕获输入，使用 `ProcessingService` 类型处理它们，然后将结果转换为另一种类型（`OutputObject`）。`GreetingLambda` 类型将所有内容整合在一起。首先让我们先看一下输入和输出类型——毕竟，它们只是简单的数据类型，没有任何逻辑：

```java
package com.example.demo;

public class InputObject {
    private String name;
    private String greeting;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getGreeting() {
        return greeting;
    }

    public void setGreeting(String greeting) {
        this.greeting = greeting;
    }
}
```

Lambda 函数期望接收两个输入值：问候语和姓名。我们马上会看到它们如何被处理服务转换：

```java
package com.example.demo;

public class OutputObject {
    private String result;
    private String requestId;

    public String getResult() {
        return result;
    }

    public void setResult(String result) {
        this.result = result;
    }

    public String getRequestId() {
        return requestId;
    }

    public void setRequestId(String requestId) {
        this.requestId = requestId;
    }
}
```

输出对象包含转换后的数据和请求 ID 的引用。我们将使用这个字段来展示如何从运行上下文获取数据。

好了，接下来是处理服务；这个类负责将输入转换为输出。在我们的情况下，它将两个输入值连接成一个字符串，如下所示：

```java
package com.example.demo;

import javax.enterprise.context.ApplicationScoped;

@ApplicationScoped
public class ProcessingService {
    public OutputObject process(InputObject input) {
        OutputObject output = new OutputObject();
        output.setResult(input.getGreeting() + " " + input.getName());
        return output;
    }
}
```

剩下的就是查看`GreetingLambda`，用于组装函数本身的类型。这个类需要实现由 Quarkus 提供的已知接口，其依赖应该已经在使用原型创建的*pom.xml*文件中配置好了。这个接口使用输入和输出类型参数化。幸运的是，我们已经有了这些。每个 Lambda 必须有一个唯一的名称，并且可以访问其运行上下文，如下所示：

```java
package com.example.demo;

import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestHandler;

import javax.inject.Inject;
import javax.inject.Named;

@Named("greeting")
public class GreetingLambda
    implements RequestHandler<InputObject, OutputObject> {
    @Inject
    ProcessingService service;

    @Override
    public OutputObject handleRequest(InputObject input, Context context) {
        OutputObject output = service.process(input);
        output.setRequestId(context.getAwsRequestId());
        return output;
    }
}
```

所有的部分都应该得到合理安排。Lambda 定义输入和输出类型并调用数据处理服务。为了演示目的，此示例展示了依赖注入的使用，但您可以通过将`ProcessingService`的行为移到`GreetingLambda`中来减少代码量。我们可以通过运行本地测试命令`mvn test`快速验证代码，或者如果您喜欢`mvn verify`，因为它还会打包函数。

请注意，当函数被打包时，附加文件被放置在*target*目录中，特别是一个名为*manage.sh*的脚本，该脚本依赖于 AWS CLI 工具，用于在与您的 AWS 帐户关联的目标位置创建、更新和删除函数。支持这些操作需要其他文件：

function.zip

包含二进制位的部署文件

sam.jvm.yaml

使用 AWS SAM CLI 进行本地测试（Java 模式）

sam.native.yaml

使用 AWS SAM CLI 进行本地测试（本地模式）

下一步需要您配置一个*执行角色*，最好参考 [AWS Lambda 开发指南](https://oreil.ly/97ACL)，以防流程已经更新。该指南将向您展示如何配置 AWS CLI（如果您尚未执行此操作）并创建一个必须添加为运行 shell 的环境变量的执行角色。例如：

```java
LAMBDA_ROLE_ARN="arn:aws:iam::1234567890:role/lambda-ex"
```

在这种情况下，`1234567890`代表您的 AWS 帐户 ID，`lambda-ex`是您选择的角色名称。我们可以继续执行函数，对于这个函数，我们有两种模式（Java、本地）和两种执行环境（本地、生产）；让我们首先处理两种环境的 Java 模式，然后再处理本地模式。

在本地环境中运行函数需要使用 Docker 守护程序，现在应该是开发人员工具箱中的常见组成部分；我们还需要使用 AWS SAM CLI 来驱动执行。记住在*target*目录中找到的附加文件集合吗？我们将使用*sam.jvm.yaml*文件以及在项目启动时由原型创建的另一个文件*payload.json*。位于目录根目录，其内容应如下所示：

```java
{
  "name": "Bill",
  "greeting": "hello"
}
```

该文件为函数接受的输入定义了值。鉴于函数已经打包，我们只需调用它，如下所示：

```java
$ sam local invoke --template target/sam.jvm.yaml --event payload.json
Invoking io.quarkus.amazon.lambda.runtime.QuarkusStreamHandler::handleRequest
(java11)
Decompressing /work/demo/target/function.zip
Skip pulling image and use local one:
amazon/aws-sam-cli-emulation-image-java11:rapid-1.24.1.

Mounting /private/var/folders/p_/3h19jd792gq0zr1ckqn9jb0m0000gn/T/tmppesjj0c8 as
/var/task:ro,delegated inside runtime container
START RequestId: 0b8cf3de-6d0a-4e72-bf36-232af46145fa Version: $LATEST
__  ____  __  _____   ___  __ ____  ______
 --/ __ \/ / / / _ | / _ \/ //_/ / / / __/
 -/ /_/ / /_/ / __ |/ , _/ ,< / /_/ /\ \
--\___\_\____/_/ |_/_/|_/_/|_|\____/___/
[io.quarkus] (main) quarkus-lambda 1.0-SNAPSHOT on
JVM (powered by Quarkus 1.13.7.Final) started in 2.680s.
[io.quarkus] (main) Profile prod activated.
[io.quarkus] (main) Installed features: [amazon-lambda, cdi]
END RequestId: 0b8cf3de-6d0a-4e72-bf36-232af46145fa
REPORT RequestId: 0b8cf3de-6d0a-4e72-bf36-232af46145fa	Init Duration: 1.79 ms
Duration: 3262.01 ms Billed Duration: 3300 ms
Memory Size: 256 MB	Max Memory Used: 256 MB
{"result":"hello Bill","requestId":"0b8cf3de-6d0a-4e72-bf36-232af46145fa"}
```

此命令将拉取一个适合运行该函数的 Docker 镜像。请注意报告的值，这可能会因您的设置而异。在我的本地环境中，此函数的执行将花费我 3.3 秒，并占用 256 MB 的内存。这可以让您了解在运行系统作为一组函数时，您将被计费多少。然而，本地环境与生产环境不同，因此让我们将该函数部署到真实环境中。我们将使用 *manage.sh* 脚本来完成这一壮举，通过调用以下命令：

```java
$ sh target/manage.sh create
$ sh target/manage.sh invoke
Invoking function
++ aws lambda invoke response.txt --cli-binary-format raw-in-base64-out
++ --function-name QuarkusLambda --payload file://payload.json
++ --log-type Tail --query LogResult
++ --output text base64 --decode
START RequestId: df8d19ad-1e94-4bce-a54c-93b8c09361c7 Version: $LATEST
END RequestId: df8d19ad-1e94-4bce-a54c-93b8c09361c7
REPORT RequestId: df8d19ad-1e94-4bce-a54c-93b8c09361c7	Duration: 273.47 ms
Billed Duration: 274 ms	Memory Size: 256 MB
Max Memory Used: 123 MB	Init Duration: 1635.69 ms
{"result":"hello Bill","requestId":"df8d19ad-1e94-4bce-a54c-93b8c09361c7"}
```

正如您所见，计费时长和内存使用量都减少了，这对我们的钱包来说是好事，尽管初始化时长增加到了 1.6，这会延迟响应，增加整个系统的总执行时间。让我们看看当我们从 Java 模式切换到本机模式时，这些数字会如何变化。您可能还记得，Quarkus 允许您将项目打包为本机可执行文件，但请记住 Lambda 需要 Linux 可执行文件，因此如果您恰好在非 Linux 环境上运行，则需要调整打包命令。以下是需要完成的操作：

```java
# for linux
$ mvn -Pnative package

# for non-linux
$ mvn package -Pnative -Dquarkus.native.container-build=true \
 -Dquarkus.native.container-runtime=docker
```

第二个命令在 Docker 容器内调用构建，并将生成的可执行文件放置在预期位置，而第一个命令则按原样执行构建。现在，有了本地可执行文件，我们可以在本地和生产环境中执行新函数。先看看本地环境：

```java
$ sam local invoke --template target/sam.native.yaml --event payload.json
Invoking not.used.in.provided.runtime (provided)
Decompressing /work/demo/target/function.zip
Skip pulling image and use local one:
amazon/aws-sam-cli-emulation-image-provided:rapid-1.24.1.

Mounting /private/var/folders/p_/3h19jd792gq0zr1ckqn9jb0m0000gn/T/tmp1zgzkuhy as
/var/task:ro,delegated inside runtime container
START RequestId: 27531d6c-461b-45e6-92d3-644db6ec8df4 Version: $LATEST
__  ____  __  _____   ___  __ ____  ______
 --/ __ \/ / / / _ | / _ \/ //_/ / / / __/
 -/ /_/ / /_/ / __ |/ , _/ ,< / /_/ /\ \
--\___\_\____/_/ |_/_/|_/_/|_|\____/___/
[io.quarkus] (main) quarkus-lambda 1.0-SNAPSHOT native
(powered by Quarkus 1.13.7.Final) started in 0.115s.
[io.quarkus] (main) Profile prod activated.
[io.quarkus] (main) Installed features: [amazon-lambda, cdi]
END RequestId: 27531d6c-461b-45e6-92d3-644db6ec8df4
REPORT RequestId: 27531d6c-461b-45e6-92d3-644db6ec8df4	Init Duration: 0.13 ms
Duration: 218.76 ms	Billed Duration: 300 ms Memory Size: 128 MB
Max Memory Used: 128 MB
{"result":"hello Bill","requestId":"27531d6c-461b-45e6-92d3-644db6ec8df4"}
```

计费时长降低了一个数量级，从 3300 毫秒降至仅 300 毫秒，并且使用的内存减半；与其 Java 对应物相比，这看起来很有希望。在生产环境中运行时，我们会得到更好的数据吗？让我们来看看：

```java
$ sh target/manage.sh native create
$ sh target/manage.sh native invoke
Invoking function
++ aws lambda invoke response.txt --cli-binary-format raw-in-base64-out
++ --function-name QuarkusLambdaNative
++ --payload file://payload.json --log-type Tail --query LogResult --output text
++ base64 --decode
START RequestId: 19575cd3-3220-405b-afa0-76aa52e7a8b5 Version: $LATEST
END RequestId: 19575cd3-3220-405b-afa0-76aa52e7a8b5
REPORT RequestId: 19575cd3-3220-405b-afa0-76aa52e7a8b5	Duration: 2.55 ms
Billed Duration: 187 ms Memory Size: 256 MB	Max Memory Used: 54 MB
Init Duration: 183.91 ms
{"result":"hello Bill","requestId":"19575cd3-3220-405b-afa0-76aa52e7a8b5"}
```

总计的计费时长结果提高了 30%，内存使用量不到以前的一半；但真正的赢家是初始化时间，大约是以前时间的 10%。在本机模式下运行您的函数会导致启动更快，各方面的数据更好。

现在轮到您决定哪种组合选项会给您带来最佳结果了。有时，即使是在生产环境中保持 Java 模式也足够好，或者一直采用本机模式可能会为您带来优势。无论哪种方式，测量都很关键—不要猜测！

# 总结

在本章中，我们涵盖了很多内容，从传统的单体架构开始，将其分解为具有可重用组件的较小部分，这些部分可以独立部署，称为微服务，一直到最小的部署单元：函数。沿途会出现权衡，因为微服务架构本质上更复杂，由于其由更多运动部分组成。网络延迟成为一个真正的问题，必须相应地加以解决。其他方面，如数据事务，由于其跨服务边界的跨度可能不同，取决于情况，会变得更加复杂。使用 Java 和本地可执行模式产生不同的结果，并且需要定制设置，每种都有其优缺点。我亲爱的读者，我的建议是评估、测量，然后选择一种组合；随时注意数字和服务级别协议（SLAs），因为您可能需要重新评估决策并进行调整。

表 4-1 总结了在我的本地环境和远程环境中以 Java 和本地映像模式运行示例应用程序时，每个候选框架的测量结果。大小列显示部署单元大小，而时间列描绘了从启动到第一个请求的时间。

表 4-1\. 测量摘要

| 框架 | Java - 大小 | Java - 时间 | 本地 - 大小 | 本地 - 时间 |
| --- | --- | --- | --- | --- |
| Spring Boot | 17 MB | 2200 ms | 78 MB | 90 ms |
| Micronaut | 14 MB | 500 ms | 60 MB | 20 ms |
| Quarkus | 13 MB | 600 ms | 47 MB | 13 ms |
| Helidon | 15 MB | 2300 ms | 94 MB | 50 ms |

作为提醒，鼓励您进行自己的测量。对于托管环境、JVM 版本和设置、框架版本、网络条件和其他环境特征的更改将产生不同的结果。显示的数字应该持保留态度，永远不要视为权威值。
