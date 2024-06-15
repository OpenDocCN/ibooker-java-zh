# 第四章：配置

在本章中，您将学习有关设置配置参数的以下内容：

+   如何配置 Quarkus 服务

+   如何在服务中注入配置参数

+   如何根据环境应用值

+   如何正确配置日志系统

+   如何为配置系统创建自定义项

# 4.1 使用自定义属性配置应用程序

## 问题

您希望使用自定义属性配置 Quarkus 应用程序。

## 解决方案

Quarkus 利用了多个 Eclipse MicroProfile 规范之一。其中之一是配置规范；但是，为了简化配置，Quarkus 仅使用一个文件来进行所有配置，即*application.properties*，该文件必须放置在类路径的根目录下。

此文件可用于配置 Quarkus 属性，如日志或默认路径，Quarkus 扩展如数据源或 Kafka，或者您为应用程序定义的自定义属性。您将在本书中看到所有这些内容，但在本节中，您将看到后者。

打开*src/main/resources/application.properties*文件，并添加以下属性：

```java
greeting.message=Hello World
```

您可以使用`org.eclipse.microprofile.config.inject.ConfigProperty`注解在字段中注入*application.properties*中定义的属性值。

打开`org.acme.quickstart.GreetingResource.java`并注入`greeting.message`属性值：

```java
@ConfigProperty(name = "greeting.message") ![1](img/1.png)
String message; ![2](img/2.png)

@GET
@Produces(MediaType.TEXT_PLAIN)
public String hello() {
    return message; ![3](img/3.png)
}
```

