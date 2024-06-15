# 第五章。配置和检查您的 Spring Boot 应用程序

任何应用程序都可能出现许多问题，其中一些问题甚至可能有简单的解决方案。然而，除了偶尔的猜测，必须在真正解决问题之前确定问题的根本原因。

调试 Java 或 Kotlin 应用程序——或者任何其他应用程序，这是每个开发人员在职业生涯早期都应该学会并在其后不断完善和扩展的基本技能。我并不认为这在所有情况下都是适用的，所以如果你尚未熟悉所选语言和工具的调试能力，请尽快探索你手头的选择。这确实在你开发的每个项目中都非常重要，并可以节省大量时间。

也就是说，调试代码只是确定、识别和隔离应用程序中显现行为的一种级别。随着应用程序变得更加动态和分布式，开发人员通常需要执行以下操作：

+   动态配置和重新配置应用程序

+   确定/确认当前设置及其来源

+   检查和监控应用程序环境和健康指标

+   临时调整活动应用程序的日志级别以识别根本原因

本章演示如何利用 Spring Boot 的内置配置能力、其自动配置报告和 Spring Boot 执行器来灵活、动态地创建、识别和修改应用程序环境设置。

# 代码检出检查

请查看代码仓库中的 *chapter5begin* 分支以开始。

# 应用程序配置

没有应用程序是孤立的。

当我说这句话时，大多数时候是为了指出这个真理：几乎在每种情况下，一个应用程序在没有与其他应用程序/服务的交互时不能提供其全部效用。但还有另一层意思同样正确：没有应用程序能够在没有以某种形式访问其环境的情况下如此有用。一个静态、不可配置的应用程序是僵化的、不灵活的和受限的。

Spring Boot 应用程序为开发人员提供了多种强大的机制，可以在应用程序运行时动态配置和重新配置其应用程序。这些机制利用了 Spring `Environment` 来管理来自所有来源的配置属性，包括以下内容：

+   当开发工具（devtools）处于活动状态时，Spring Boot 开发者工具（devtools）全局设置属性位于 *$HOME/.config/spring-boot* 目录中。

+   测试中的 `@TestPropertySource` 注解。

+   测试中的 `properties` 属性，可在 `@SpringBootTest` 和各种测试注解中用于测试应用程序片段。

+   命令行参数。

+   来自 `SPRING_APPLICATION_JSON`（嵌入在环境变量或系统属性中的内联 JSON）的属性。

+   `ServletConfig` 初始化参数。

+   `ServletContext` 初始化参数。

+   来自 *java:comp/env* 的 JNDI 属性。

+   Java 系统属性（`System.getProperties()`）。

+   操作系统环境变量。

+   仅包含 `random.*` 属性的 `RandomValuePropertySource`。

+   打包在 jar 外的特定配置文件的应用程序属性（*application-{profile}.properties* 和 YAML 变体）。

+   打包在 jar 中的特定配置文件的应用程序属性（*application-{profile}.properties* 和 YAML 变体）。

+   打包在 jar 外的应用程序属性（*application.properties* 和 YAML 变体）。

+   打包在 jar 中的应用程序属性（*application.properties* 和 YAML 变体）。

+   `@PropertySource` 注解用于 `@Configuration` 类；请注意，这些属性源在应用程序上下文刷新之前不会添加到 `Environment` 中，这对于在刷新开始之前读取的某些属性（例如 `logging.*` 和 `spring.main.*`）进行配置来说为时已晚。

+   通过设置 `SpringApplication.setDefaultProperties` 指定的默认属性。注意：前述属性源按优先级降序列出：来自列表更高位置的属性将替换来自列表较低位置的相同属性。^(1)

所有这些都可能非常有用，但在本章的代码场景中，我特别选择了其中几个：

+   命令行参数

+   操作系统环境变量

+   打包在 jar 内的应用程序属性（*application.properties* 和 YAML 变体）。

让我们从应用程序的 *application.properties* 文件中定义的属性开始，并逐步深入了解。

## `@Value`

`@Value` 注解可能是将配置设置引入代码中最直接的方法。围绕模式匹配和 Spring 表达语言（SpEL）构建，它简单而强大。

我将从我们应用程序的 *application.properties* 文件中定义一个属性开始，如 图 5-1 所示。

![sbur 0501](img/sbur_0501.png)

###### 图 5-1\. 在 application.properties 中定义 `greeting-name`

