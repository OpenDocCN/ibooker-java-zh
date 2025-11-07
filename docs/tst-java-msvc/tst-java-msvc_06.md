## 第七章. 端到端测试

*本章涵盖*

+   微服务应用的端到端测试

+   端到端测试的工具

+   为端到端测试设置 Arquillian

端到端测试建立在集成测试之上，而集成测试又建立在所有其他你已经了解的测试形式之上。正如其名所示，端到端测试旨在从开始到结束（或者如果你更喜欢，从上到下）测试你的应用程序。理论上，这些测试应该模拟你的应用程序的真实世界用户，或者至少执行真实用户的操作。在实践中，这些测试通常是最难编写的，并且消耗最多的开发时间。与其他类型的测试相比，端到端测试几乎总是较慢，因此它们通常与常规的开发过程隔离——例如，在 Jenkins 构建服务器上（[`jenkins.io`](https://jenkins.io)）。

| |
| --- |

##### 注意

学习有关持续集成和交付的所有内容超出了本书的范围，所以我们认为可能还有另一本小书在计划中。（对于那些熟悉这个主题的人来说，请原谅这个双关语。）话虽如此，为了让你开始，第十章提供了一个关于如何使用 Jenkins 设置部署管道的相当详细的讨论。

| |
| --- |

端到端测试理想情况下应提供一个尽可能接近你的生产环境的环境，同时又是隔离的，这样它就不会损害实际系统。这可能意味着为画廊应用程序提供简单的图像目录副本，或者为数据仓库应用程序提供企业数据库的快照等复杂的东西。有时这可能是不可能的——例如，拥有一个真实的 SAP 端点进行测试可能是一个过高的开销，你希望避免；因此，使用一个模拟的、一个模拟的或 WireMock 将是一个合法的解决方案。

### 7.1. 端到端测试在整体测试图景中的位置

端到端测试是必需的，以确保从前端入口到后端正确接收输入。真正的挑战是如何将所有独立的微服务在单个机器上串联起来。本书的例子中有四个微服务，其中一个是 Spring Boot 应用程序，还有一个调用这些服务的 UI 应用程序。为了测试 UI，微服务必须在 UI 启动之前可用。之后，你需要执行必要的测试，记录和收集结果，然后关闭并清理环境。从某种意义上说，你是在将单体应用程序重新组装起来以进行测试。

单个微服务也可能需要所有依赖的服务，如数据库，都处于运行状态并已填充数据。正如你可以想象的那样，这个需求列表可以无限延伸。

也可能需要重复这个过程进行进一步的测试，但我们建议你尽量保持一切运行直到所有测试完成。尽可能地将测试批量进行，因为重启环境在时间上代价很高。

### 7.2\. 端到端测试技术

从原则上讲，有两种类型的端到端测试：*水平*和*垂直*。我们将在稍后描述它们。这两种类型都可以在白盒或黑盒环境中执行，或者结合使用。*白盒环境*是指测试应用程序的可见或外部元素，而*黑盒环境*测试的是实际的功能（在后台）。

为了将这个概念应用到实际情境中：假设你有一个最终用户必须与之交互的 UI。用户可以可视化和对暴露的应用程序执行操作。这些操作导致用户对结果的期望。从逻辑上讲，为了模拟用户与 UI 的交互，你必须为测试提供 UI。这是一个白盒环境，因为用户可以看到操作的发生。

在用户与 UI 交互的白盒场景的同一范围内，操作可能会在底层服务器上调用过程。这些后端过程对用户是不可见的，但可能会产生用户最终会遇到的结果。这是一个黑盒环境，因为操作是在黑暗中进行的，换句话说。

端到端测试通常结合了白盒和黑盒环境，尤其是在基于浏览器的应用程序中。白盒是浏览器：你可以看到，并且可能影响测试。黑盒是服务器：你的操作发送一个请求，这个请求被无形地处理，然后返回一个响应。

#### 7.2.1\. 垂直测试

*垂直*端到端测试旨在测试应用程序展示的功能深度。对于一个用户界面（UI），这意味着在用户执行操作时，从一个视图转换到另一个视图之前，需要执行正确的验证。这种验证可能需要特定的用户权限在 LDAP 中，例如，从数据库检索正确的设置。

你基本上是在上下查看你所看到的，并确保一切井然有序。所有元素都应该根据你指定的环境存在，如图图 7.1 所示。

##### 图 7.1\. 一个简单的白盒垂直测试

![](img/07fig01.jpg)

该图可能看起来很简单，但执行这些测试很重要。你的应用程序的用户有相关的用户权限；如果由于你的代码中对用户权限的错误解释，搜索按钮没有被渲染或输入文本框被禁用，用户会如何反应？

#### 7.2.2\. 水平测试

*横向*端到端测试旨在测试应用的全范围。对于一个 UI，这意味着测试用户执行操作时，一个视图过渡到另一个视图。垂直测试确保你准备好通过验证进行过渡；横向测试则检查操作是否发生，以及结果是否符合预期。

这里，你从左到右寻找从一个视图到下一个视图的正确过渡。你的操作是否导致了正确的预期？或者，换句话说，是否处理了无效操作（负面测试）？图 7.2 展示了示例。

##### 图 7.2\. 黑盒和白盒横向测试

![图片](img/07fig02.jpg)

白盒操作向黑盒服务器发送请求，服务器随后返回一个作为列表渲染的响应。你需要测试过渡并确保列表显示且内容正确。

### 7.3\. 端到端测试工具简介

端到端测试在单体应用中编写起来非常困难和复杂。在微服务架构中，编写此类测试更加复杂，所以任何能帮助你的是个加分项。幸运的是，一系列优秀的工具和（你猜对了）Arquillian 扩展可用以帮助你完成这项任务。让我们看看其中的一些。

#### 7.3.1\. Arquillian Cube

*Arquillian Cube* ([`arquillian.org/arquillian-cube/`](http://arquillian.org/arquillian-cube/)) 是 Arquillian 测试框架的一个扩展，它使得管理 Docker 环境中的容器成为可能。你可以使用这个扩展来部署一个设计的 Docker 镜像，并对其进行测试或在其中测试。镜像上托管的环境可以像你希望的那样简单或复杂。因此，而不是开发者需要在测试代码中了解并尝试提供对所有协作者（如数据库或其他服务）的访问，你只需将开发者的测试发送到一个已经一切准备就绪的镜像——只有 DevOps 需要担心由镜像提供的不断变化的环境。

Docker 镜像通常应该托管一个应用服务器。Arquillian 按照常规方式打包你的应用，并将 WAR 或 EAR 文件发布到托管服务器。这与第 4.1 节中描述的生命周期相同；唯一的区别在于，这里不是部署到本地应用服务器，而是部署到托管环境中的服务器。

这是一个重要的主题，您将在第八章（kindle_split_017_split_000.xhtml#ch08）中找到您需要了解的所有内容，我们将讨论 Docker。现在，我们将专注于 *基本* 的端到端单元测试——这可能在术语上似乎有些矛盾，因为仍然有很多东西需要整合。我们按此顺序介绍内容，以便您了解构建环境所涉及到的挑战，并可以看到可能出错的地方。在您踩下油门之前，了解引擎盖下发生的事情会更好。

#### 7.3.2\. Arquillian Drone

*Arquillian Drone* ([`arquillian.org/arquillian-extension-drone/`](http://arquillian.org/arquillian-extension-drone/)) 是 Arquillian 测试框架的一个扩展，它使得访问知名的 Selenium WebDriver ([`seleniumhq.github.io/docs`](https://seleniumhq.github.io/docs)) 成为可能，而 Selenium WebDriver 又被用于浏览器自动化。在测试基于 Web 的 UI 时，浏览器自动化是一个关键需求。它使得测试能够模拟真实用户浏览您的应用程序并输入或操作数据的行为。

为什么您要使用这个扩展，如果它只是 WebDriver 的包装器呢？好吧，任何使用裸 WebDriver API 编写测试的人都会很快告诉你，即使是相对简单的测试，也需要大量的样板代码。Drone 隐藏了大部分这些样板代码，让您能够专注于编写测试的核心部分。仍然有一些设置需要执行，但我们在本章后面的示例中会介绍这些内容。

#### 7.3.3\. Arquillian Graphene 2

*Arquillian Graphene 2* ([`github.com/arquillian/arquillian-graphene`](https://github.com/arquillian/arquillian-graphene)) 是（正如其名称所示）第二代快速开发扩展，旨在补充 Selenium WebDriver。尽管它可以用来创建独立的 AJAX 启用测试，但 Graphene 与 Arquillian Drone 扩展一起使用效果最佳。

#### 7.3.4\. JMeter

*JMeter* ([`jmeter.apache.org`](http://jmeter.apache.org)) 是 Apache 软件基金会的一个项目，可以用来对几乎所有类型的端点进行负载测试。它是一个完全基于 Java 的解决方案，因此可以在所有支持的平台上使用。它能够模拟大量的网络流量，主要用于测试应用端点的弹性。您将使用它来创建一些简单的压力测试，以确保您的服务能够承受一定的负载。

#### 7.3.5\. Cukes in Space

*Cukes in Space* ([`github.com/cukespace/cukespace`](https://github.com/cukespace/cukespace)) 是一个 Arquillian 扩展，允许您使用常见的 Given-When-Then 规范在 Cucumber JVM ([`cucumber.io/docs/reference`](https://cucumber.io/docs/reference)) 上运行测试。

### 7.4\. 示例端到端测试

现在您已经有了工具，是时候进行一个示例端到端测试了。正如我们提到的，这样的测试很复杂，这个也不例外。您正在使用各种技术进行演示微服务应用程序，所以您必须在端到端测试中处理这种额外的复杂性。好处是您可以看到一系列解决方案，这应该有助于您在未来开发自己的测试。这里没有硬性规则——您使用您拥有的工具，以获得您需要的成果。现在放手一搏。

我们使用一个简单的应用程序作为前端 UI，以突出显示端到端过程；这里没有惊喜。在源代码根目录下打开命令行，并运行以下命令：

```
cd web
mvn clean install -DskipTests
```

这将构建 Web 应用程序并确保所有必需的依赖项都可用并缓存。我们在这里跳过测试，因为我们想首先详细解释它。

#### 7.4.1. 构建微服务

您需要做的第一件事是确保您迄今为止看到的所有示例微服务代码都已构建并准备好供您使用。您将使用这些项目生成的真实 WAR 和 JAR 文件来创建一个更真实的端到端测试。

在源代码根目录下打开命令行，并运行以下命令：

```
cd comments
./gradlew war -x test

cd ../aggregator
./gradlew war -x test

cd ../game
mvn install -DskipTests

cd ../video
./gradlew build -x test
```

| |
| --- |

##### 注意

您在前面章节中彻底测试了所有微服务应用程序，所以在这里为了简洁起见，您将跳过测试。此外，这些项目中的某些测试被设计为失败，以突出某个点或提供用户练习。

| |
| --- |

#### 7.4.2. 添加构建依赖项和配置

接下来，您需要将相关的物料清单（BOM）导入添加到您的 Web UI 应用程序的构建脚本`dependencyManagement`部分（在 code/web/pom.xml 中）。只有一个新的导入项：`arquillian-drone-bom`工件。这个新的 BOM 确保 Drone 所需的所有依赖项都可用。

##### 列表 7.1. 添加`arquillian-drone-bom`工件

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.jboss.arquillian</groupId>
            <artifactId>arquillian-bom</artifactId>
            <version>${version.arquillian-bom}</version>
            <scope>import</scope>
            <type>pom</type>
        </dependency>
        <dependency>
            <groupId>org.jboss.arquillian.extension</groupId>
            <artifactId>arquillian-drone-bom</artifactId>
            <version>2.0.0.Final</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

UI 应用程序需要`graphene-webdriver`工件来注入 Selenium WebDriver。这个 WebDriver 是与浏览器通信以执行操作的东西。

##### 列表 7.2. 添加`graphene-webdriver`工件

```
<dependency>
    <groupId>org.jboss.arquillian.graphene</groupId>
    <artifactId>graphene-webdriver</artifactId>
    <version>2.1.0.Final</version>
    <type>pom</type>
    <scope>test</scope>
</dependency>
```

##### 安装自动化浏览器驱动程序

您需要确保为测试浏览器安装了适当的浏览器驱动程序。我们使用 Chrome 浏览器作为默认的测试浏览器，因此请确保 ChromeDriver 二进制文件（[`mng.bz/VZig`](http://mng.bz/VZig)）已下载并且可以在测试配置中对 Arquillian 可访问。这定义在`webdriver`扩展元素中，使用项目`arquillian.xml`文件中的`chromeDriverBinary`属性。在撰写本文时，您应该能够从[www.seleniumhq.org/download](http://www.seleniumhq.org/download)找到支持浏览器的大量当前驱动程序。

##### 列表 7.3. 添加浏览器驱动程序

```
<extension qualifier="webdriver">
    <property name="browser">${browser:chrome}</property>            *1*
    <!--https://sites.google.com/a/chromium.org/chromedriver/-->
    <property name="chromeDriverBinary">/home/andy/dev/chromedriver
     </property>                                                   *2*
</extension>
```

+   ***1* 浏览器属性，默认值为 chrome**

+   ***2* 指向驱动二进制的 chromeDriverBinary 属性**

| |
| --- |

##### 提示

总是会有很多关于使用不同测试浏览器的信息在网上。理想的解决方案是为您希望测试的每个浏览器定义不同的构建配置文件，在运行时覆盖`browser`属性。

| |
| --- |

##### 使用和定义 NoSQL 数据库

一些有用的依赖项是`de.flapdoodle.embed.mongo`和`embedded-redis`工件（code/web/pom.xml）

##### 列表 7.4\. 添加依赖项

```
<dependency>
    <groupId>de.flapdoodle.embed</groupId>
    <artifactId>de.flapdoodle.embed.mongo</artifactId>
    <version>2.0.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>com.github.kstyrc</groupId>
    <artifactId>embedded-redis</artifactId>
    <version>0.6</version>
</dependency>
```

| |
| --- |

##### 警告

有可能向构建脚本添加各种未定义的神奇插件以确保这些运行时依赖项启动。这**不是**推荐的，因为测试将不再自包含，并且无法由 IDE 直接运行。

| |
| --- |

列表 7.4 中的整洁库能够在测试中直接启动 MongoDB 实例和 Redis 实例，相对开销很小。由于 JUnit 测试的整体生命周期管理，`@BeforeClass`通常不足以启动这些需求。实现 JUnit 规则允许您在测试生命周期中更深入地包含它们。规则中的变量用于保持对进程的引用，以便在测试完成后进行清理。您将在稍后定义的规则中使用的库将为您下载并初始化 MongoDB 和 Redis 实例。您无需手动准备任何东西，并且您的测试保持自包含。

下面的列表（code/web/src/test/java/book/web/rule/MongodRule.java）中展示的简单 Mongod 规则启动了一个绑定到提供的宿主和端口的 Mongod 实例。对于您的测试，您不需要更多，但库 API 允许进行完整的配置。如果您需要，可以轻松修改规则以接受更多参数。

##### 列表 7.5\. Mongod 规则

```
package book.web.rule;

import de.flapdoodle.embed.mongo.MongodExecutable;
import de.flapdoodle.embed.mongo.MongodProcess;
import de.flapdoodle.embed.mongo.MongodStarter;
import de.flapdoodle.embed.mongo.config.MongodConfigBuilder;
import de.flapdoodle.embed.mongo.config.Net;
import de.flapdoodle.embed.mongo.distribution.Version;
import de.flapdoodle.embed.process.runtime.Network;
import org.junit.rules.ExternalResource;

import java.io.IOException;

public class MongodRule extends ExternalResource {

    private final MongodStarter starter
            = MongodStarter.getDefaultInstance();
    private MongodExecutable mongodExe;
    private MongodProcess mongodProcess;
    private String host;
    private int port;

    public MongodRule(String host, int port) {
        this.host = host;
        this.port = port;
    }

    @Override
    protected void before() throws Throwable {                          *1*
        try {
            mongodExe = starter.prepare(new MongodConfigBuilder()
                    .version(Version.Main.PRODUCTION)
                    .net(new Net(this
                    .host, this.port, Network.localhostIsIPv6())).build());
            mongodProcess = mongodExe.start();
        } catch (final IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    protected void after() {                                            *2*
        //Stop MongoDB
        if (null != mongodProcess) {
            mongodProcess.stop();
        }
        if (null != mongodExe) {
            mongodExe.stop();
        }
    }
}
```

+   ***1* 在准备阶段创建并执行 Mongod 进程。**

+   ***2* 在清理阶段确保进程被终止并清理。**

Redis 规则基本上与（code/web/src/test/java/book/web/rule/RedisRule.java）相同，但它使用库提供的`RedisServer`。

##### 列表 7.6\. Redis 规则

```
package book.web.rule;

import org.junit.rules.ExternalResource;
import redis.embedded.RedisServer;

public class RedisRule extends ExternalResource {

    private RedisServer redisServer;
    private int port;

    public RedisRule(int port) {
        this.port = port;
    }

    @Override
    protected void before() throws Throwable {
        try {
            redisServer = new RedisServer(this.port);
            redisServer.start();
        } catch (final Throwable e) {
            e.printStackTrace();
        }
    }

    @Override
    protected void after() {
        //Stop Redis
        if (null != redisServer) {
            redisServer.stop();
        }
    }
}
```

| |
| --- |

##### 注意

向 Krzysztof Styrc ([`github.com/kstyrc`](https://github.com/kstyrc)) 和 Michael Mosmann ([`github.com/michaelmosmann`](https://github.com/michaelmosmann)) 及他们的团队致敬，感谢他们提供这些优秀的开源项目！

| |
| --- |

我们希望您能看出，使用 JUnit 规则机制来包装各种测试资源非常简单。正如我们提到的，您将在后面的示例测试中使用这些规则。

##### 提供微服务运行时环境

我们为这个示例测试提供的首要运行时环境是 Apache TomEE ([`tomee.apache.org`](http://tomee.apache.org))。你可以选择任何 Java EE 环境来部署你的 WAR 文件，只要它是 EE 兼容的。你还有一个 Spring Boot 应用程序和一个 WildFly fat JAR，但稍后你将处理这些。

Apache TomEE 提供了一个 Maven 插件，允许你在目标目录中自动创建一个可运行的 server 目录 gamerwebapp。这是一个完整的 TomEE 服务器发行版，由插件下载并解压。每次运行构建时，此过程都会附加到 Maven *validate*阶段。将以下代码添加到 code/web/pom.xml 中。

##### 列表 7.7\. 添加 Apache TomEE 运行时环境

```
<plugin>
    <groupId>org.apache.tomee.maven</groupId>
    <artifactId>tomee-maven-plugin</artifactId>
    <version>${version.tomee}</version>
    <configuration>
        <catalinaBase>target/gamerwebapp</catalinaBase>
        <tomeeClassifier>plus</tomeeClassifier>
        <deployOpenEjbApplication>true</deployOpenEjbApplication>
        <removeDefaultWebapps>true</removeDefaultWebapps>
        <removeTomeeWebapp>true</removeTomeeWebapp>
    </configuration>
    <executions>
        <execution>
            <id>gamerwebapp</id>
            <phase>validate</phase>
            <configuration>
                <attach>false</attach>
                <zip>false</zip>
            </configuration>
            <goals>
                <goal>build</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

有一个注意事项：插件在 Maven 构建阶段将 TomEE 服务器下载到项目的一个本地目录。如果你只是运行构建脚本，这将是可行的。但如果你想在 IDE 中调试测试，你需要至少运行一次构建脚本：

```
cd web
mvn clean validate -DskipTests
```

第一次运行此操作时，需要一段时间，因为 TomEE 需要下载并解压到项目本地目录。后续构建将快得多。

然后你进行一点 Maven 魔法，使用`<artifactId>maven-resources-plugin</artifactId>`来创建之前创建的 gamerwebapp 目录的多个副本。以下代码会为每个你想要部署为独立微服务 WAR 文件的微服务 WAR 文件重复。再次提醒，打开你的 IDE 中的 pom.xml 文件以获取完整信息。

##### 列表 7.8\. 将 target/gamerwebapp 复制到 target/commentsservice

```
<execution>
    <id>create-commentsservice</id>
    <phase>validate</phase>
    <goals>
        <goal>copy-resources</goal>
    </goals>
    <configuration>
        <outputDirectory>target/commentsservice</outputDirectory>
        <includeEmptyDirs>true</includeEmptyDirs>
        <resources>
            <resource>
                <directory>target/gamerwebapp</directory>
                <filtering>false</filtering>
            </resource>
        </resources>
    </configuration>
</execution>
```

##### 在 arquillian.xml 中添加一个组

你已经知道你可以在 arquillian.xml 配置文件中定义多个容器。这对于提供由 Arquillian 框架支持的多个微服务环境来说非常完美。

##### 列表 7.9\. 定义一个组

```
    <group qualifier="tomee-cluster" default="true">                *1*
    <container qualifier="gamerweb" default="true">                 *2*
        <configuration>
            <property name="httpPort">8080</property>               *3*
            <property name="stopPort">-1</property>
            <property name="ajpPort">-1</property>
            <property name="classifier">plus</property>
            <property name="appWorkingDir">target/gamerweb_work
     </property>
            <property name="dir">target/gamerwebservice
     </property>                                                  *4*
        </configuration>
    </container>
    <container qualifier="commentsservice" default="false">
        <configuration>
            <property name="httpPort">8282</property>
            <property name="stopPort">-1</property>
            <property name="ajpPort">-1</property>
            <property name="classifier">plus</property>
            <property name="appWorkingDir">target/
     commentsservice_work</property>
            <property name="dir">target/commentsservice
     </property>
        </configuration>
    </container>
    <container qualifier="gameaggregatorservice" default="false">
        <configuration>
            <property name="httpPort">8383</property>
            <property name="stopPort">-1</property>
            <property name="ajpPort">-1</property>
            <property name="classifier">plus</property>
            <property name="appWorkingDir">target/
     gameaggregatorservice_work</property>
            <property name="dir">target/gameaggregatorservice
     </property>
            <property name="properties">
                com.sun.jersey.server.impl.cdi.
     lookupExtensionInBeanManager=true
            </property>
        </configuration>
    </container>
</group>
```

+   ***1* 组定义，在此标记为默认**

+   ***2* 定义一个具有唯一限定符的容器**

+   ***3* 容器特定的属性，用于定义 HTTP 端口和协议端口。TomEE 使用-1 来指示应使用随机端口。**

+   ***4* 实际服务器目录的路径（如之前在 pom.xml 中定义/创建的）**

如果你还在担心 Spring Boot 和 WildFly 微服务，不要担心。你几乎完成了；只需再有一个部分。

#### 7.4.3\. 在测试中添加@Deployment 和@TargetsContainer

测试类正在变得庞大——它有很多事情要做。我们将关注每个核心元素。你可以打开你选择的 IDE 中的测试以获取完整信息，但你已经在之前的章节中看到了很多。在示例范围之外，我们建议将大部分初始化和部署代码放在一个抽象类中，以便在其他测试中重用。

|  |
| --- |

##### 提示

如果可能，从一个微服务的 WAR 文件 ShrinkWrap 开始构建部署。然后向微服务所需的依赖项混合中添加`provided`范围的工件。这意味着测试尽可能接近真实的生产部署。

| |
| --- |

下一步你需要做的是确保**所有**你的微服务都对测试类可用。你将从简单的部署开始（code/web/src/test/java/book/web/EndToEndTest.java）。后面的章节将涵盖使用 JUnit 规则进行更复杂的部署。

##### 列表 7.10\. 添加`@Deployment`和`@TargetsContainer`

```
@Deployment(name = "commentsservice", testable = false)
@TargetsContainer("commentsservice")
public static Archive commentsservice() throws Exception {

        return ShrinkWrap.create(ZipImporter.class, "commentsservice.war")
         .importFrom(getFile
         ("comments/build/libs/commentsservice.war"))
         .as(WebArchive.class).addAsLibraries(Maven.resolver()
         .resolve("org.mongodb:mongodb-driver:3.2.2")
         .withTransitivity().as(JavaArchive.class)).addClass
         (MongoClientProvider.class)
         .addAsWebInfResource("test-web.xml", "web.xml")
         .addAsWebInfResource("test-resources.xml"
         , "resources.xml");
    }

    @Deployment(name = "gameaggregatorservice", testable = false)
    @TargetsContainer("gameaggregatorservice")
    public static Archive gameaggregatorservice() throws Exception {
        return ShrinkWrap
          .create(ZipImporter.class, "gameaggregatorservice.war")
          .importFrom(
          getFile("aggregator/build/libs/gameaggregatorservice.war"))
          .as(WebArchive.class).addAsLibraries(
          Maven.resolver().resolve("org.mongodb:mongodb-driver:3.2.2")
          .withTransitivity().as(JavaArchive.class))
                .addClass(MongoClientProvider.class)
                .addAsWebInfResource("test-web.xml", "web.xml")
                .addAsWebInfResource("test-resources.xml", "resources.xml");
    }

@Deployment(name = "gamerweb", testable = false)                 *1*
@TargetsContainer("gamerweb")                                    *2*
public static Archive gamerWebService() throws Exception {
    return ShrinkWrap.create(MavenImporter.class)
      .loadPomFromFile("pom.xml")
      .importBuildOutput().as(WebArchive
      .class).addAsWebInfResource("test-web.xml", "web.xml");
}
```

+   ***1* 定义一个唯一的部署名称，这在针对多个容器进行测试时很重要**

+   ***2* @TargetsContainer(“[name]”)确保应用程序被部署到在 arquillian.xml 中定义的指定容器中。**

在理论上，你可以在 arquillian.xml 文件中指定无限数量的容器，然后可以将它们绑定到测试中的无限数量的部署。

| |
| --- |

##### 注意

TomEE 插件包括选项，可以直接将`provided`范围的工件添加到服务器运行时 lib 目录中（毕竟这就是`provided`的含义）。有关更多信息，请参阅插件文档([`tomee.apache.org/maven/index.html`](http://tomee.apache.org/maven/index.html))。

| |
| --- |

#### 7.4.4\. 跨源资源共享

你可能已经注意到，为了允许从不同主机访问 RESTful 端点，你在多个微服务上启用了跨源资源共享（CORS）。你应该只在完全理解其影响后在自己的环境中这样做。但在测试环境中，尤其是在多个服务绑定到同一本地机器上的多个端口时，这通常是必要的。

CORS 仅在服务从不同于服务主机的不同主机接收请求的场景中是必需的：例如，一个独立的微服务。配置需要在服务主机上进行，以允许指定的主机消费可用的服务。

CORS 配置因不同的应用程序服务器而异，因此你需要检查你选择的服务器的相关文档。以下示例展示了 Apache TomEE 的宽松配置（code/web/src/test/resources/test-web.xml）。

##### 列表 7.11\. 为 TomEE 或 Tomcat 启用 CORS

```
<web-app 

         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
 http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" version="3.0">

    <filter>
        <filter-name>CorsFilter</filter-name>
        <filter-class>org.apache.catalina.filters.CorsFilter
            </filter-class>
        <async-supported>true</async-supported>
        <init-param>
            <param-name>cors.allowed.origins</param-name>
            <param-value>*</param-value>
        </init-param>
        <init-param>
            <param-name>cors.allowed.methods</param-name>
            <param-value>GET,POST,HEAD,OPTIONS,PUT</param-value>
        </init-param>
        <init-param>
            <param-name>cors.allowed.headers</param-name>
            <param-value>
                Content-Type,X-Requested-With,accept,Origin,
                Access-Control-Request-Method,Access-Control-Request-Headers
            </param-value>
        </init-param>
        <init-param>
            <param-name>cors.exposed.headers</param-name>
            <param-value>Access-Control-Allow-Origin,
            Access-Control-Allow-Credentials</param-value>
        </init-param>
        <init-param>
            <param-name>cors.support.credentials</param-name>
            <param-value>false</param-value>
        </init-param>
        <init-param>
            <param-name>cors.preflight.maxage</param-name>
            <param-value>10</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>CorsFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

</web-app>
```

你的大部分服务在生产中可能托管在同一个域中，并且不需要这样的宽松配置。

#### 7.4.5\. 使用@ClassRule 应对混合环境

在撰写本文时，Arquillian 无法默认在同一个运行时中混合不同的容器环境。当你需要在测试中使用这种情况时，这将会成为一个问题，因为你使用了多个环境。你可以创建自己的容器实现，它封装了多个环境，但这超出了本书的范围。

我们选择的解决方案简单，尽管有点冗长。你知道 Spring Boot 和 WildFly Swarm 项目会生成胖 JAR，并且这些 JAR 文件是可执行的。你也看到了如何定义 JUnit 规则来包装外部进程（Mongod 和 Redis）。有了这些知识，使用 JVM 的 `ProcessBuilder` 来执行这些服务并在规则中管理进程生命周期相对简单，如列表 7.12（code/web/src/test/java/book/web/rule/MicroserviceRule.java）所示。

但是，即使进程可能已经启动，你也不能在端点可访问之前使用该服务。为了解决这个问题，你可以在测试序列中使用一个简单的连接方法，等待指定端点的有效连接。

##### 列表 7.12\. code/web/src/test/java/book/web/rule/MicroserviceRule.java

```
package book.web.rule;

import okhttp3.OkHttpClient;
import okhttp3.Request;
import org.junit.Assert;
import org.junit.rules.ExternalResource;

import java.io.File;
import java.net.MalformedURLException;
import java.net.URL;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicBoolean;
import java.util.concurrent.atomic.AtomicReference;
import java.util.concurrent.locks.ReentrantLock;
import java.util.logging.Logger;

public class MicroserviceRule extends ExternalResource {

    private final Logger log =
        Logger.getLogger(MicroserviceRule.class.getName());

    private final ReentrantLock lock = new ReentrantLock();
    private final CountDownLatch latch = new CountDownLatch(1);
    private final AtomicBoolean poll = new AtomicBoolean(true);
    private final AtomicReference<URL> url =
        new AtomicReference<>();
    private File file;
    private String[] args;
    private ResolutionStrategy strategy =
        new DefaultJavaResolutionStrategy();
    private long time = 30;
    private TimeUnit unit = TimeUnit.SECONDS;

    public MicroserviceRule(URL url) {
        this.url.set(url);
    }

    public MicroserviceRule(String url) {
        try {
            this.url.set(new URL(url));
        } catch (MalformedURLException e) {
            throw new RuntimeException("Invalid URL: " + url, e);
        }
    }

    public MicroserviceRule withExecutableJar(File file,
        String... args) {                                                 *1*

        Assert.assertTrue("The file must exist and be readable: " + file,
            file.exists() && file.canRead());

        this.file = file;
        this.args = args;
        return this;
    }

    public MicroserviceRule withJavaResolutionStrategy(ResolutionStrategy
        strategy) {
        this.strategy = (null != strategy ? strategy : this.strategy);
        return this;
    }

    public MicroserviceRule withTimeout(int time, TimeUnit unit) {
        this.time = time;
        this.unit = unit;
        return this;
    }

    private Process process;

    @Override
    protected void before() throws Throwable {

        Assert.assertNotNull("The MicroserviceRule requires a
     valid jar file", this.file);
        Assert.assertNotNull("The MicroserviceRule requires a
     valid url", this.url.get());

        this.lock.lock();

        try {
            ArrayList<String> args = new ArrayList<>();
            args.add(this.strategy.getJavaExecutable().toString());       *2*
            args.add("-jar");
            args.add(this.file.toString());

            if (null != this.args) {
                args.addAll(Arrays.asList(this.args));
            }

            ProcessBuilder pb =
                new ProcessBuilder(args.toArray(new String[args.size()]));
            pb.directory(file.getParentFile());
            pb.inheritIO();
            process = pb.start();                                         *3*

            log.info("Started " + this.file);

            final Thread t = new Thread(() -> {
                if (MicroserviceRule.this.connect(
                    MicroserviceRule.this.url.get())) {                   *4*
                    MicroserviceRule.this.latch.countDown();
                }
            }, "Connect thread :: " + this.url.get());

            t.start();

            if (!latch.await(this.time, this.unit)) {                     *5*
                throw new RuntimeException("Failed to connect
     to server within timeout: "
                        + this.url.get());
            }

        } finally {
            this.poll.set(false);
            this.lock.unlock();
        }
    }

    @Override
    protected void after() {

        this.lock.lock();                                                 *6*

        try {
            if (null != process) {
                process.destroy();
                process = null;
            }
        } finally {
            this.lock.unlock();
        }
    }

    private boolean connect(final URL url) {

        do {
            try {
                Request request = new Request.Builder().url(url).build();

                if (new OkHttpClient().newCall(request)
                    .execute().isSuccessful()) {                          *7*
                    return true;
                } else {
                    throw new Exception("Unexpected family");
                }
            } catch (Exception ignore) {

                if (poll.get()) {
                    try {
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        return false;
                    }
                }
            }
        } while (poll.get());

        return false;
    }

}
```

+   ***1* 使用构建器模式来设置参数，使得规则的内联使用变得简单。**

+   ***2* 使用默认的 ResolutionStrategy 来查找 Java 可执行文件（可覆盖）**

+   ***3* 启动微服务的胖 JAR 可执行进程**

+   ***4* 轮询指定的端点 URL 以建立成功连接**

+   ***5* 在指定时间段后等待连接或超时**

+   ***6* 使用锁来同步启动和关闭过程**

+   ***7* 你只关心端点的有效连接，而不是响应。**

正如使用 `before` 来初始化资源一样，你使用 `after` 来清理它们。

|  |
| --- |

##### 注意

你实际上并不是在测试端点；你使用它们来确认你的端到端测试准备就绪。请随意编写自己的连接检查例程以满足你的需求。

|  |
| --- |

现在规则已经实现，你所需要做的就是将它们用于测试。

|  |
| --- |

##### 警告

避免使用非静态的 JUnit `@Rule` 注解，因为它会停止和启动每个单独测试的 *所有* 规则进程！

|  |
| --- |

##### 列表 7.13\. code/web/src/test/java/book/web/EndToEndTest.java - @ClassRule

```
@ClassRule
public static RuleChain chain = RuleChain                                  *1*
        .outerRule(new MongodRule("localhost", 27017))                     *2*
        .around(new RedisRule(6379))                                       *3*
        .around(new MicroserviceRule("http://localhost:8899/?videoId=5123
 &gameName=Zelda").withExecutableJar
                (getFile("video/build/libs/video-service-0.1.0.jar"),
                "--server.port=8899")
                .withJavaResolutionStrategy
                    (new DefaultJavaResolutionStrategy()).withTimeout
                    (1, TimeUnit.MINUTES))                                 *4*

        .around(new MicroserviceRule("http://localhost:8181?query=")
                .withExecutableJar(getFile
                ("game/target/gameservice-swarm.jar"), "-Dswarm" +
                ".http.port=8181").withJavaResolutionStrategy
                (new DefaultJavaResolutionStrategy()).withTimeout(1,
                TimeUnit.MINUTES));
```

+   ***1* 使用静态 JUnit RuleChain 确保规则尽可能早地按定义的顺序执行**

+   ***2* 以 MongodRule 开始**

+   ***3* 使用 RedisRule**

+   ***4* RedisRule 在 MicroserviceRule 之后，如果它们不相互依赖，可以指定任何顺序。**

如果你的任何服务相互依赖，你可以在定义启动顺序的地方这样做。

|  |
| --- |

##### 提示

定义超时在所有情况下几乎都不是确定性的。对于端到端测试，你必须出于明显的原因打破这个规则——连接轮询可能永远不会成功。尝试调整参数以尽可能地在你的环境中实现确定性。

|  |
| --- |

#### 7.4.6\. 使用 @OperateOnDeployment 操作部署

你已经多次看到如何定义和使用多个部署。在 列表 7.13 中，`@OperateOnDeployment` 注解指向特定测试应使用的部署。你还在使用 `@ArquillianResource` 注解注入资源 URL。你使用 `@InSequence` 注解来提供测试的顺序，严格来说，这对于正常单元测试来说是一个大忌。对于端到端测试，你已经知道你必须逻辑上协调环境，因此几乎不可能允许测试任意运行。总会有一个操作顺序，但测试仍然应该执行一个工作单元。

##### 列表 7.14\. 定义多个部署

```
private static final AtomicReference<URL>
    commentsservice = new AtomicReference<>();
private static final AtomicReference<URL>
    gameaggregatorservice = new AtomicReference<>();
private static final AtomicReference<URL>
    gamerweb = new AtomicReference<>();

@Test
@InSequence(1)
@OperateOnDeployment("commentsservice")
public void testRunningInCommentsService(@ArquillianResource final URL url)
    throws Exception {
    commentsservice.set(url);
    Assert.assertNotNull(commentsservice.get());
    assertThat(commentsservice.get().toExternalForm(),
        containsString("commentsservice"));
}

@Test
@InSequence(2)
@OperateOnDeployment("gameaggregatorservice")
public void testRunningInGameAggregatorService(@ArquillianResource
    final URL url) throws Exception {
    gameaggregatorservice.set(url);
    Assert.assertNotNull(gameaggregatorservice.get());
    assertThat(gameaggregatorservice.get().toExternalForm(),
        containsString("gameaggregatorservice"));
}

@Test
@InSequence(4)
@OperateOnDeployment("gamerweb")
public void testRunningInGamerWeb(@ArquillianResource final URL url)
    throws Exception {
    gamerweb.set(url);
    Assert.assertNotNull(gamerweb.get());
    assertThat(gamerweb.get().toExternalForm(), containsString("gamerweb"));
}
```

#### 7.4.7\. 介绍 @Drone、页面对象、@Location 和 WebDriver

WebDriver 可以通过 `@Drone` 注解直接注入到你的测试中。更好的方法是创建所有与浏览器相关的测试的 *页面对象*。页面对象不过是虚拟包装器，旨在表示你的 UI 应用程序的单个可查看页面或元素。它们应该只封装与当前页面或元素相关的特定逻辑。

以下列表显示了 `Index` 页面对象的示例（code/web/src/test/java/book/web/page/Index.java）。

##### 列表 7.15\. 一个页面对象示例

```
package book.web.page;

import org.jboss.arquillian.drone.api.annotation.Drone;
import org.jboss.arquillian.graphene.Graphene;
import org.jboss.arquillian.graphene.page.Location;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;

@Location("/")                                  *1*
public class Index {

    @Drone
    private WebDriver browser;                  *2*

    @FindBy(id = "tms-search")                  *3*
    private WebElement search;

    @FindBy(id = "tms-button")
    private WebElement button;

    @FindBy(className = "col-sm-3")
    private List list;                          *4*

    public void navigateTo(String url) {        *5*
        browser.manage().window().maximize();
        browser.get(url);
    }

    public List searchFor(String text) {
        search.sendKeys(text);                  *6*

        Graphene.guardAjax(button).click();     *7*

        return list;                            *8*
    }

    public String getSearchText() {
        return search.getAttribute("value");
    }
}
```

+   ***1* 可选的 @Location 注解定义了页面在服务器上的位置。**

+   ***2* 使用 WebDriver 访问浏览器功能**

+   ***3* 使用 Selenium @FindBy 注解通过其物理 DOM 标识符定位的元素**

+   ***4* 提供一个嵌入式页面片段（稍后详细介绍）**

+   ***5* 以编程方式管理浏览器环境**

+   ***6* 向选定的 HTML 元素（在这个例子中是文本框）发送按键**

+   ***7* 使用 Graphene 等待（阻塞）列表响应**

+   ***8* 返回页面片段以供进一步使用**

这里还使用了一个酷炫的功能：*页面片段*，它们是表示 UI 页面动态元素的对象。下一个列表显示了示例（code/web/src/test/java/book/web/page/List.java）。你可以使用页面片段为测试提供一个逻辑模型。它们可以嵌套，这使得它们对于定义 UI 转换非常有用。使用 Graphene 允许测试在 UI 执行转换时自动阻塞。

##### 列表 7.16\. 一个页面片段示例

```
package book.web.page;

import org.jboss.arquillian.graphene.Graphene;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;

import java.util.ArrayList;
import java.util.Collection;

public class List {

    @FindBy(className = "list-group")
    private WebElement list;

    @FindBy(id = "detail-view")
    private Detail detail;                                           *1*

    public Collection<String> getResults() {                         *2*

        ArrayList<String> results = new ArrayList<>();

        if (null != list) {
            java.util.List<WebElement> elements = list.findElements
                    (By.cssSelector("a > p"));

            for (WebElement element : elements) {
                results.add(element.getText());
            }
        }

        return results;
    }

    public Detail getDetail(int index) {

        java.util.List<WebElement> elements = list.findElements(By
                .cssSelector("a > p"));

        if (!elements.isEmpty()) {
            Graphene.guardAjax(elements.get(index)).click();         *3*
        }

        return detail;                                               *4*
    }
}
```

+   ***1* 提供另一个嵌套页面片段**

+   ***2* 收集结果以用于测试中的验证**

+   ***3* 使用 Graphene 等待（阻塞）详细响应**

+   ***4* 返回详细片段以供进一步使用**

| |
| --- |

##### 注意

这个简单的例子打开了 Selenium 浏览器自动化的广阔、狂野世界。这个主题可以轻易填满另一本书；花些时间访问 [www.seleniumhq.org/docs](http://www.seleniumhq.org/docs) 的文档。

| |
| --- |

#### 7.4.8\. 在测试中使用页面对象

一旦您定义了一个页面对象，您就可以使用`@Page`注解将其注入到您的测试中，如下所示（code/web/src/test/java/book/web/EndToEndTest.java）。然后可以使用它来操作测试浏览器以执行测试。

##### 列表 7.17\. 注入页面对象

```
@Page
@OperateOnDeployment("gamerweb")
private Index page;                                            *1*

@Test
@InSequence(7)
public void testTheUI() throws Exception {

    System.out.println("gameaggregatorservice = " +
        gameaggregatorservice.get().toExternalForm());

    page.navigateTo(gamerweb.get().toExternalForm());          *2*
    List list = page.searchFor("Zelda");

    Assert.assertEquals("", "Zelda", page.getSearchText());

    Assert.assertThat(list.getResults(),
        hasItems("The Legend of Zelda: Breath of the Wild"));

    Detail detail = list.getDetail(0);

    Assert.assertTrue(detail.getImageURL().startsWith("http"));

    if (null != System.getProperty("dev.hack")) {              *3*
        new ServerSocket(9999).accept();
    }
}
```

+   ***1* 页面对象使用@Page 注入，并且它将@OperateOnDeployment(“gamerweb”)。**

+   ***2* 使用页面对象方法**

+   ***3* 简单的开发者技巧来停止测试（更多内容请参阅“开发技巧”侧边栏）**

注意，您在`navigateTo`方法中提供了 URL。这是使用`@Location`注解的程序化替代方案，有时它可能过于僵化。

| |
| --- |

**开发技巧**

使用`ServerSocket`来停止运行是一个简单的开发技巧，您可能会觉得很有用。实际上，通过在 IDE 或 Maven 中“运行”测试类，您此时已经启动了所有微服务。这项技术对于开发 UI 很有用，因为您知道所有服务端点的位置。向测试运行时提供`-Ddev.hack=true`系统属性将确保它无限期地等待 9999 端口的连接。

您可以将 Web 应用程序部署到 IDE 中的可调试容器中，并继续针对运行中的服务进行开发。要停止测试运行时，请执行简单的`curl localhost:9999`调用。

显然，这种方法对每个人来说可能都不适用，微服务通常由个别团队开发和部署，但这可能值得思考。

| |
| --- |

最后可能引起兴趣的是 Graphene 可用的配置选项。各种守卫方法会阻塞一个可配置的周期。默认配置可以通过命令行覆盖

```
-Darq.extension.graphene.waitAjaxInterval=3
```

或者，在 arquillian.xml 中，如下所示。

##### 列表 7.18\. 覆盖默认配置

```
<extension qualifier="graphene">
    <property name="waitAjaxInterval">3</property>
</extension>
```

除了我们在这里提到的，还有许多其他使用石墨烯的方法，但同样，这个主题超出了本书的范围。如果您想了解更多关于可用性的信息，您可以在[`docs.jboss.org/author/display/ARQGRA2/Home`](https://docs.jboss.org/author/display/ARQGRA2/Home)找到项目文档。

#### 7.4.9\. 运行测试

在所有这些之后，让这个测试运行起来可能真的会是一场噩梦，对吧？错！运行测试应该像任何其他单元测试一样直观——毕竟，这就是目标。

| |
| --- |

##### 注意

如果您还没有获取所需的 API 密钥并定义相应的环境条目，如第二章所述，请在运行测试之前这样做。因为这个是端到端测试，这个测试会使用这些密钥对 REST API 进行实际调用；测试还需要互联网连接。

| |
| --- |

这里是命令：

```
cd web
mvn clean install
```

就这些了。如果一切就绪，以下事情会发生：

> **1**.  测试类规则部署并启动了 MongoDB 和 Redis 服务器。
> 
> **2**.  测试类规则启动了独立的微服务。
> 
> **3**.  启动了所有 Arquillian 管理的容器，并部署了相应的应用程序。
> 
> **4**.  Arquillian Drone 启动了一个测试浏览器，并显示了用户界面。
> 
> **5**.  每个测试按顺序运行，以提供对微服务和 Web 应用程序的访问。
> 
> **6**.  环境被干净地终止——所有服务器都已关闭。

图 7.3 展示了示例。

##### 图 7.3\. 测试期间的 UI 自动化

![](img/07fig03_alt.jpg)

| |
| --- |

##### 警告

如果你遇到错误消息“目标存在，我们没有标记为覆盖它”，请执行`maven clean`周期。这个错误通常意味着最后一次测试运行没有正确完成，并且在终止时未能执行清理。

|  |
| --- |

你会注意到这个测试非常慢。在端到端测试中，这通常难以避免，因为需要执行大量的连接操作。唯一的真正解决方案是在除开发机器之外的专用机器上执行端到端测试。我们建议在专用构建模块中创建所有端到端测试。然后，可以在仅在外部机器（如 Jenkins 或 Bamboo）上启用的构建配置文件中激活此模块。有关 Maven 构建配置文件的信息，请参阅[`mng.bz/JDcT`](http://mng.bz/JDcT)。

### 7.5\. 练习

你现在有一个基本的测试模板，其中包含你执行有效端到端测试所需的全部功能，使用多个微服务。我们知道它并不漂亮，但这主要是因为你没有使用纯种环境。你在这里的唯一真正任务是评估为测试提供的信息和环境。

`@InSequence(7)`测试从垂直测试断言开始，检查输入是否正确。然后通过触发搜索并检索结果`List`执行水平操作。显然，这个测试违反了执行单一工作单元的规则，但它这样做是为了演示目的。继续通过将断言提取到更进一步的`@InSequence(x)`测试方法中。

此外，尽管你已经启动了 Mongod 和 Redis 实例，但你实际上并没有用任何测试数据对数据库进行初始化。尝试跳回到第五章并添加一个填充器到测试中。世界是你的牡蛎！

### 摘要

+   端到端测试与其他所有形式的测试一样重要，如果不是更重要的话，并且应该在所有需要任何形式用户交互的应用程序上执行。额外的开销将在以后带来回报：这是一个承诺！

+   总是尽可能使用抽象和重用代码来构建自己的测试套件。

+   在 UI 应用程序的设计早期规划端到端测试，以减少后续的影响。

+   在您的应用程序中为特色标签添加逻辑上明确定义的 UI 属性，如`name`和`id`，将确保它准备好接受通过 Selenium 驱动程序推广的自动化操作。尽量定义并坚持您自己的命名约定。