![1](img/#co_configuration_CO1-1)

注入`greeting.message`属性的值

![2](img/#co_configuration_CO1-2)

将字段置于包保护范围内

![3](img/#co_configuration_CO1-3)

返回配置的数值

###### 提示

出于性能原因，在使用 GraalVM 和反射时，我们建议您在运行时注入的字段上使用*protected-package*范围。您可以在[Quarkus CDI 参考指南](https://oreil.ly/8e1Sd)中了解更多信息。

在新的终端窗口中，向`/hello`发出请求，查看输出消息是否为*application.properties*中配置的值：

```java
curl http://localhost:8080/hello

Hello World
```

如果要使配置字段非强制且提供默认值，可以使用`@ConfigProperty`注解的`defaultValue`属性。

打开`org.acme.quickstart.GreetingResource.java`文件并注入`greeting.upper-case`属性值：

```java
@ConfigProperty(name = "greeting.upper-case",
                defaultValue = "true") ![1](img/1.png)
boolean upperCase;
@GET
@Path("/optional")
@Produces(MediaType.TEXT_PLAIN)
public String helloOptional() {
    return upperCase ? message.toUpperCase() : message;
}
```

![1](img/#co_configuration_CO2-1)

将`greeting.upper-case`属性的默认值设置为 true

然后在终端窗口，向`/hello/optional`发出请求，查看输出消息是否为大写：

```java
curl http://localhost:8080/hello/optional

HELLO WORLD
```

支持多值属性——您只需将字段类型定义为`Arrays`、`java.util.List`或`java.util.Set`之一，具体取决于您的需求/偏好。属性值的分隔符是逗号（`,`），转义字符是反斜杠（`\`）。

打开*src/main/resources/application.properties*文件，并添加具有三个值的以下属性：

```java
greeting.suffix=!!, How are you???
```

打开 *org.acme.quickstart.GreetingResource.java* 并注入 `greeting.suffix` 属性值：

```java
@ConfigProperty(name = "greeting.suffix")
List<String> suffixes;
@GET
@Path("/list")
@Produces(MediaType.TEXT_PLAIN)
public String helloList() {
    return message + suffixes.get(1);
}
```

并在终端窗口中发出 `/hello/list` 请求，以查看输出消息是否包含第二后缀：

```java
curl http://localhost:8080/hello/list

Hello World How are you?
```

YAML 格式也支持配置应用程序。在这种情况下，文件名为 *application.yaml* 或 *application.yml*。

要开始使用 YAML 配置文件，您需要添加 `config-yaml` 扩展：

```java
./mvnw quarkus:add-extension -Dextensions="config-yaml"
```

给定以下使用 `properties` 格式的配置文件：

```java
greeting.message=Hello World

%staging.quarkus.http.port=8182

quarkus.http.cors=true
quarkus.http.cors.methods=GET,PUT,POST
```

YAML 格式中的等效格式如下：

```java
greeting:
  message: Hello World ![1](img/1.png)
"%staging": ![2](img/2.png)
  quarkus:
    http:
      port: 8182
quarkus:
  http:
    cors:
      ~: true ![3](img/3.png)
      methods: GET,PUT,POST
```

![1](img/#co_configuration_CO3-1)

简单属性被设置为一个结构

![2](img/#co_configuration_CO3-2)

支持包含在引号中的配置文件

![3](img/#co_configuration_CO3-3)

当存在子键时，使用 `~` 来引用无前缀部分

## 讨论

Eclipse MicroProfile 配置带有以下内置转换器，以将配置值映射为 Java 对象：

+   `boolean` 和 `java.lang.Boolean`；true 的值为 `true`、`1`、`YES`、`Y` 和 `ON`，其他任何值都被视为 `false`

+   `byte` 和 `java.lang.Byte`

+   `short` 和 `java.lang.Short`

+   `int` 和 `java.lang.Integer`

+   `long` 和 `java.lang.Long`

+   `float` 和 `java.lang.Float`

+   `double` 和 `java.lang.Double`

+   `char` 和 `java.lang.Character`

+   基于调用 `Class.forName` 的结果的 `java.lang.Class`

如果不存在内置转换器或自定义转换器，则在目标对象中检查以下方法。如果存在内置转换器或自定义转换器，则使用找到的方法实例化转换器对象，并将字符串参数传递进行转换：

+   目标类型具有 `public static T of(String)` 方法

+   目标类型具有 `public static T valueOf(String)` 方法

+   目标类型具有带有 `String` 参数的公共构造函数

+   目标类型具有 `public static T parse(CharSequence)` 方法

# 4.2 以编程方式访问配置属性

## 问题

您希望以编程方式访问配置属性，而不是使用 `org.eclipse.microprofile.config.inject.ConfigProperty` 注解进行注入。

## 解决方案

在您希望以编程方式访问属性的对象中注入 `org.eclipse.microprofile.config.Config` 类。

Eclipse MicroProfile 配置规范允许您注入 `org.eclipse.microprofile.config.Config` 以便以编程方式获取属性，而不是直接使用 `ConfigProperty` 进行注入。

打开 `org.acme.quickstart.GreetingResource.java` 并注入 `Config` 类：

```java
@Inject ![1](img/1.png)
Config config;
@GET
@Path("/config")
@Produces(MediaType.TEXT_PLAIN)
public String helloConfig() {
    config.getPropertyNames().forEach( p -> System.out.println(p)); ![2](img/2.png)

    return config.getValue("greeting.message", String.class); ![3](img/3.png)
}
```

![1](img/#co_configuration_CO4-1)

使用 `Inject` CDI 注解来注入实例

![2](img/#co_configuration_CO4-2)

现在可以访问属性列表

![3](img/#co_configuration_CO4-3)

需要将属性转换为最终类型

您可以通过调用 `ConfigProvider.getConfig()` 方法访问 `Config` 类，而无需使用 CDI。

# 4.3 在外部覆盖配置值

## 问题

你希望在运行时覆盖任何配置值。

## 解决方案

你可以通过将其设置为系统属性或环境变量来在运行时覆盖任何属性。

Quarkus 允许你通过将配置设置为系统属性（`-Dproperty.name=value`）和/或环境变量（`export PROPERTY_NAME=value`）来覆盖任何配置属性。系统属性比环境变量具有更高的优先级。

外部化这些属性的示例可以是数据库 URL、用户名或密码，因为它们仅在目标环境中知道。但是你需要知道这是一种权衡，因为可用的运行时属性越多，Quarkus 能够执行的构建时间预处理就越少。

让我们对配方 4.1 中使用的应用程序进行打包，并通过设置系统属性覆盖`greeting.message`属性：

```java
./mvnw clean package -DskipTests

java -Dgreeting.message=Aloha -jar target/getting-started-1.0-SNAPSHOT-runner.jar
```

在新的终端窗口中运行以下命令验证属性已从`Hello World`覆盖为`Aloha`：

```java
curl localhost:8080/hello

Aloha
```

对于环境变量，针对给定属性名称支持三种命名约定。这是因为一些操作系统只允许字母字符和下划线(`_`），而不允许其他字符，如点(`.`）。为了支持所有可能的情况，使用以下规则：

1.  完全匹配（`greeting.message`）。

1.  将非字母数字字符替换为下划线（`greeting_message`）。

1.  将非字母数字字符替换为下划线，并将其余字符转换为大写（`GREETING_MESSAGE`）。

这是*application.properties*文件：

```java
greeting.message=Hello World
```

您可以使用以下任何环境变量名称来覆盖其值，因为它们都是等效的：

```java
export greeting.message=Aloha
export greeting_message=Aloha
export GREETING_MESSAGE=Aloha
```

还有一个特殊的地方，你可以将*application.properties*文件放在应用程序之外，放在一个名为*config*的目录内，该目录位于应用程序运行的地方。该文件中定义的任何运行时属性都将覆盖默认配置。

###### 重要提示

*config/application.properties*也适用于开发模式，但是您需要将其添加到构建工具的输出目录才能使其正常工作（在 Maven 的情况下，是*target*目录；在 Gradle 的情况下，是*build*），因此您需要注意在运行`clean`任务时重新创建它的需要。

除了环境变量和*application.properties*文件之外，您还可以将*.env*文件放在当前工作目录中，以覆盖配置值，遵循环境变量格式（`GREETING_MESSAGE=Aloha`）。

# 4.4 配置与配置文件

## 问题

你希望根据运行 Quarkus 的环境覆盖配置值。

## 解决方案

Quarkus 支持配置文件的概念。这允许您在同一文件中为同一属性具有多个配置值，并使不同的值适合您正在运行服务的环境。

配置文件的语法是`%{profile}.config.key=value`。

## 讨论

Quarkus 带有三个内置配置文件。

开发

激活开发模式时（即，`quarkus:dev`）。

测试

在运行测试时激活。

生产

当不在开发或测试模式下运行时的默认配置文件；您不需要在 *application.properties* 中设置它，因为它会隐式设置。

打开 *src/main/resources/application.properties* 文件，并设置在开发模式下以端口 8181 启动 Quarkus：

```java
%dev.quarkus.http.port=8181
```

在此更改后，启动服务以再次检查监听端口是否为 8181 而不是默认端口（8080）：

```java
./mvnw compile quarkus:dev

INFO  [io.qua.dep.QuarkusAugmentor] (main) Beginning quarkus augmentation
INFO  [io.qua.dep.QuarkusAugmentor] (main) Quarkus augmentation completed
 in 671ms
INFO  [io.quarkus] (main) Quarkus 1.4.1 started in 1.385s. Listening on:
 http://0.0.0.0:8181
INFO  [io.quarkus] (main) Profile dev activated. Live Coding activated.
INFO  [io.quarkus] (main) Installed features:
 [cdi, hibernate-validator, resteasy]
```

注意，现在监听地址是[*http://0.0.0.0:8181*](http://0.0.0.0:8181)，而不是默认地址。

最后，回滚到 8080 端口，在 *application.properties* 中删除 `%dev.quarkus.http.port=8181` 行，以与书中其他部分使用的端口对齐。

# 4.5 更改日志配置

## 问题

您希望更改默认的日志配置。

## 解决方案

Quarkus 使用统一的配置模型，其中所有配置属性都放在同一个文件中。对于 Quarkus 来说，这个文件就是 *application.properties*，您可以在其中配置许多日志方面。

例如，如果要更改日志级别，只需将 `quarkus.log.level` 设置为最低日志级别。

打开 *src/main/resources/application.properties* 并添加以下内容：

```java
quarkus.log.level=DEBUG
```

现在启动应用程序以查看控制台中打印了大量新的日志消息：

```java
./mvnw compile quarkus:dev

...
[INFO] --- quarkus-maven-plugin:0.22.0:dev (default-cli) @ getting-started ---
Listening for transport dt_socket at address: 5005
DEBUG [org.jbo.logging] (main) Logging Provider: \
 org.jboss.logging.JBossLogManagerProvider
INFO  [io.qua.dep.QuarkusAugmentor] (main) Beginning quarkus augmentation
DEBUG [io.qua.run.con.ConverterSupport] (main) Populate SmallRye config builder
 with converter for class java.net.InetSocketAddress of priority 200
DEBUG [io.qua.run.con.ConverterSupport] (main) Populate SmallRye config builder
 with converter for class org.wildfly.common.net.CidrAddress of priority 200
```

###### 注意

我们必须跨多行进行格式化以适应书籍；我们使用反斜杠来指示这一点。

您还可以通过使用 `quarkus.log.file.enable` 属性将日志存储到文件中。输出默认写入名为 *quarkus.log* 的文件中：

```java
quarkus.log.file.enable=true
```

###### 注意

在开发并从源目录工作时，您的日志文件将位于 *target* 目录中。

# 4.6 添加应用程序日志

## 问题

您希望向应用程序添加日志行。

## 解决方案

大多数时候，您的应用程序需要编写自己的日志消息，而不仅仅依赖于 Quarkus 提供的默认日志。应用程序可以使用任何支持的日志 API 进行日志记录，并且这些日志将被合并。

Quarkus 支持这些日志记录库：

+   JDK java.util.logging

+   JBoss 日志记录

+   SLF4J

+   Apache Commons Logging

让我们看看如何使用 `JBoss Logging` 记录内容。打开 `org.acme.quickstart.GreetingResource.java` 并在调用特定端点时记录消息：

```java
private static org.jboss.logging.Logger logger =
                org.jboss.logging.Logger.getLogger(GreetingResource.class); ![1](img/1.png)

@GET
@Path("/log") ![2](img/2.png)
@Produces(MediaType.TEXT_PLAIN)
public String helloLog() {
    logger.info("I said Hello"); ![3](img/3.png)
    return "hello";
}
```

![1](img/#co_configuration_CO5-1)

创建日志记录器实例

![2](img/#co_configuration_CO5-2)

终端子路径是 */log*

![3](img/#co_configuration_CO5-3)

信息级别的日志

现在启动应用程序：

```java
./mvnw compile quarkus:dev
```

在新的终端窗口中，向 `/hello/log` 发出请求：

```java
curl http://localhost:8080/hello/log
```

如果检查您启动 Quarkus 的终端，您会看到下一个日志行：

```java
INFO  [org.acm.qui.GreetingResource] (executor-thread-1) I said Hello
```

## 讨论

日志是按类别进行的。适用于类别的配置也适用于该类别的所有子类别，除非有更具体的子类别配置。

分类由类的位置表示（即定义它们的包或子包）。例如，如果要将 Undertow 安全日志设置为跟踪级别，需要在*application.properties*中设置`quarkus.log.category."io.undertow.request.security".level=TRACE`属性。

跟随前面的示例，让我们限制来自`org.acme.quickstart`包（及其子类）的日志行，以确保最低日志级别为`WARNING`：

```java
quarkus.log.category."org.acme.quickstart".level=WARNING ![1](img/1.png)
```

![1](img/#co_configuration_CO6-1)

引号是设置类别的必需部分

如果您重复请求[*http://localhost:8080/hello/log*](http://localhost:8080/hello/log)，则日志行不再被记录。

# 4.7 高级日志

## 问题

您希望将所有服务的日志集中记录。

## 解决方案

在处理微服务架构和 Kubernetes 时，日志记录是需要考虑的重要事项，因为每个服务都会单独记录日志；但作为开发人员或操作员，您可能希望将所有日志集中到一个位置，以便作为整体使用。

Quarkus 日志还支持 JSON 和 GELF 输出。

这些日志可以以 JSON 格式而不是纯文本形式编写，以供通过注册`logging-json`扩展进行机器处理：

```java
./mvnw quarkus:add-extension -Dextensions="logging-json"
```

使用 GELF 扩展以生成 GELF 格式的日志，并使用 TCP 或 UDP 发送它们。

*Graylog 扩展日志格式*（GELF）现在被三个最常用的集中式日志系统所支持：

+   Graylog（MongoDB，Elasticsearch，Graylog）

+   ELK（Elasticsearch，Logstash，Kibana）

+   EFK（Elasticsearch，Fluentd，Kibana）

要开始以 GELF 格式记录日志，您只需添加`logging-gelf`扩展即可：

```java
./mvnw quarkus:add-extension -Dextensions="logging-gelf"
```

日志代码不会更改，因此仍使用相同的接口：

```java
private static org.jboss.logging.Logger logger =
                org.jboss.logging.Logger.getLogger(GreetingResource.class); ![1](img/1.png)

@GET
@Path("/log") ![2](img/2.png)
@Produces(MediaType.TEXT_PLAIN)
public String helloLog() {
    logger.info("I said Hello"); ![3](img/3.png)
    return "hello";
}
```

![1](img/#co_configuration_CO7-1)

创建记录器实例

![2](img/#co_configuration_CO7-2)

终端子路径为*/log*

![3](img/#co_configuration_CO7-3)

`info`级别的日志

必须在*application.properties*中配置 GELF 处理程序：

```java
quarkus.log.handler.gelf.enabled=true ![1](img/1.png)
quarkus.log.handler.gelf.host=localhost ![2](img/2.png)
quarkus.log.handler.gelf.port=12201 ![3](img/3.png)
```

![1](img/#co_configuration_CO8-1)

启用扩展

![2](img/#co_configuration_CO8-2)

设置发送日志消息的主机

![3](img/#co_configuration_CO8-3)

设置端点端口

###### 重要

如果使用 Logstash（ELK），则需要启用能够理解 GELF 格式的输入插件：

```java
input {
  gelf {
    port => 12201
  }
}
output {
  stdout {}
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
  }
}
```

###### 重要

如果使用 Fluentd（EFK），则需要启用能够理解 GELF 格式的输入插件：

```java
<source>
  type gelf
  tag example.gelf
  bind 0.0.0.0
  port 12201
</source>

<match example.gelf>
  @type elasticsearch
  host elasticsearch
  port 9200
  logstash_format true
</match>
```

## 讨论

Quarkus 日志还通过默认支持 syslog 格式，无需添加任何扩展。在 Fluentd 中，syslog 格式可用作 Quarkus 中 GELF 格式的替代：

```java
quarkus.log.syslog.enable=true
quarkus.log.syslog.endpoint=localhost:5140
quarkus.log.syslog.protocol=udp
quarkus.log.syslog.app-name=quarkus
quarkus.log.syslog.hostname=quarkus-test
```

###### 重要

您需要启用能够理解 Fluentd 中 syslog 格式的输入插件：

```java
<source>
  @type syslog
  port 5140
  bind 0.0.0.0
  message_format rfc5424
  tag system
</source>

<match **>
  @type elasticsearch
  host elasticsearch
  port 9200
  logstash_format true
</match>
```

如果使用 Kubernetes，最简单的日志记录方式是记录到控制台，并在集群中安装一个中央日志管理器以收集所有日志行。

## 参见

要了解更多高级日志主题，请访问以下网站：

+   [Logstash/Gelf 记录器](https://oreil.ly/Mj9Ha)

# 4.8 使用自定义配置文件配置

## 问题

您希望为您创建的自定义配置文件设置不同的配置值。

## 解决方案

到目前为止，您已经看到 Quarkus 具有内置的配置文件，因此您可以为同一属性设置不同的配置值，并使它们适应环境。但是，使用 Quarkus，您也可以设置自己的配置文件。

您唯一需要做的就是指定要启用的配置文件，要么使用`quarkus.profile`系统属性，要么使用`QUARKUS_PROFILE`环境变量。如果两者都设置了，系统属性优先于环境变量。

然后，您需要做的唯一一件事是创建带有配置文件名的属性，并将当前配置文件设置为该名称。让我们创建一个名为*staging*的新配置文件，该配置文件将覆盖 Quarkus 的侦听端口。

打开*src/main/resources/application.properties*文件，并设置在启用`staging`配置文件时将 Quarkus 启动到端口 8182：

```java
%staging.quarkus.http.port=8182
```

然后使用`staging`配置文件启动应用程序：

```java
./mvnw -Dquarkus.profile=staging compile quarkus:dev

INFO  [io.qua.dep.QuarkusAugmentor] (main) Beginning quarkus augmentation
INFO  [io.qua.dep.QuarkusAugmentor] (main) Quarkus augmentation completed
 in 640ms
INFO  [io.quarkus] (main) Quarkus 0.23.2 started in 1.300s. Listening on:
 http://0.0.0.0:8182
INFO  [io.quarkus] (main) Profile staging activated. Live Coding activated.
INFO  [io.quarkus] (main) Installed features: [cdi, hibernate-validator,
 resteasy]
```

在这种情况下，使用系统属性方法，但您也可以使用`QUARKUS_PROFILE`环境变量设置它。

## 讨论

如果要在测试中设置运行配置文件，则只需要在构建脚本中将`quarkus.test.profile`系统属性设置为给定配置文件即可，例如，在 Maven 中：

```java
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-surefire-plugin</artifactId>
<version>${surefire-plugin.version}</version>
<configuration>
    <systemPropertyVariables>
        <quarkus.test.profile>foo</quarkus.test.profile>
        <buildDirectory>${project.build.directory}</buildDirectory>
    </systemPropertyVariables>
</configuration>
```

或者，在 Gradle 中：

```java
test {
    useJUnitPlatform()
    systemProperty "quarkus.test.profile", "foo"
}
```

另外，您可以更改默认的生产配置文件。Quarkus 中内置的配置文件是`prod`，因此当您在不指定任何配置文件的情况下运行应用程序时，这是默认配置文件，其中包含的值被使用。但是，您可以在构建时更改该配置文件，以便在应用程序运行时不指定任何配置文件时，您的配置文件是默认配置文件。

你需要做的唯一一件事是使用`quarkus.profile`系统属性构建应用程序，并将要设置为默认值的配置文件命名为配置文件值：

```java
./mvnw package -Pnative -Dquarkus.profile=prod-kubernetes` ./target/getting-started-1.0-runner ![1](img/1.png)
```

![1](img/#co_configuration_CO9-1)

默认情况下，该命令将使用`prod-kubernetes`配置文件启用

# 4.9 创建自定义源

## 问题

您希望从除*application.properties*文件之外的任何其他源加载配置参数。

## 解决方案

Quarkus 使用 Eclipse MicroProfile Configuration 规范来实现有关配置的所有逻辑。该规范提供了`org.eclipse.microprofile.config.spi.ConfigSource` [Java SPI](https://oreil.ly/o0A51) 接口，用于实现一种自定义加载配置属性的方式，而不是使用 Quarkus 提供的默认方式。

例如，您可以从数据库、XML 文件或 REST API 加载配置属性。

让我们创建一个简单的*内存配置源*，该源在实例化时从`Map`中获取配置属性。创建一个名为`org.acme.quickstart.InMemoryConfigSource.java`的新类：

```java
package org.acme.quickstart;

import java.util.HashMap;
import java.util.Map;

import org.eclipse.microprofile.config.spi.ConfigSource;

public class InMemoryConfigSource implements ConfigSource {

    private Map<String, String> prop = new HashMap<>();

    public InMemoryConfigSource() { ![1](img/1.png)
        prop.put("greeting.color", "red");
    }

    @Override
    public int getOrdinal() { ![2](img/2.png)
        return 500;
    }

    @Override
    public Map<String, String> getProperties() { ![3](img/3.png)
        return prop;
    }

    @Override
    public String getValue(String propertyName) { ![4](img/4.png)
        return prop.get(propertyName);
    }

    @Override
    public String getName() { ![5](img/5.png)
        return "MemoryConfigSource";
    }

}
```

![1](img/#co_configuration_CO10-1)

用属性填充映射

![2](img/#co_configuration_CO10-2)

用于确定值的重要性；最高的序数优先于优先级较低的序数

![3](img/#co_configuration_CO10-3)

获取所有属性作为 `Map`；在这种情况下，它是直接的

![4](img/#co_configuration_CO10-4)

获取单个属性的值

![5](img/#co_configuration_CO10-5)

返回此配置源的名称

然后，您需要将其注册为 Java SPI。在 *src/main/resources/META-INF* 下创建 *services* 文件夹。接下来，在 *services* 文件夹内创建一个名为 *org.eclipse.microprofile.config.spi.ConfigSource* 的文件，其内容如下：

```java
org.acme.quickstart.InMemoryConfigSource
```

最后，您可以修改 `org.acme.quickstart.GreetingResource.java` 类来注入这个属性：

```java
@ConfigProperty(name = "greeting.color") ![1](img/1.png)
String color;

@GET
@Path("/color")
@Produces(MediaType.TEXT_PLAIN)
public String color() {
    return color;
}
```

![1](img/#co_configuration_CO11-1)

注入在 `InMemoryConfigSource` 中定义的属性值

然后在终端窗口中对 `/hello/color` 发出请求，以查看输出消息是否为自定义源中配置的值：

```java
curl http://localhost:8080/hello/color

red
```

## 讨论

每个 `ConfigSource` 都有一个指定的序数，用于在同一应用程序中对多个配置源定义的情况下设置值的重要性。如果有多个 `ConfigSource`，则使用较高的序数 `ConfigSource` 而不是具有较低值的 `ConfigSource`。使用以下列表中的默认值作为参考，系统属性将优先于一切，如果没有找到其他 `ConfigSources`，则使用 *src/main/resources* 目录中的 *application.properties* 文件：

+   将系统属性设为 400

+   将环境变量设为 300

+   *config* 目录下的 *application.properties* 至 260

+   项目中的 *application.properties* 至 250

# 4.10 创建自定义转换器

## 问题

您想要实现一个自定义转换器。

## 解决方案

您可以通过实现 `org.eclipse.microprofile.config.spi.Converter` Java SPI 将属性从 `String` 转换为任何类型的对象。

Quarkus 使用 Eclipse MicroProfile Configuration 规范来实现所有关于配置的逻辑。该规范提供了 `org.eclipse.microprofile.config.spi.Converter` [Java SPI](https://oreil.ly/kcqQw) 接口，用于将配置值转换为自定义类型。

例如，您可以将百分比值（即，15%）转换为 `Percentage` 类型，将百分比包装为 `double` 类型。

创建一个新的 POJO 类 `org.acme.quickstart.Percentage.java`：

```java
package org.acme.quickstart;

public class Percentage {

    private double percentage;

    public Percentage(double percentage) {
        this.percentage = percentage;
    }

    public double getPercentage() {
        return percentage;
    }

}
```

然后创建一个名为 `org.acme.quickstart.PercentageConverter.java` 的类，将 `String` 表示转换为 `Percentage`：

```java
package org.acme.quickstart;

import javax.annotation.Priority;

import org.eclipse.microprofile.config.spi.Converter;

@Priority(300) ![1](img/1.png)
public class PercentageConverter implements Converter<Percentage> { ![2](img/2.png)

    @Override
    public Percentage convert(String value) {

        String numeric = value.substring(0, value.length() - 1);
        return new Percentage (Double.parseDouble(numeric) / 100);

    }

}
```

![1](img/#co_configuration_CO12-1)

设置优先级；在这种特定情况下可能是可选的

![2](img/#co_configuration_CO12-2)

泛型类型，将其类型设置为要转换的类型

然后，您需要将其注册为 Java SPI。在 *src/main/resources/META-INF/services* 下创建 *services* 文件夹。接下来，在 *services* 文件夹内创建一个名为 *org.eclipse.microprofile.config.spi.Converter* 的文件，其内容如下：

```java
org.acme.quickstart.PercentageConverter
```

然后，您可以修改 `org.acme.quickstart.GreetingResource.java` 类以注入此属性：

```java
@ConfigProperty(name = "greeting.vat")
Percentage vat;

@GET
@Path("/vat")
@Produces(MediaType.TEXT_PLAIN)
public String vat() {
    return Double.toString(vat.getPercentage());
}
```

最后，您需要在 *src/main/resources* 目录中的 *application.properties* 文件中添加一个新属性：

```java
greeting.vat = 21%
```

然后，在终端窗口中，发出请求 `/hello/vat`，以查看输出消息是否将增值税转换为双倍：

```java
curl http://localhost:8080/hello/vat

0.21
```

## 讨论

默认情况下，如果在转换器上找不到 `@Priority` 注释，则将其注册为优先级为 100。Quarkus 转换器注册为优先级 200，因此如果要替换 Quarkus 转换器，应使用更高的值；如果不需要替换 Quarkus 转换器，则默认转换器完全合适。

Recipe 4.1 中展示了一些 Quarkus 核心转换器的列表。

# 4.11 分组配置值

## 问题

您希望避免一遍又一遍地设置配置属性的公共前缀。

## 解决方案

您可以使用 `@⁠i⁠o​.⁠q⁠u⁠a⁠r⁠k⁠u⁠s⁠.⁠a⁠r⁠c⁠.⁠c⁠o⁠n⁠f⁠i⁠g⁠.⁠C⁠o⁠n⁠f⁠i⁠g⁠P⁠r⁠o⁠p⁠e⁠r⁠t⁠i⁠e⁠s` 注释来分组共同的属性（具有相同的前缀）。

当您在应用程序中创建临时配置属性时，通常这些属性将具有相同的前缀（即 `greetings`）。要注入所有这些属性，您可以使用 `@ConfigProperty` 注释（如 Recipe 4.1 中所示），或者您可以使用 `io.quarkus.arc.config.ConfigProperties` 注释将属性组合在一起。

使用 *application.properties* 文件：

```java
greeting.message=Hello World
greeting.suffix=!!, How are you???
```

让我们实现一个类，使用 `io.quarkus.arc.config.ConfigProperties` 注释将配置属性映射到 Java 对象中。创建一个新类 `org.acme.quickstart.GreetingConfiguration.java`：

```java
package org.acme.quickstart;

import java.util.List;
import java.util.Optional;

import javax.validation.constraints.Max;
import javax.validation.constraints.Min;

import io.quarkus.arc.config.ConfigProperties;

@ConfigProperties(prefix = "greeting") ![1](img/1.png)
public class GreetingConfiguration {

    public String message; ![2](img/2.png)
    public String suffix = "!"; ![3](img/3.png)
}
```

![1](img/#co_configuration_CO13-1)

将其设置为具有共同前缀的配置 POJO

![2](img/#co_configuration_CO13-2)

映射 `greeting.message` 属性

![3](img/#co_configuration_CO13-3)

如果未设置该属性，则 `greeting.suffix` 的默认值

在上述代码中需要注意的一点是，`prefix` 属性并非强制性。如果未设置，那么将由类名（去除 `Configuration` 后缀部分）确定要使用的前缀。在这种情况下，`prefix` 属性可以自动解析为 `greeting`。

然后，您可以注入此配置 POJO 开始使用配置值。

您可以修改 `org.acme.quickstart.GreetingResource.java` 类以注入此类：

```java
@Inject ![1](img/1.png)
GreetingConfiguration greetingConfiguration;

@GET
@Path("/configurations")
@Produces(MediaType.TEXT_PLAIN)
public String helloConfigurations() {
    return greetingConfiguration.message + greetingConfiguration.suffix;
}
```

![1](img/#co_configuration_CO14-1)

配置通过 CDI `@Inject` 注释注入

然后，在终端窗口中，发出请求 `/hello/configurations`，以查看配置值在 Java 内部被填充，例如：

```java
curl http://localhost:8080/hello/configurations

Hello World!!, How are you???
```

正如您现在所看到的，您无需通过使用 `@ConfigProperty` 注释每个字段，只需利用类定义即可获取属性名称或默认值。

## 讨论

此外，Quarkus 支持嵌套对象配置，因此您也可以使用内部类映射子类别。

假设我们在*application.properties*中添加了一个名为`greeting.output.recipients`的新属性：

```java
greeting.output.recipients=Ada,Alexandra
```

您可以使用内部类将其映射到配置对象中。修改`org.acme.quickstart.GreetingConfiguration.java`类。然后添加一个代表子类别`output`的新内部类，并将其注册为字段：

```java
public OutputConfiguration output; ![1](img/1.png)

public static class OutputConfiguration {
    public List<String> recipients;
}
```

![1](img/#co_configuration_CO15-1)

子类别的名称是字段名称（`output`）

然后，您可以访问`greetingConfiguration.output.recipients`字段以获取该值。您还可以使用 Bean Validation 注解对字段进行注释，以验证所有配置值在启动时是否有效。如果它们无效，应用程序将无法启动，并将在日志中指示验证错误。

# 4.12 验证配置数值

## 问题

您希望验证配置数值是否正确。

## 解决方案

使用 Bean Validation 规范验证通过`@ConfigProperty`注解注入的属性值是否有效。

Bean Validation 规范允许您使用注解在对象上设置约束。Quarkus 将 Eclipse MicroProfile 配置规范与 Bean Validation 规范集成在一起，因此您可以一起使用它们验证配置值是否符合某些标准。此验证在启动时执行，如果有任何违规，控制台将显示错误消息，并且启动过程将中止。

您首先需要注册*Quarkus Bean Validation*依赖项。您可以通过手动编辑*pom.xml*或从项目根目录运行下一个 Maven 命令来执行此操作：

```java
./mvnw quarkus:add-extension -Dextensions="quarkus-hibernate-validator"
```

在此之后，您需要创建一个配置对象，这是您在前一篇章节学到的。在下一个示例中，设置对`greeting.repeat`配置属性的约束，以便不能设置超出 1 到 3 范围之外的重复次数。

要验证整数范围，使用以下 Bean Validation 注解：`j⁠a⁠v⁠a⁠x​.⁠v⁠a⁠l⁠i⁠d⁠a⁠t⁠i⁠o⁠n⁠.⁠c⁠o⁠n⁠s⁠t⁠r⁠a⁠i⁠n⁠t⁠s⁠.⁠M⁠a⁠x` 和 `javax.validation.constraints.Min`。打开`org.acme.quickstart.GreetingConfiguration.java`并添加 Bean Validation 注解：

```java
@Min(1) ![1](img/1.png)
@Max(3) ![2](img/2.png)
public Integer repeat;
```

![1](img/#co_configuration_CO16-1)

最小接受值

![2](img/#co_configuration_CO16-2)

最大接受值

打开*src/main/resources/application.properties*文件，并将`greeting.repeat`配置属性设置为 7：

```java
greeting.repeat=7
```

启动应用程序，您将看到一个错误消息，通知某个配置值违反了定义的约束之一：

```java
./mvnw compile quarkus:dev
```

## 讨论

在这个例子中，你已经看到了 Bean Validation 规范的简要介绍，以及一些可以用来验证字段的注解。然而，Hibernate Validation 和使用的 Bean Validation 实现支持更多的约束，比如`@Digits`、`@Email`、`@NotNull`和`@NotBlank`。