为了展示如何使用这个属性，我在应用程序中创建了一个额外的 `@RestController` 来处理与问候应用程序用户相关的任务，如 图 5-2 所示。

![sbur 0502](img/sbur_0502.png)

###### 图 5-2\. 问候 `@RestController` 类

注意，`@Value` 注解适用于 `name` 成员变量，并接受类型为 `String` 的单个名为 `value` 的参数。我使用 SpEL 定义 `value`，将变量名（作为要评估的表达式）放置在 `${` 和 `}` 之间的定界符中。还有一件事需要注意：在这个示例中，SpEL 允许在冒号后设置默认值——“Mirage”，用于在应用程序 `Environment` 中未定义变量的情况。

在执行应用程序并查询 */greeting* 端点时，应用程序如预期地响应“Dakota”，如 图 5-3 所示。

![sbur 0503](img/sbur_0503.png)

###### 图 5-3\. 具有定义属性值的问候响应

为了验证默认值正在被评估，我在 *application.properties* 中以 `#` 注释掉以下行，并重新启动应用程序：

```java
#greeting-name=Dakota
```

查询 *greeting* 端点现在返回 Figure 5-4 中所示的响应。由于应用程序的 `Environment` 中不再定义 `greeting-name`，因此期望的默认值“Mirage”生效了。

![sbur 0504](img/sbur_0504.png)

###### Figure 5-4\. 默认值问候响应

使用自定义属性和 `@Value` 提供了另一个有用的功能：一个属性的值可以使用另一个属性的值进行推导/构建。

为了演示属性嵌套的工作原理，我们至少需要两个属性。我在 *application.properties* 中创建了第二个属性 `greeting-coffee`，如 Figure 5-5 所示。

![sbur 0505](img/sbur_0505.png)

###### Figure 5-5\. 属性值传递给另一个属性

接下来，我向我们的 `GreetingController` 添加了一些代码，以表示一个带有咖啡的问候和我们可以访问以查看结果的端点。请注意，我还为 `coffee` 的值提供了一个默认值，如 Figure 5-6 所示。

![sbur 0506](img/sbur_0506.png)

###### Figure 5-6\. 向 `GreetingController` 添加咖啡问候

为了验证正确的结果，我重新启动应用程序并查询新的 */greeting/coffee* 端点，结果显示在 Figure 5-7 中。请注意，由于问题中的两个属性都在 *application.properties* 中定义，因此显示的值与这些值的定义一致。

![sbur 0507](img/sbur_0507.png)

###### Figure 5-7\. 查询咖啡问候端点

就像生活和软件开发中的所有事物一样，`@Value` 确实有一些限制。由于我们为 `greeting-coffee` 属性提供了一个默认值，我们可以将其在 *application.properties* 中的定义注释掉，`@Value` 注解仍然会使用 `GreetingController` 中的 `coffee` 成员变量正确处理其（默认）值。但是，如果在属性文件中注释掉 `greeting-name` 和 `greeting-coffee` 两者，那么实际上没有任何 `Environment` 源定义它们，进而在应用程序尝试使用 `GreetingController` 中的 (现在未定义的) `greeting-name` 引用 `greeting-coffee` 时会导致以下错误：

```java
org.springframework.beans.factory.BeanCreationException:
    Error creating bean with name 'greetingController':
        Injection of autowired dependencies failed; nested exception is
        java.lang.IllegalArgumentException:
            Could not resolve placeholder 'greeting-name' in value
            "greeting-coffee: ${greeting-name} is drinking Cafe Ganador"
```

###### 注意

完整的堆栈跟踪已删除以提高简洁性和清晰性。

另一个 *application.properties* 中定义的属性并且仅通过 `@Value` 使用的限制是：它们不被 IDE 认为是应用程序使用的，因为它们只在引号限定的 `String` 变量中的代码中被引用；因此，与代码没有直接的关联。当然，开发人员可以通过目视检查属性名称和用法的正确拼写，但这完全是手动操作，因此更容易出错。

正如你所想象的那样，一种类型安全且可验证的属性使用和定义机制将是更好的全面选择。

## @ConfigurationProperties

欣赏`@Value`的灵活性，但也意识到其局限性，Spring 团队创建了`@ConfigurationProperties`。使用`@ConfigurationProperties`，开发者可以定义属性，将相关属性分组，并以可验证和类型安全的方式引用/使用它们。

例如，如果在应用的*application.properties*文件中定义了一个未在代码中使用的属性，开发者将看到其名称被突出显示，以标识其为确认的未使用属性。同样地，如果属性定义为`String`，但与不同类型的成员变量相关联，IDE 将指出类型不匹配。这些都是捕捉简单但频繁错误的宝贵帮助。

为了演示如何使用`@ConfigurationProperties`，我将从定义一个 POJO 开始，用于封装所需的相关属性：在这种情况下，我们先前引用的`greeting-name`和`greeting-coffee`属性。如下所示的代码中，我创建了一个`Greeting`类来保存这两个属性：

```java
class Greeting {
    private String name;
    private String coffee;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getCoffee() {
        return coffee;
    }

    public void setCoffee(String coffee) {
        this.coffee = coffee;
    }
}
```

为了注册`Greeting`以管理配置属性，我添加了如下所示的`@ConfigurationProperties`注解，并指定了用于所有`Greeting`属性的前缀。此注解仅为使用配置属性准备该类；还必须告知应用程序处理这种方式注释的类，以便包含在应用程序`Environment`中的属性。请注意产生的有用错误消息：

![sbur 0508](img/sbur_0508.png)

###### 图 5-8\. 注解和错误

在大多数情况下，指示应用处理`@ConfigurationProperties`类并将其属性添加到应用的`Environment`中最好的方法是将`@ConfigurationPropertiesScan`注解添加到主应用类中，如下所示：

```java
@SpringBootApplication
@ConfigurationPropertiesScan
public class SburRestDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(SburRestDemoApplication.class, args);
    }

}
```

###### 注意

除了要求 Boot 扫描`@ConfigurationProperties`类之外的例外情况是，如果需要有条件地启用某些`@ConfigurationProperties`类或者正在创建自己的自动配置。然而，在所有其他情况下，应使用`@ConfigurationPropertiesScan`来扫描和启用类似于 Boot 组件扫描机制中的`@ConfigurationProperties`类。

为了使用注解处理器生成元数据，使 IDE 能够连接`@ConfigurationProperties`类和*application.properties*文件中定义的相关属性之间的关联，我将以下依赖项添加到项目的*pom.xml*构建文件中：

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

###### 注意

这个依赖项也可以在项目创建时从 Spring Initializr 自动选择并添加。

一旦将配置处理器依赖项添加到构建文件中，就需要刷新/重新导入依赖项并重新构建项目以充分利用它们。要重新导入依赖项，我在 IntelliJ 中打开 Maven 菜单，并点击左上角的重新导入按钮，如 图 5-9 所示。

![sbur 0509](img/sbur_0509.png)

###### 图 5-9\. 重新导入项目依赖

###### 注意

除非选项被禁用，IntelliJ 也会在更改的 *pom.xml* 上方显示一个小按钮，允许快速重新导入，而无需打开 Maven 菜单。重导入按钮位于其底部左侧带有圆形箭头的小 *m*，悬停在第一个依赖项的 `<groupid>` 条目上时可见；当重新导入完成时，它会消失。

一旦更新了依赖项，我从 IDE 中重新构建项目以整合配置处理器。

现在，要为这些属性定义一些值。返回到 *application.properties*，当我开始输入 `greeting` 时，IDE 会显示匹配的属性名，如 图 5-10 所示。

![sbur 0510](img/sbur_0510.png)

###### 图 5-10\. `@ConfigurationProperties` 的完整 IDE 属性支持

要使用这些属性替代之前使用的属性，需要进行一些重构。

我可以完全放弃 `GreetingController` 自己的成员变量 `name` 和 `coffee` 以及它们的 `@Value` 注解；相反，我创建了一个成员变量用于管理 `Greeting` bean 现在管理的 `greeting.name` 和 `greeting.coffee` 属性，并通过构造函数注入到 `GreetingController` 中，如下面的代码所示：

```java
@RestController
@RequestMapping("/greeting")
class GreetingController {
    private final Greeting greeting;

    public GreetingController(Greeting greeting) {
        this.greeting = greeting;
    }

    @GetMapping
    String getGreeting() {
        return greeting.getName();
    }

    @GetMapping("/coffee")
    String getNameAndCoffee() {
        return greeting.getCoffee();
    }
}
```

运行应用程序并查询 *greeting* 和 *greeting/coffee* 端点会得到 图 5-11 中捕获的结果。

![sbur 0511](img/sbur_0511.png)

###### 图 5-11\. 检索 `Greeting` 属性

由 `@ConfigurationProperties` bean 管理的属性仍然从 `Environment` 及其所有潜在来源中获取其值；与基于 `@Value` 的属性相比，唯一显著缺失的是在注解成员变量中指定默认值的能力。乍看起来可能不太理想，但这并不是问题，因为应用程序的 *application.properties* 文件通常用于定义应用程序的合理默认值。如果需要不同的属性值以适应不同的部署环境，这些环境特定的值通过其他来源，例如环境变量或命令行参数，被摄入到应用程序的 `Environment` 中。简而言之，`@ConfigurationProperties` 简单地强制执行了更好的默认属性值的实践。

## 潜在的第三方选项

对于`@ConfigurationProperties`已经令人印象深刻的实用性的进一步扩展，是能够包装第三方组件并将它们的属性合并到应用的`Environment`中。为了演示这一点，我创建了一个 POJO 来模拟一个可能被整合到应用中的组件。请注意，在典型的使用案例中，当这个特性最有用时，我们会向项目添加一个外部依赖，并参考组件的文档来确定创建 Spring bean 的类，而不像我在这里手动创建。

在接下来的代码清单中，我创建了一个模拟的第三方组件称为`Droid`，具有两个属性——`id`和`description`——及其关联的访问器和修改器方法：

```java
class Droid {
    private String id, description;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }
}
```

下一步的操作与真实的第三方组件完全相同：将组件实例化为 Spring bean。可以通过几种方式从定义的 POJO 创建 Spring bean，但对于这个特定的用例来说，最合适的方法是在带有`@Configuration`注解的类中创建一个`@Bean`注解的方法，无论是直接还是通过元注解。

一个将`@Configuration`包含在其定义中的元注解是`@SpringBootApplication`，它位于主应用程序类上。这就是为什么开发人员经常将 bean 创建方法放在这里的原因。

###### 注意

在 IntelliJ 和大多数其他具有良好 Spring 支持的 IDE 和高级文本编辑器中，可以深入探索 Spring 元注解来探索嵌套的注解。在 IntelliJ 中，Cmd+LeftMouseClick（在 MacOS 上）将展开注解。`@SpringBootApplication`包含`@SpringBootConfiguration`，它包含`@Configuration`，使得与凯文·贝肯（Kevin Bacon）仅有两度分离。

在接下来的代码清单中，我演示了 bean 创建方法以及必需的`@ConfigurationProperties`注解和`prefix`参数，指示应将`Droid`属性合并到`Environment`中的顶层属性组`droid`中：

```java
@SpringBootApplication
@ConfigurationPropertiesScan
public class SburRestDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(SburRestDemoApplication.class, args);
    }

    @Bean
    @ConfigurationProperties(prefix = "droid")
    Droid createDroid() {
        return new Droid();
    }
}
```

正如以往一样，需要重新构建项目以便配置处理器检测到此新配置属性源暴露的属性。执行构建后，我们可以返回到*application.properties*，看到`droid`属性现在完整地展现出来，包括类型信息，如图 5-12 所示。

![sbur 0512](img/sbur_0512.png)

###### 图 5-12\. `droid`属性和类型信息现在在*application.properties*中可见

我为`droid.id`和`droid.description`分配了一些默认值，用作默认值，如图 5-13 所示。对于所有的`Environment`属性来说，这是一个养成的好习惯，即使是从第三方获取的属性也不例外。

![sbur 0513](img/sbur_0513.png)

###### 图 5-13。在 *application.properties* 中分配了默认值的 `droid` 属性

为了验证一切是否按预期运行与 `Droid` 属性，我创建了一个非常简单的 `@RestController`，其中包含一个单独的 `@GetMapping` 方法，如下所示的代码：

```java
@RestController
@RequestMapping("/droid")
class DroidController {
    private final Droid droid;

    public DroidController(Droid droid) {
        this.droid = droid;
    }

    @GetMapping
    Droid getDroid() {
        return droid;
    }
}
```

构建并运行项目后，我查询新的 */droid* 端点，并确认适当的响应，如 图 5-14 中所示。

![sbur 0514](img/sbur_0514.png)

###### 图 5-14。查询 */droid* 端点以检索来自 Droid 的属性

# 自动配置报告

如前所述，Boot 通过自动配置为开发人员执行了大量操作：使用所选功能、依赖项和代码来设置应用程序所需的 bean，从而实现所选功能和代码。还提到了根据用例更具体（符合您的用例）地实现功能所需的任何自动配置的能力。但是，如何查看创建了哪些 bean、未创建哪些 bean，以及是什么条件促使了其中任一结果呢？

使用 JVM 的灵活性，通过在几种方式之一中使用 `debug` 标志，可以很容易地生成自动配置报告：

+   使用 `--debug` 选项执行应用程序的 jar 文件：`java -jar bootapplication.jar --debug`

+   使用 JVM 参数执行应用程序的 jar 文件：`java -Ddebug=true` `-jar bootapplication.jar`

+   将 `debug=true` 添加到应用程序的 *application.properties* 文件

+   在您的 shell（Linux 或 Mac）中执行 `export DEBUG=true` 或将其添加到 Windows 环境中，然后运行 `java -jar bootapplication.jar`

###### 注

任何方式将肯定值添加到应用程序的 `Environment` 中，如前所述，都将提供相同的结果。这些只是更常用的选项。

自动配置报告的部分列出了正匹配的条件——这些条件评估为真，并导致执行操作的条件——列在以“Positive matches”为标题的部分中。我在这里复制了该部分标题，以及一个正匹配及其相应的自动配置操作的示例：

```java
============================
CONDITIONS EVALUATION REPORT
============================

Positive matches:
-----------------
   DataSourceAutoConfiguration matched:
      - @ConditionalOnClass found required classes 'javax.sql.DataSource',
      'org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseType'
      (OnClassCondition)
```

这种匹配符合我们的预期，尽管确认以下内容总是好的：

+   JPA 和 H2 是应用程序依赖项。

+   JPA 与 SQL 数据源一起使用。

+   H2 是一个嵌入式数据库。

+   找到支持嵌入式 SQL 数据源的类。

结果，调用了 `DataSourceAutoConfiguration`。

同样，"Negative matches" 部分显示了 Spring Boot 自动配置未执行的操作以及原因，如下所示：

```java
Negative matches:
-----------------
   ActiveMQAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required class
          'javax.jms.ConnectionFactory' (OnClassCondition)
```

在这种情况下，未执行 `ActiveMQAutoConfiguration`，因为应用程序在启动时未找到 JMS `ConnectionFactory` 类。

另一个有用的信息片段是列出“无条件类”的部分，这些类无需满足任何条件即可创建。鉴于前一节，我列出了一个特别感兴趣的类：

```java
Unconditional classes:
----------------------
    org.springframework.boot.autoconfigure.context
     .ConfigurationPropertiesAutoConfiguration
```

正如您所见，`ConfigurationPropertiesAutoConfiguration`始终被实例化，以管理 Spring Boot 应用程序中创建和引用的任何`ConfigurationProperties`；它是每个 Spring Boot 应用程序的一部分。

# 执行器

执行器

n. 特指：用于移动或控制某物的机械装置

Spring Boot Actuator 的原始版本于 2014 年达到了一般可用性（GA），因为它为生产中的 Boot 应用程序提供了宝贵的洞察。通过 HTTP 端点或 Java 管理扩展（JMX）提供正在运行的应用程序的监控和管理功能，Actuator 涵盖并公开了 Spring Boot 的所有生产准备功能。

随着 Spring Boot 2.0 版本的彻底改造，Actuator 现在利用 Micrometer 仪表化库提供度量标准，通过与多种主流监控系统的一致外观类似于 SLF4J 处理各种日志机制，大大扩展了可以在任何给定的 Spring Boot 应用程序中通过执行器集成、监控和公开的范围。

要开始使用执行器，我将在当前项目的`pom.xml`依赖项部分中添加另一个依赖项。如下片段所示，`spring-boot-starter-actuator`依赖项提供了必要的功能；为此，它将 Actuator 本身和 Micrometer 一起带到 Spring Boot 应用程序中，并具备几乎零配置的自动配置能力：

```java
<dependencies>
    ... (other dependencies omitted for brevity)
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

刷新/重新导入依赖后，我再次运行应用程序。应用程序运行时，通过访问其主要端点，我们可以查看执行器默认公开的信息。再次使用 HTTPie 完成此操作，如图 5-15 所示。

![sbur 0515](img/sbur_0515.png)

###### 图 5-15\. 访问执行器端点，默认配置

###### 注

执行器的所有信息默认情况下都集中在应用程序的*/actuator*端点下，但这也是可配置的。

这似乎并不像执行器所创造的那么热闹（和粉丝基地）。但这种简洁是有意的。

执行器可以访问并公开有关正在运行的应用程序的大量信息。此信息对开发人员、运营人员以及可能希望威胁您应用程序安全性的不良个人尤为有用。遵循 Spring Security 的默认安全目标，执行器的自动配置仅公开非常有限的*health*和*info*响应——事实上，*info*默认为空集，提供应用程序心跳和其他少量信息（OOTB）。

与大多数 Spring 的事物一样，您可以创建一些非常复杂的机制来控制对各种 Actuator 数据源的访问，但也有一些快速、一致和低摩擦的选项可用。 现在让我们来看看这些。

可以通过属性轻松配置 Actuator，要么使用一组包含的端点，要么使用一组排除的端点。 为了简单起见，我选择了包含路线下内容添加到 *application.properties* 中：

```java
management.endpoints.web.exposure.include=env, info, health
```

在此示例中，我指示应用程序（和 Actuator）仅暴露*/actuator/env*、*/actuator/info*和*/actuator/health*端点（和任何下级端点）。

图 5-16 在重新运行应用程序并查询其*/actuator*端点后确认了预期结果。

为了充分展示 Actuator 的 OOTB 能力，我可以进一步禁用安全性，*仅用于演示目的*，通过使用前述的*application.properties* 设置的通配符：

```java
management.endpoints.web.exposure.include=*
```

###### 警告

这一点不容忽视：对于敏感数据的安全机制应该仅在演示或验证目的下禁用。*永远不要为生产应用程序禁用安全性*。

![sbur 0516](img/sbur_0516.png)

###### 图 5-16\. 在指定要包括的端点后访问 Actuator

启动应用程序时进行验证，Actuator 忠实地报告了它当前正在暴露的端点数量和到达它们的根路径——在这种情况下，默认为*/actuator*——如下所示的启动报告片段。 这是一个有用的提醒/警告，可以快速进行视觉检查，以确保在将应用程序推进到目标部署之前不会暴露更多端点：

```java
INFO 22115 --- [           main] o.s.b.a.e.web.EndpointLinksResolver      :
    Exposing 13 endpoint(s) beneath base path '/actuator'
```

要检查通过 Actuator 当前可访问的所有映射，只需查询提供的 Actuator 根路径以检索完整列表：

```java
mheckler-a01 :: ~/dev » http :8080/actuator
HTTP/1.1 200
Connection: keep-alive
Content-Type: application/vnd.spring-boot.actuator.v3+json
Date: Fri, 27 Nov 2020 17:43:27 GMT
Keep-Alive: timeout=60
Transfer-Encoding: chunked

{
    "_links": {
        "beans": {
            "href": "http://localhost:8080/actuator/beans",
            "templated": false
        },
        "caches": {
            "href": "http://localhost:8080/actuator/caches",
            "templated": false
        },
        "caches-cache": {
            "href": "http://localhost:8080/actuator/caches/{cache}",
            "templated": true
        },
        "conditions": {
            "href": "http://localhost:8080/actuator/conditions",
            "templated": false
        },
        "configprops": {
            "href": "http://localhost:8080/actuator/configprops",
            "templated": false
        },
        "env": {
            "href": "http://localhost:8080/actuator/env",
            "templated": false
        },
        "env-toMatch": {
            "href": "http://localhost:8080/actuator/env/{toMatch}",
            "templated": true
        },
        "health": {
            "href": "http://localhost:8080/actuator/health",
            "templated": false
        },
        "health-path": {
            "href": "http://localhost:8080/actuator/health/{*path}",
            "templated": true
        },
        "heapdump": {
            "href": "http://localhost:8080/actuator/heapdump",
            "templated": false
        },
        "info": {
            "href": "http://localhost:8080/actuator/info",
            "templated": false
        },
        "loggers": {
            "href": "http://localhost:8080/actuator/loggers",
            "templated": false
        },
        "loggers-name": {
            "href": "http://localhost:8080/actuator/loggers/{name}",
            "templated": true
        },
        "mappings": {
            "href": "http://localhost:8080/actuator/mappings",
            "templated": false
        },
        "metrics": {
            "href": "http://localhost:8080/actuator/metrics",
            "templated": false
        },
        "metrics-requiredMetricName": {
            "href": "http://localhost:8080/actuator/metrics/{requiredMetricName}",
            "templated": true
        },
        "scheduledtasks": {
            "href": "http://localhost:8080/actuator/scheduledtasks",
            "templated": false
        },
        "self": {
            "href": "http://localhost:8080/actuator",
            "templated": false
        },
        "threaddump": {
            "href": "http://localhost:8080/actuator/threaddump",
            "templated": false
        }
    }
}
```

Actuator 端点列表提供了捕获和暴露供检查的信息范围的良好概念，但对于好的和坏的行为者特别有用的是以下内容：

*/actuator/beans*

应用程序创建的所有 Spring bean

*/   */actuator/conditions*

创建 Spring bean 所需满足的条件（或不满足的条件）； 类似于之前讨论过的条件评估报告

*/actuator/configprops*

应用程序可访问的所有`Environment` 属性

*/actuator/env*

应用程序正在运行的环境的种种方面； 特别有用的是看到每个个体`configprop` 值的来源

*/actuator/health*

健康信息（基本或扩展，取决于设置）

*/actuator/heapdump*

启动堆转储以进行故障排除和/或分析

*/actuator/loggers*

每个组件的日志级别

*/actuator/mappings*

所有端点映射和支持详细信息

*/actuator/metrics*

应用程序当前正在捕获的指标

*/actuator/threaddump*

启动线程转储以进行故障排除和/或分析

这些，以及其余的预配置执行器端点，在需要时都非常方便，并且易于访问进行检查。继续专注于应用程序的环境，即使在这些端点之中也有同行中的先例。

## 让执行器打开

正如提到的，执行器的默认安全姿态有意仅公开非常有限的*health*和*info*响应。实际上，*/actuator/health*端点提供了一个相当实用的应用程序状态“UP”或“DOWN”。

大多数应用程序都有依赖关系，执行器会跟踪健康信息；除非获得授权，否则它不会公开其他信息。为了显示预配置依赖项的扩展健康信息，我将以下属性添加到*application.properties*中：

```java
management.endpoint.health.show-details=always
```

###### 注意

健康指标的`show-details`属性有三个可能的取值：`never`（默认值）、`when_authorized`和`always`。在这个例子中，我选择`always`仅仅是为了演示可能性，但对于每个投入生产的应用程序，正确的选择应该是要么`never`，要么`when_authorized`，以限制应用程序扩展的健康信息的可见性。

重新启动应用程序会导致将应用程序的主要组件的健康信息添加到访问*/actuator/health*端点时的整体应用程序健康摘要中，参见图 5-17。

![sbur 0517](img/sbur_0517.png)

###### 图 5-17\. 扩展健康信息

## 使用执行器更环保意识

开发者经常会受到一种毛病的困扰——包括在内的现在这个公司——即当行为与预期不符时，完全了解当前应用程序环境/状态的假设。这并不完全出乎意料，尤其是如果是自己写的异常代码。一个相对快速且非常宝贵的第一步是*检查所有的假设*。你*知道*那个值是多少吗？还是你只是确信你知道？

你有检查过吗？

特别是在输入驱动结果的代码中，这应该是一个必需的起点。执行器帮助使这一过程变得轻松。查询应用程序的*/actuator/env*端点返回所有环境信息；以下是该结果的部分，仅显示到目前为止在应用程序中设置的属性：

```java
{
    "name": "Config resource 'classpath:/application.properties' via location
     'optional:classpath:/'",
    "properties": {
        "droid.description": {
            "origin": "class path resource [application.properties] - 5:19",
            "value": "Small, rolling android. Probably doesn't drink coffee."
        },
        "droid.id": {
            "origin": "class path resource [application.properties] - 4:10",
            "value": "BB-8"
        },
        "greeting.coffee": {
            "origin": "class path resource [application.properties] - 2:17",
            "value": "Dakota is drinking Cafe Cereza"
        },
        "greeting.name": {
            "origin": "class path resource [application.properties] - 1:15",
            "value": "Dakota"
        },
        "management.endpoint.health.show-details": {
            "origin": "class path resource [application.properties] - 8:41",
            "value": "always"
        },
        "management.endpoints.web.exposure.include": {
            "origin": "class path resource [application.properties] - 7:43",
            "value": "*"
        }
    }
}
```

执行器不仅显示每个定义属性的当前值，还显示其来源，甚至显示定义每个值的行号和列号。但是，如果其中一个或多个值被另一个来源覆盖，例如在执行应用程序时由外部环境变量或命令行参数？

为了演示一个典型的生产绑定应用程序场景，我从应用程序的目录中使用命令行运行`mvn clean package`，然后使用以下命令执行应用程序：

```java
java -jar target/sbur-rest-demo-0.0.1-SNAPSHOT.jar --greeting.name=Sertanejo
```

再次查询 */actuator/env*，您可以看到有一个新的命令行参数部分，其中只有一个条目`greeting.name`：

```java
{
    "name": "commandLineArgs",
    "properties": {
        "greeting.name": {
            "value": "Sertanejo"
        }
    }
}
```

遵循之前提到的`Environment`输入的优先顺序，命令行参数应该覆盖*application.properties*内部设置的值。查询 */greeting* 端点返回预期的“Sertanejo”；同样，通过 SpEL 表达式查询 */greeting/coffee* 也将覆盖的值合并到响应中：`Sertanejo is drinking Cafe Cereza`。

通过 Spring Boot Actuator，试图弄清楚错误的、数据驱动行为变得更加简单。

## 使用 Actuator 增加日志记录的音量

与开发和部署软件中的许多其他选择一样，为生产应用程序选择日志记录级别涉及权衡。选择更多日志记录会导致更多的系统级工作和存储消耗，以及捕获更多相关和不相关的数据。这反过来可能会使寻找难以捉摸的问题变得更加困难。

作为提供 Boot 生产就绪功能的使命的一部分，Actuator 也解决了这个问题，允许开发人员为大多数或所有组件设置典型的日志级别，当关键问题出现时可以临时更改这个级别……所有这些都是在现场、生产中的 Spring Boot 应用中进行的。Actuator 通过向适用的端点发送简单的`POST`请求来便捷地设置和重置日志级别。例如，图 5-18 显示了`org.springframework.data.web`的默认日志级别。

![sbur 0518](img/sbur_0518.png)

###### 图 5-18\. `org.springframework.data.web`的默认日志级别

特别值得注意的是，由于未为此组件配置日志级别，因此使用了有效级别“INFO”。再次强调，当未提供具体配置时，Spring Boot 会提供合理的默认设置。

如果我收到有关正在运行的应用程序的问题并希望增加日志记录以帮助诊断和解决问题，那么为了特定组件执行此操作所需的全部步骤就是向其 */actuator/loggers* 端点发布一个新的 JSON 格式的`configuredLevel`值，如下所示：

```java
echo '{"configuredLevel": "TRACE"}'
  | http :8080/actuator/loggers/org.springframework.data.web
```

现在重新查询日志级别，确认 `org.springframework.data.web` 的记录器现在设置为“TRACE”，将为应用程序提供详尽的诊断日志记录，如图 5-19 所示。

###### 警告

“TRACE”在确定难以捉摸的问题时可能至关重要，但它是一个相当重量级的日志级别，甚至比“DEBUG”还要详细——在生产应用程序中使用可以提供重要的信息，但要注意其影响。

![sbur 0519](img/sbur_0519.png)

###### 图 5-19\. `org.springframework.data.web`的新“TRACE”日志级别

# 代码检查

获取完整的章节代码，请从代码库的*chapter5end*分支查看。

# 总结

开发人员必须拥有有用的工具来建立、识别和隔离生产应用程序中表现出的行为是至关重要的。随着应用程序变得越来越动态和分布式，通常需要做以下工作：

+   配置和动态重新配置应用程序

+   确定/确认当前设置及其来源

+   检查和监控应用程序环境和健康指标

+   临时调整实时应用程序的日志级别以识别根本原因

本章展示了如何利用 Spring Boot 的内置配置能力、其自动配置报告和 Spring Boot 执行器来灵活动态地创建、识别和修改应用程序环境设置。

在下一章中，我将深入探讨数据：如何使用各种行业标准和领先的数据库引擎定义其存储和检索，以及 Spring Data 项目和工具如何以最简单和强大的方式支持它们的使用。

^(1) [Spring Boot 属性源的优先级顺序](https://oreil.ly/OrderPredSB)。
