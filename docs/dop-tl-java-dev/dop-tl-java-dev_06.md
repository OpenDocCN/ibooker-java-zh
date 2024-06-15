# 第六章：包管理

伊什切尔·鲁伊斯

当你阅读这句话时，世界的某个地方正在编写一行代码。这行代码最终将成为一个工件的一部分，该工件将成为组织内部一个或多个企业产品中使用的构建块，或者通过公共存储库（尤其是 Java 和 Kotlin 库的 Maven 中央库）共享。

当今比以往任何时候都提供了更多的库、二进制文件和工件，随着全球开发人员继续推进下一代产品和服务，这一收藏还将继续增长。处理和管理这些工件现在比以往任何时候都需要更多的努力——由于依赖项数量不断增加，形成了一个复杂的连接网络。使用错误版本的工件是一个容易陷入的陷阱，导致混乱和构建失败，最终阻碍精心计划的项目发布日期。

对于开发人员来说，现在比以往任何时候都更加重要的是不仅要理解他们面前的源代码的功能和特殊之处，还要了解他们的项目如何打包以及如何将构建块组装成最终产品。深入了解构建过程本身以及我们的自动构建工具在幕后如何运作，对于避免延误和几小时的不必要故障排除至关重要——更不用说防止大量的错误进入生产环境了。

访问提供常见编码问题解决方案的大量第三方资源可以帮助加速我们项目的开发，但也引入了错误或意外行为的风险。了解这些组件如何进入项目，以及它们来自何处，将有助于故障排除工作。确保我们对内部生产的工件负责任将使我们能够在决定缺陷修复和功能开发的优先级时提升我们的决策能力，并有助于铺平通向生产发布的道路。开发人员不再只精通于他们面前代码的语义，还要了解包管理的复杂性。

# 为什么“构建和交付”不再足够？

不久以前，软件开发人员认为构建一个工件是辛苦甚至有时史诗般的努力的顶点。有时候满足截止日期意味着采用捷径和文档不全的步骤。自那时以来，行业的要求已经发生了变化，以带来更快的交付周期、多样化的环境、定制的工件、爆炸性的代码库和存储库以及多模块包。今天，构建一个工件只是更大商业周期的一个步骤之一。

成功的领导者认识到，最好的创新源于试错。这就是为什么他们把测试、实验和失败作为他们生活和公司流程的一个必不可少的部分的原因。

一种创新、快速扩展、推出更多产品、改善应用程序或产品的质量或用户体验、推出新功能的方法是通过 A/B 测试。什么是 A/B 测试？根据在哥伦比亚大学创办应用分析项目的 Kaiser Fung 所说，*A/B 测试* 在其最基本的层面上是一种用于比较两个版本的东西以确定哪个表现更好的百年老方法。今天，几家初创公司、微软等大型公司以及其他几家领先公司—包括亚马逊、Booking.com、Facebook 和 Google—每年都在进行[超过 10,000 次的在线控制实验](https://oreil.ly/vRKPP)。

Booking.com 对其网站上的每个新功能进行比较测试，比较从照片和内容的选择到按钮颜色和位置的细节。通过将几个版本相互比较并跟踪客户反馈，该公司能够不断改善用户体验。

我们如何交付和部署由众多工件组成的多个软件版本？我们如何找到瓶颈？我们如何知道我们正朝着正确的方向发展？我们如何跟踪工作得很好或者对我们不利的情况？我们如何保持可重现的结果，但又具有丰富的谱系？这些问题的答案可以通过捕获和分析有关工作流程和工件的输入、输出和状态的相关、上下文、清晰和具体信息来找到。所有这些都是可能的，这要归功于元数据。

# 全都是关于元数据

正如 W. Edwards Deming 所说：“信仰上帝；所有其他人带来数据。” *元数据* 被定义为相关信息的结构化键/值存储。换句话说，它是适用于特定实体的属性或特性的集合，在我们的案例中适用于工件和流程。

元数据使得能够发现相关性和因果关系，以及对组织行为和结果的洞察。因此，元数据可以显示组织是否对其利益相关者的目标保持敏感。

附加数据可以在后期用于提取或衍生更多信息。这些数据有助于扩展视角并创造更多故事或叙述。选择添加哪些属性、基数和值很重要—太多了会影响性能；太少了会错过信息。如果值太多，洞察力就会丧失。

一个好的起点是回答关于软件开发周期每个阶段的主要阶段的以下问题：谁？什么？如何？在哪里？何时？提出正确的问题只是一半的工作，然而，拥有清晰、相关、具体和清晰的答案，并能进行归一化或枚举，总是一个好的做法。

## 有见地的元数据的关键属性

有见地的数据应具备以下所有特点：

上下文化

所有数据都需要在参考框架内进行解释。为了提取和比较可能的场景，分析阶段非常重要。

相关

值的可变性对结果产生影响，或者描述结果或过程中的特定阶段或时间。

具体

值描述清晰的事件（即初始值、结束值）。

清晰

可能的值是众所周知或定义的，可计算且可比较的。

独特

具有单一、独特的值。

可扩展

因为人类知识的丰富程度在不断增加，所以数据需要定义机制，以便标准可以进化和扩展，以适应新属性。

一旦您已经定义了记录软件开发周期阶段、输入、输出和状态的内容的“什么”、“何时”、“为什么”和“如何”，您还需要牢记元数据子集的消费者。一方面，您可能会有一个中间的私有消费者，该消费者将以不同方式消费和响应一组值，从触发子流水线、推广构建、在不同环境中部署，或发布工件。另一方面，您可能会有最终的外部消费者，他们能够提取信息，并通过技能和经验将其转化为洞察力，以帮助实现组织的整体目标。

## 元数据考虑

以下是关于元数据的重要考虑因素：

隐私与安全

考虑清楚暴露值的后果。

可见性

并非所有消费者都对所有数据感兴趣。

格式和编码

一种特定的属性可能在不同阶段以不同格式暴露，但在命名、含义及可能的一般值上需要保持一致。

现在让我们转向使用构建工具生成和打包元数据。在 Java 生态系统中，构建工具众多，其中最流行的可能是 Apache Maven 和 Gradle；因此，深入讨论它们是有意义的。然而，如果您的构建依赖于不同的构建工具，本节提供的信息仍然可能很有用，因为一些收集和打包元数据的技术可能会被重复使用。

现在，在我们开始实际的代码片段之前，我们必须明确三个行动项：

1.  确定应该与工件打包的元数据。

1.  弄清楚在构建过程中如何获取元数据。

1.  处理元数据，并以适当的格式或格式记录它。

以下子节涵盖了每个方面。

## 确定元数据

构建环境中有大量信息可以转换为元数据并与构件一起打包。一个很好的例子是构建时间戳，它标识了构建生成构件的时间和日期。可以遵循许多时间戳格式，但我建议使用[ISO 8601](https://oreil.ly/PsZkB)，其使用`java.text.SimpleDateformat`格式化表示为`yyyy-MM-dd'T'HH:mm:ssXXX`——当捕获时间戳依赖于`java.util.Date`时非常有用。或者，如果捕获时间戳依赖于`java.time.LocalDateTime`，则可以使用`java.time.format.DateTimeFormatter.ISO_OFFSET_DATE_TIME`。构建的操作系统详细信息也可能是感兴趣的，以及 JDK 信息，如版本、ID 和供应商。幸运的是，这些信息由 JVM 捕获并通过[`System`](https://oreil.ly/CKMsE)属性公开。

考虑包括构件的 ID 和版本（即使这些值通常编码在构件的文件名中），作为一种预防措施，以防构件在某些时候被重命名。版本控制信息也至关重要。从源代码控制中获取的有用信息包括提交哈希、标签和分支名称。此外，您可能希望捕获特定的构建信息，例如运行构建的用户、构建工具的名称、ID 和版本，以及构建机器的主机名和 IP 地址。这些键/值对可能是最重要和最常见的元数据。但是，您可能还需要选择其他工具和系统所需的额外键/值对，以消费生成的构件。

###### 注意

强调检查团队和组织关于敏感数据访问和可见性的政策有多么重要是不够的。如果敏感数据键/值对被第三方或外部消费者访问，这些键/值对可能被视为安全风险，尽管它们对内部消费者非常重要。

## 获取元数据

在确定需要捕获哪些元数据之后，我们必须找到一种使用我们选择的构建工具来收集元数据的方法。某些键/值对可以直接从环境、系统设置和由 JVM 公开的命令标志（作为环境变量或`System`属性）中获取。额外的属性可能由构建工具本身公开，无论是作为额外的命令行参数还是作为工具配置设置中的配置元素。

让我们假设我们需要在某一时刻捕获以下键/值对：

+   JDK 信息，例如版本和供应商

+   操作系统信息，如名称、架构和版本

+   构建时间戳

+   来自版本控制系统的当前提交哈希（假设为 Git）

这些值可以通过 Maven 捕获，通过使用前两个项目的`System`属性的组合和第三方插件来实现最后两个项目。当涉及到提供与 Git 集成的插件时，Maven 和 Gradle 都有丰富的选择；然而，我推荐选择[Maven 的 git-commit-id 插件](https://oreil.ly/EwiLP)和[Gradle 的 versioning 插件](https://oreil.ly/qjEOi)，因为这些插件到目前为止是最通用的。

现在，Maven 允许以多种方式定义属性，最常见的是作为*pom.xml*构建文件中`<properties>`部分内的键/值对。每个键的值都是自由文本，尽管您可以使用简化表示法引用`System`属性或使用命名约定引用环境变量。假设您想访问在`System`属性中找到的`java.version`键的值。这可以通过使用`${}`简写表示法，如`${java.version}`来完成。相反，对于环境变量，您可以使用`${env.*NAME*}`表示法。例如，可以使用`${env.TOKEN}`表达式在*pom.xml*构建文件中访问名为`TOKEN`的环境变量的值。将`git-commit-id`插件和构建属性组合在一起可能会导致类似以下的*pom.xml*文件：

```java
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.acme</groupId>
  <artifactId>example</artifactId>
  <version>1.0.0-SNAPSHOT</version>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <build.jdk>${java.version} (${java.vendor} ${java.vm.version})</build.jdk>
    <build.os>${os.name} ${os.arch} ${os.version}</build.os>
    <build.revision>${git.commit.id}</build.revision>
    <build.timestamp>${git.build.time}</build.timestamp>
  </properties>

  <build>
    <plugins>
      <plugin>
        <groupId>pl.project13.maven</groupId>
        <artifactId>git-commit-id-plugin</artifactId>
        <version>4.0.3</version>
        <executions>
          <execution>
            <id>resolve-git-properties</id>
            <goals>
              <goal>revision</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <verbose>false</verbose>
          <failOnNoGitDirectory>false</failOnNoGitDirectory>
          <generateGitPropertiesFile>true</generateGitPropertiesFile>
          <generateGitPropertiesFilename>
            ${project.build.directory}/git.properties
          </generateGitPropertiesFilename>
          <dateFormat>yyyy-MM-dd'T'HH:mm:ssXXX</dateFormat>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

注意到`build.jdk`和`build.os`的值已经包含格式，因为它们是更简单值的组合，而`build.revision`和`build.timestamp`的值来自 Git 插件定义的属性。我们还未确定最终的格式和包含元数据的文件或文件，这就是为什么我们看到它在`<properties>`部分中定义的原因。这种设置允许这些值在需要时被其他插件重用和消费。选择这种设置的另一个原因是，外部工具（比如在构建流水线中找到的工具）可以更容易地读取这些值，因为它们位于特定部分，而不是在构建文件的多个位置。

还要注意所选择的版本值，`1.0.0-SNAPSHOT`。您可以根据需要使用任何字符组合作为版本。然而，习惯上至少使用定义两个数字的`*major*`.`*minor*`格式的字母数字序列。存在多种版本控制约定，各有优缺点。尽管如此，使用`-SNAPSHOT`标签具有特殊含义，因为它表示该构件尚未准备好投入生产。一些工具在检测到快照版本时会有不同的行为；例如，它们可以阻止构件发布到生产环境。

与 Maven 相比，Gradle 在定义和编写构建文件时没有短缺的选项。首先，自 Gradle 4 以来，您可以选择两种构建文件格式：Apache Groovy DSL 或 Kotlin DSL。无论您选择哪种，您很快会发现有更多选项可以捕获和格式化元数据。其中一些可能是习惯用法的，有些可能需要额外的插件，甚至有些可能被视为过时或已废弃。为了简单起见，我们将采用 Groovy 和小型习惯用法表达式。我们将类似于 Maven 一样捕获相同的元数据，前两个值来自`System`属性和由`versioning` Git 插件提供的提交哈希，但构建时间戳将通过使用自定义代码即时计算。以下代码片段显示了如何实现这一点：

```java
plugins {
  id 'java-library'
  id 'net.nemerosa.versioning' version '2.14.0'
}

version = '1.0.0-SNAPSHOT'

ext {
  buildJdk = [
    System.properties['java.version'],
    '(' + System.properties['java.vendor'],
    System.properties['java.vm.version'] + ')'
  ].join(' ')
  buildOs = [
    System.properties['os.name'],
    System.properties['os.arch'],
    System.properties['os.version']
  ].join(' ')
  buildRevision = project.extensions.versioning.info.commit
  buildTimestamp = new Date().format("yyyy-MM-dd'T'HH:mm:ssXXX")
}
```

这些计算得出的数值将作为动态项目属性提供，稍后可以通过其他配置的元素（如扩展、任务、Groovy 的闭包、Groovy 和 Kotlin 的动作以及 DSL 公开的其他元素）在构建过程中消耗。现在只剩下将元数据记录在指定的格式中。

## 写入元数据

您可能需要记录多种格式或文件中的元数据。格式的选择取决于预期的消费者。有些消费者需要一种独特的格式，其他消费者无法读取，而其他人可能理解各种格式。请务必查阅特定消费者的文档，了解其支持的格式和选项，并检查您选择的构建工具是否提供了集成。您可能会发现，您的构建工具中提供了插件，可以简化您所需的元数据记录过程。为了演示目的，我们将使用两种流行格式记录元数据：Java 属性文件和 JAR 的清单。

我们可以利用 Maven 的[资源过滤](https://oreil.ly/X1x0q)，这已经集成到[资源插件](https://oreil.ly/YqOSO)中，是每个构建都可以访问的核心插件之一。为了使其工作，我们必须将以下代码片段添加到前面的*pom.xml*文件中的`<build>`部分：

```java
<resources>
  <resource>
    <directory>src/main/resources</directory>
    <filtering>true</filtering>
  </resource>
</resources>
```

还需要一个位于*src/main/resources*的配套属性文件。我选择*META-INF/metadata.properties*作为位于工件 JAR 内部的属性文件的相对路径和名称。当然，您可以根据需要选择不同的命名约定。此文件依赖于变量占位符替换，这些变量将从项目属性中解析，例如我们在`<properties>`部分设置的那些。按照惯例，在构建文件中几乎不需要配置信息。属性文件看起来像这样：

```java
build.jdk       = ${build.jdk}
build.os        = ${build.os}
build.revision  = ${build.revision}
build.timestamp = ${build.timestamp}
```

在 JAR 的清单中记录元数据需要调整 `jar-maven-plugin` 的配置，适用于构建文件。下面的片段必须包含在 `<build>` 部分中的 `<plugins>` 部分内。换句话说，它与本节前面看到的 `git-commit-id` 插件是同级的：

```java
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-jar-plugin</artifactId>
  <version>3.2.0</version>
  <configuration>
    <archive>
      <manifestEntries>
        <Build-Jdk>${build.jdk}</Build-Jdk>
        <Build-OS>${build.os}</Build-OS>
        <Build-Revision>${build.revision}</Build-Revision>
        <Build-Timestamp>${build.timestamp}</Build-Timestamp>
      </manifestEntries>
    </archive>
  </configuration>
</plugin>
```

注意，即使此插件是核心插件集的一部分，也定义了特定的插件版本。背后的原因是必须声明所有插件版本以便实现可重复的构建。否则，你会发现构建可能会有所不同，因为根据运行构建所使用的 Maven 版本的不同，可能会解析不同的插件版本。清单中的每个条目由大写键和捕获值组成。使用 `mvn package` 运行构建会解析捕获的属性，将解析后的值复制到 *target/classes* 目录中的元数据属性文件中，该文件将被添加到最终 JAR 中，并且将元数据注入到 JAR 的清单中。我们可以通过检查生成的构件的内容来验证这一点：

```java
$ mvn verify
$ jar tvf target/example-1.0.0-SNAPSHOT.jar
    0 Sun Jan 10 20:41 CET 2021 META-INF/
  131 Sun Jan 10 20:41 CET 2021 META-INF/MANIFEST.MF
  205 Sun Jan 10 20:41 CET 2021 META-INF/metadata.properties
    0 Sun Jan 10 20:41 CET 2021 META-INF/maven/
    0 Sun Jan 10 20:41 CET 2021 META-INF/maven/com.acme/
    0 Sun Jan 10 20:41 CET 2021 META-INF/maven/com.acme/example/
 1693 Sun Jan 10 19:13 CET 2021 META-INF/maven/com.acme/example/pom.xml
  109 Sun Jan 10 20:41 CET 2021 META-INF/maven/com.acme/example/pom.properties
```

两个文件如预期般在 JAR 文件中找到。提取 JAR 并查看属性文件和 JAR 清单的内容将得到以下结果：

```java
build.jdk       = 11.0.9 (Azul Systems, Inc. 11.0.9+11-LTS)
build.os        = Mac OS X x86_64 10.15.7
build.revision  = 0ab9d51a3aaa17fca374d28be1e3f144801daa3b
build.timestamp = 2021-01-10T20:41:11+01:00
```

```java
Manifest-Version: 1.0
Created-By: Maven Jar Plugin 3.2.0
Build-Jdk-Spec: 11
Build-Jdk: 11.0.9 (Azul Systems, Inc. 11.0.9+11-LTS)
Build-OS: Mac OS X x86_64 10.15.7
Build-Revision: 0ab9d51a3aaa17fca374d28be1e3f144801daa3b
Build-Timestamp: 2021-01-10T20:41:11+01:00
```

你已经看到如何使用 Maven 收集元数据。现在让我们看看通过另一个构建工具 Gradle 来记录元数据的相同方法。首先，我们将配置由我们应用于构建的 `java-library` 插件提供的标准 `processResources` 任务。额外的配置可以追加到之前展示的 Gradle 构建文件中，如下所示：

```java
processResources {
  expand(
    'build_jdk'      : project.buildJdk,
    'build_os'       : project.buildOs,
    'build_revision' : project.buildRevision,
    'build_timestamp': project.buildTimestamp
  )
}
```

注意到键名中使用 `_` 作为令牌分隔符，这是由 Gradle 默认的资源过滤机制所决定的。如果我们像之前 Maven 中那样使用 `.`，Gradle 将期望在资源过滤时找到具有匹配 `jdk`、`os`、`revision` 和 `timestamp` 属性的 `build` 对象。然而这个对象并不存在，会导致构建失败。改变令牌分隔符可以避免这个问题，但也迫使我们修改属性文件的内容如下：

```java
build.jdk       = ${build_jdk}
build.os        = ${build_os}
build.revision  = ${build_revision}
build.timestamp = ${build_timestamp}
```

配置 JAR 清单是一个简单的操作，因为 `jar` 任务为此行为提供了一个入口点，如下片段所示，也可以追加到现有的 Gradle 构建文件中：

```java
jar {
  manifest {
    attributes(
      'Build-Jdk'      : project.buildJdk,
      'Build-OS'       : project.buildOs,
      'Build-Revision' : project.buildRevision,
      'Build-Timestamp': project.buildTimestamp
    )
  }
}
```

正如之前看到的那样，每个清单条目使用大写键和相应的捕获值。使用 `gradle jar` 运行构建应该会产生类似 Maven 提供的结果：属性文件将被复制到目标位置，在最终 JAR 中包含其值占位符的替换值，并且 JAR 清单也将被丰富为包含元数据。检查 JAR 文件显示它包含了预期的文件：

```java
$ gradle jar
$ jar tvf build/libs/example-1.0.0-SNAPSHOT.jar
     0 Sun Jan 10 21:08:22 CET 2021 META-INF/
    25 Sun Jan 10 21:08:22 CET 2021 META-INF/MANIFEST.MF
   165 Sun Jan 10 21:08:22 CET 2021 META-INF/metadata.properties
```

解压 JAR 并查看每个文件内部的结果如下：

```java
build.jdk       = 11.0.9 (Azul Systems, Inc. 11.0.9+11-LTS)
build.os        = Mac OS X x86_64 10.15.7
build.revision  = 0ab9d51a3aaa17fca374d28be1e3f144801daa3b
build.timestamp = 2021-01-10T21:08:22+01:00
```

```java
Manifest-Version: 1.0
Build-Jdk: 11.0.9 (Azul Systems, Inc. 11.0.9+11-LTS)
Build-OS: Mac OS X x86_64 10.15.7
Build-Revision: 0ab9d51a3aaa17fca374d28be1e3f144801daa3b
Build-Timestamp: 2021-01-10T21:08:22+01:00
```

完美！就是这样。让我鼓励你根据需要添加或删除键/值对，并配置其他插件（用于 Maven 和 Gradle）以公开附加元数据或提供其他处理和记录元数据到特定格式的方法。

# Maven 和 Gradle 的依赖管理基础

依赖管理自 Maven *1.x* 于 2002 年首次亮相以来，已成为 Java 项目的重要组成部分。该功能的核心是声明编译、测试和消费特定项目所需的构件，依赖于附加到构件的附加元数据，例如其组标识符、构件标识符、版本，有时还包括分类器。这些元数据通常以广为人知的文件格式公开：在 *pom.xml* 文件中表达的 [Apache Maven POM](https://oreil.ly/1Kzp6)。其他构建工具能够理解该格式，甚至可以生成和发布 *pom.xml* 文件，尽管使用完全不相关的格式声明构建方面，例如 Gradle 使用的 `build.gradle`（Groovy）或 `build.gradle.kts`（Kotlin）构建文件。

尽管自 Maven 早期以来便是提供的核心功能，并且也是 Gradle 的核心功能，但依赖管理和依赖解析仍然是许多人的绊脚石。尽管声明依赖项的规则并不复杂，但您可能会发现自己受到发布的具有无效、误导或缺失约束的元数据的影响。以下各小节是使用 Maven 和 Gradle 进行依赖管理的入门指南，但这绝不是详尽的解释——仅此话题就可以写一整本书。

换句话说，亲爱的读者，请小心，前方有龙。我将尽力指出最安全的路径。我们将从 Maven 开始，因为它是使用 *pom.xml* 文件格式定义构件元数据的构建工具。

## 使用 Apache Maven 进行依赖管理

你可能以前遇到过 POM 文件，毕竟它无处不在。具有模型版本 4.0.0 的 POM 文件负责定义产生和消费构件的方式。在 Maven 4 版本中，这两个功能已分开，尽管模型版本出于兼容性原因保持不变。预计当引入 Maven 5.0.0 版本时，模型格式将会发生变化，尽管目前写作时尚无详细信息。有一点可以确定：Maven 开发人员热衷于保持向后兼容性。让我们从基础知识开始。

依赖由三个必需元素标识：`groupId`、`artifactId` 和 `version`。这些元素统称为 *Maven 坐标* 或 *GAV 坐标*，其中 GAV 如你所料，代表 `groupId`、`artifactId`、`version`。偶尔你会发现定义了名为 `classifier` 的第四个元素的依赖项。

让我们逐一分解它们。`artifactId` 和 `version` 都很直接；前者定义了构件的“名称”，后者定义了版本号。同一个 `artifactId` 可能关联多个不同的版本。`groupId` 用于组合一组具有某种关系的构件，即它们都属于同一个项目或提供彼此相关的行为。`classifier` 添加了构件的另一个维度，尽管它是可选的。分类器通常用于区分特定设置下的构件，例如操作系统或 Java 版本。操作系统分类器的示例可以在 JavaFX 二进制文件中找到，例如 *javafx-controls-15-win.jar*、*javafx-controls-15-mac.jar* 和 *javafx-controls-15-linux.jar*，它们标识了可以与 Windows、macOS 和 Linux 平台一起使用的 JavaFX 控件二进制版本 15。

另一组常见的分类器是 `sources` 和 `javadoc`，它们标识包含源代码和生成文档（通过 Javadoc 工具）的 JAR 文件。GAV 坐标的组合必须是唯一的；否则，依赖解析机制将很难找到正确的依赖项使用。

POM 文件允许你在 `<dependencies>` 部分内定义依赖项，其中你会列出每个依赖项的 GAV 坐标。在其最简单的形式中，看起来像这样：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
 http://maven.apache.org/xsd/maven-4.0.0.xsd"
  xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.acme</groupId>
  <artifactId>example</artifactId>
  <version>1.0.0-SNAPSHOT</version>

  <dependencies>
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-collections4</artifactId>
      <version>4.4</version>
    </dependency>
  </dependencies>
</project>
```

以这种方式列出的依赖项称为 *直接依赖项*，因为它们在 POM 文件中是显式声明的。即使依赖项可能在标记为当前 POM 的父 POM 中声明，此分类也成立。什么是父 POM？它就像另一个 *pom.xml* 文件一样，只是你的 POM 通过使用 `<parent>` 部分将它标记为父/子关系。通过这种方式，父 POM 定义的配置可以被子 POM 继承。我们可以通过调用 `mvn dependency:tree` 命令来检查依赖图，该命令解析依赖图并将其打印出来：

```java
$ mvn dependency:tree
[INFO] Scanning for projects...
[INFO]
[INFO] --------------------------< com.acme:example >--------------------------
[INFO] Building example 1.0.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ example ---
[INFO] com.acme:example:jar:1.0.0-SNAPSHOT
[INFO] \- org.apache.commons:commons-collections4:jar:4.4:compile
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

这里我们可以看到当前 POM（通过其 GAV 坐标标识为 `com.acme:example:1.0.0-SNAPSHOT`）具有单个直接依赖项。在 `commons-collections4` 依赖项的输出中找到了两个额外的元素：第一个是 `jar`，它标识了构件的类型，第二个是 `compile`，它标识了依赖的范围。稍后我们会回到范围的问题，但可以简单地说，如果对于一个依赖项没有显式定义 `<scope>` 元素，那么它的默认范围就是 `compile`。现在，当消费包含直接依赖项的 POM 时，它会从消费 POM 的角度带来这些依赖项作为传递性。下一个示例展示了具体的设置：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
 http://maven.apache.org/xsd/maven-4.0.0.xsd"
  xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.acme</groupId>
  <artifactId>example</artifactId>
  <version>1.0.0-SNAPSHOT</version>

  <dependencies>
    <dependency>
      <groupId>commons-beanutils</groupId>
      <artifactId>commons-beanutils</artifactId>
      <version>1.9.4</version>
    </dependency>
  </dependencies>
</project>
```

使用与之前相同的命令解析和打印依赖图产生了以下结果：

```java
$ mvn dependency:tree
[INFO] Scanning for projects...
[INFO]
[INFO] --------------------------< com.acme:example >--------------------------
[INFO] Building example 1.0.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ example ---
[INFO] com.acme:example:jar:1.0.0-SNAPSHOT
[INFO] \- commons-beanutils:commons-beanutils:jar:1.9.4:compile
[INFO]    +- commons-logging:commons-logging:jar:1.2:compile
[INFO]    \- commons-collections:commons-collections:jar:3.2.2:compile
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

这告诉我们，`commons-beanutils` 组件有两个依赖关系设置在`com⁠pile`范围内，从`com.acme:example:1.0.0-SNAPSHOT`的角度来看，这两个传递依赖关系被视为传递的。这两个传递依赖关系似乎没有自己的直接依赖关系，因为它们的列表中没有任何内容。然而，如果你查看`commons-logging`的 POM 文件，你会发现以下依赖声明：

```java
<dependencies>
  <dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>logkit</groupId>
    <artifactId>logkit</artifactId>
    <version>1.0.1</version>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>avalon-framework</groupId>
    <artifactId>avalon-framework</artifactId>
    <version>4.1.5</version>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <version>2.3</version>
    <scope>provided</scope>
    <optional>true</optional>
  </dependency>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>3.8.1</version>
    <scope>test</scope>
  </dependency>
</dependencies>
```

正如你所见，实际上有五个依赖关系！然而，其中四个定义了一个额外的`<optional>`元素，而另外两个定义了`<scope>`的不同值。标记为`<optional>`的依赖关系可能对编译和测试生产者（本例中的`commons-logging`）是必需的，但并不一定适用于消费者；这是根据具体情况确定的。

现在我们再次看到它们时，是时候讨论范围了。*范围*确定依赖项是否包括在类路径中，并限制其传递性。Maven 定义了六个范围，如下所示：

`compile`

默认的范围，如果未指定范围，则使用之前看到的范围。这个范围中的依赖项将用于项目中的所有类路径（compile、runtime、test），并将传播到消费项目中。

`provided`

类似于`compile`，但不影响运行时类路径，也不是传递的。在这个范围内设置的依赖项预期由托管环境提供，例如作为 WAR 包打包并在应用服务器中启动的 Web 应用程序。

`runtime`

这个范围指示依赖项不是编译所必需的，而是执行所必需的。运行时和测试类路径都包括在这个范围内设置的依赖项，而编译类路径则被忽略。

`test`

定义了编译和运行测试所需的依赖项。这个范围不是传递的。

`system`

类似于`provided`，但必须列出具有显式路径（相对或绝对）的依赖项。因此，这个范围被视为不良实践，应该尽量避免使用。对于少数用例，它可能会有所帮助，但你必须承担后果。最好情况下，它是留给专家的选择——换句话说，可以想象这个范围根本不存在。

`import`

仅适用于`pom`类型的依赖项（如果未指定，则默认为`jar`）。只能用于在`<dependencyManagement>`部分内声明的依赖项。在这个范围内的依赖项将被其自身`<dependencyManagement>`部分中的依赖项列表所替代。

`<dependencyManagement>`部分有三个目的：为传递依赖项提供版本提示，提供一个可以使用`import`范围导入的依赖项列表，并在父子 POM 组合中使用一组默认值。让我们看看第一个目的。假设在你的 POM 文件中定义了以下依赖项：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
 http://maven.apache.org/xsd/maven-4.0.0.xsd"
  xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.acme</groupId>
  <artifactId>example</artifactId>
  <version>1.0.0-SNAPSHOT</version>

  <dependencies>
    <dependency>
      <groupId>com.google.inject</groupId>
      <artifactId>guice</artifactId>
      <version>4.2.2</version>
    </dependency>
    <dependency>
      <groupId>com.google.truth</groupId>
      <artifactId>truth</artifactId>
      <version>1.0</version>
    </dependency>
  </dependencies>
</project>
```

`guice` 和 `truth` 的 artifact 都将 `guava` 定义为直接依赖。这意味着从消费者的角度来看，`guava` 被视为一个传递依赖。如果我们解析并打印出依赖图，我们会得到以下结果：

```java
$ mvn dependency:tree
[INFO] Scanning for projects...
[INFO]
[INFO] --------------------------< com.acme:example >--------------------------
[INFO] Building example 1.0.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ example ---
[INFO] com.acme:example:jar:1.0.0-SNAPSHOT
[INFO] +- com.google.inject:guice:jar:4.2.2:compile
[INFO] |  +- javax.inject:javax.inject:jar:1:compile
[INFO] |  +- aopalliance:aopalliance:jar:1.0:compile
[INFO] |  \- com.google.guava:guava:jar:25.1-android:compile
[INFO] |     +- com.google.code.findbugs:jsr305:jar:3.0.2:compile
[INFO] |     +- com.google.j2objc:j2objc-annotations:jar:1.1:compile
[INFO] |     \- org.codehaus.mojo:animal-sniffer-annotations:jar:1.14:compile
[INFO] \- com.google.truth:truth:jar:1.0:compile
[INFO]    +- org.checkerframework:checker-compat-qual:jar:2.5.5:compile
[INFO]    +- junit:junit:jar:4.12:compile
[INFO]    |  \- org.hamcrest:hamcrest-core:jar:1.3:compile
[INFO]    +- com.googlecode.java-diff-utils:diffutils:jar:1.3.0:compile
[INFO]    +- com.google.auto.value:auto-value-annotations:jar:1.6.3:compile
[INFO]    \- com.google.errorprone:error_prone_annotations:jar:2.3.1:compile
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

`guava` 的解析版本结果是 `25.1-android`，因为这是在图中首先找到的版本。看看如果我们颠倒依赖项的顺序，然后再次解析图形会发生什么：

```java
$ mvn dependency:tree
[INFO] Scanning for projects...
[INFO]
[INFO] --------------------------< com.acme:example >--------------------------
[INFO] Building example 1.0.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ example ---
[INFO] com.acme:example:jar:1.0.0-SNAPSHOT
[INFO] +- com.google.truth:truth:jar:1.0:compile
[INFO] |  +- com.google.guava:guava:jar:27.0.1-android:compile
[INFO] |  |  +- com.google.guava:failureaccess:jar:1.0.1:compile
[INFO] |  |  +- com.google.guava:listenablefuture:jar:
                        9999.0-empty-to-avoid-conflict
[INFO] |  |  +- com.google.code.findbugs:jsr305:jar:3.0.2:compile
[INFO] |  |  +- com.google.j2objc:j2objc-annotations:jar:1.1:compile
[INFO] |  |  \- org.codehaus.mojo:animal-sniffer-annotations:jar:1.17:compile
[INFO] |  +- org.checkerframework:checker-compat-qual:jar:2.5.5:compile
[INFO] |  +- junit:junit:jar:4.12:compile
[INFO] |  |  \- org.hamcrest:hamcrest-core:jar:1.3:compile
[INFO] |  +- com.googlecode.java-diff-utils:diffutils:jar:1.3.0:compile
[INFO] |  +- com.google.auto.value:auto-value-annotations:jar:1.6.3:compile
[INFO] |  \- com.google.errorprone:error_prone_annotations:jar:2.3.1:compile
[INFO] \- com.google.inject:guice:jar:4.2.2:compile
[INFO]    +- javax.inject:javax.inject:jar:1:compile
[INFO]    \- aopalliance:aopalliance:jar:1.0:compile
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

现在，`guava` 的解析版本恰好是 `27.0.1-android`，因为它是在图中首先找到的版本。这种特定行为常常让人不知所措和沮丧。作为开发者，我们习惯于版本控制规范，尤其是 [语义化版本控制](https://semver.org)。

语义化版本控制告诉我们，版本号标记（由点分隔）根据其位置具有特定含义。第一个标记标识主要发布版本，第二个标记标识次要发布版本，第三个标记标识构建/补丁/修订版本。通常情况下，版本 `27.0.1` 被视为比 `25.1.0` 更近，因为主要编号 `27` 大于 `25`。在我们的情况下，图中有两个 `guava` 版本，`27.0.1-android` 和 `25.1-android`，并且两者在当前 POM 的同一级别——即传递图中仅下降一级处找到。

开发者通常会认为，因为我们知道语义化版本控制，并且可以清楚地确定哪个版本更新，所以 Maven 也能做到这一点，但现实并非如此！ Maven 从不查看版本号，而只看图中的位置。这就是为什么如果我们改变依赖项的顺序，会得到不同的结果。我们可以使用 `<dependencyManagement>` 部分来解决这个问题。

在 `<dependencyManagement>` 部分定义的依赖通常具有三个主要的 GAV 坐标。当 Maven 解析依赖关系时，它会查看此部分中的定义，以确定是否有匹配的 `groupId` 和 `artifactId`，如果匹配，则使用关联的 `version`。不管依赖项在图中的深度如何，或者它在图中出现了多少次，只要匹配，就会选择明确指定的版本。我们可以通过向消费者 POM 添加类似于以下内容的 `<dependencyManagement>` 部分来验证这一点：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
 http://maven.apache.org/xsd/maven-4.0.0.xsd"
  xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.acme</groupId>
  <artifactId>example</artifactId>
  <version>1.0.0-SNAPSHOT</version>

  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>com.google.guava</groupId>
        <artifactId>guava</artifactId>
        <version>29.0-jre</version>
      </dependency>
    </dependencies>
  </dependencyManagement>

  <dependencies>
    <dependency>
      <groupId>com.google.truth</groupId>
      <artifactId>truth</artifactId>
      <version>1.0</version>
    </dependency>
    <dependency>
      <groupId>com.google.inject</groupId>
      <artifactId>guice</artifactId>
      <version>4.2.2</version>
    </dependency>
  </dependencies>
</project>
```

我们可以看到 `guava` 的声明使用了 `com.google.guava:guava:29.0-jre` 坐标，这意味着如果一个传递依赖恰好匹配给定的 `groupId` 和 `artifactId`，将会使用版本 `29.0-jre`。我们知道这将在我们的消费者 POM 中发生两次。当解析并打印出依赖图时，我们得到以下结果：

```java
$ mvn dependency:tree
[INFO] Scanning for projects...
[INFO]
[INFO] --------------------------< com.acme:example >--------------------------
[INFO] Building example 1.0.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ example ---
[INFO] com.acme:example:jar:1.0.0-SNAPSHOT
[INFO] +- com.google.truth:truth:jar:1.0:compile
[INFO] |  +- com.google.guava:guava:jar:29.0-jre:compile
[INFO] |  |  +- com.google.guava:failureaccess:jar:1.0.1:compile
[INFO] |  |  +- com.google.guava:listenablefuture:jar:
                        9999.0-empty-to-avoid-conflict
[INFO] |  |  +- com.google.code.findbugs:jsr305:jar:3.0.2:compile
[INFO] |  |  +- org.checkerframework:checker-qual:jar:2.11.1:compile
[INFO] |  |  \- com.google.j2objc:j2objc-annotations:jar:1.3:compile
[INFO] |  +- org.checkerframework:checker-compat-qual:jar:2.5.5:compile
[INFO] |  +- junit:junit:jar:4.12:compile
[INFO] |  |  \- org.hamcrest:hamcrest-core:jar:1.3:compile
[INFO] |  +- com.googlecode.java-diff-utils:diffutils:jar:1.3.0:compile
[INFO] |  +- com.google.auto.value:auto-value-annotations:jar:1.6.3:compile
[INFO] |  \- com.google.errorprone:error_prone_annotations:jar:2.3.1:compile
[INFO] \- com.google.inject:guice:jar:4.2.2:compile
[INFO]    +- javax.inject:javax.inject:jar:1:compile
[INFO]    \- aopalliance:aopalliance:jar:1.0:compile
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

注意 `guava` 的选择版本确实是 `29.0-jre`，而不是本章早些时候看到的旧版本，这证实了 `<dependencyManagement>` 部分按预期执行其工作。

`<dependencyManagement>` 的第二个目的是列出可能被导入的依赖项，通过使用 `import` 范围和类型为 `pom` 的依赖项来实现。这些类型的依赖通常定义了它们自己的 `<dependencyManagement>` 部分，虽然这些 POM 可以增加更多的部分。定义了 `<dependencyManagement>` 部分但没有 `<dependencies>` 部分的 POM 依赖项称为材料清单（BOM）。通常，BOM 依赖项定义了一组为特定目的归属在一起的构件。尽管在 Maven 文档中没有明确定义，但可以找到两种类型的 BOM 依赖项：

库

所有声明的依赖关系都属于同一个项目，即使它们可能具有不同的组 ID，甚至可能有不同的版本。例如，可以看到 [helidon-bom](https://oreil.ly/bcMHI)，它将 Helidon 项目的所有构件分组在一起。

堆栈

依赖项按行为和它们带来的协同作用进行分组。依赖关系可能属于不同的项目。请参阅 [helidon-dependencies](https://oreil.ly/wgmVx) 的示例，它将之前的 `helidon-bom` 与其他依赖项（如 Netty、日志记录等）分组在一起。

让我们将 `helidon-dependencies` 视为依赖项的源头。检查此 POM，我们发现在其 `<dependencyManagement>` 部分声明了几十个依赖项，以下摘录中仅显示了其中的几个：

```java
<artifactId>helidon-dependencies</artifactId>
<packaging>pom</packaging>
<!-- additional elements elided -->
<dependencyManagement>
  <dependencies>
    <!-- more dependencies elided -->
    <dependency>
      <groupId>io.netty</groupId>
      <artifactId>netty-handler</artifactId>
      <version>4.1.51.Final</version>
    </dependency>
    <dependency>
      <groupId>io.netty</groupId>
      <artifactId>netty-handler-proxy</artifactId>
      <version>4.1.51.Final</version>
    </dependency>
    <dependency>
      <groupId>io.netty</groupId>
      <artifactId>netty-codec-http</artifactId>
      <version>4.1.51.Final</version>
    </dependency>
    <!-- more dependencies elided -->
  </dependencies>
</dependencyManagement>
```

在我们自己的 POM 中使用这个 BOM 依赖项需要再次使用 `<dependencyManagement>` 部分。我们还将为 `netty-handler` 定义一个显式依赖，就像我们在定义依赖时做过的那样，只是这次我们会省略 `<version>` 元素。POM 最终看起来像这样：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
 http://maven.apache.org/xsd/maven-4.0.0.xsd"
  xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.acme</groupId>
  <artifactId>example</artifactId>
  <version>1.0.0-SNAPSHOT</version>

  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>io.helidon</groupId>
        <artifactId>helidon-dependencies</artifactId>
        <version>2.2.0</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>

  <dependencies>
    <dependency>
      <groupId>io.netty</groupId>
      <artifactId>netty-handler</artifactId>
    </dependency>
  </dependencies>
</project>
```

注意 `helidon-dependencies` 依赖项是如何导入的。必须定义一个关键元素 `<type>`，它必须设置为 `pom`。还记得本章前面提到的吗？如果未指定值，默认情况下依赖关系的类型为 `jar`。在这里，我们知道 `helidon-dependencies` 是一个 BOM；因此它没有与之关联的 JAR 文件。如果我们省略类型元素，Maven 将发出警告并且无法解析 `netty-handler` 的版本，所以务必确保正确设置这个元素。解析依赖关系图会产生以下结果：

```java
$ mvn dependency:tree
[INFO] Scanning for projects...
[INFO]
[INFO] --------------------------< com.acme:example >--------------------------
[INFO] Building example 1.0.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ example ---
[INFO] com.acme:example:jar:1.0.0-SNAPSHOT
[INFO] \- io.netty:netty-handler:jar:4.1.51.Final:compile
[INFO]    +- io.netty:netty-common:jar:4.1.51.Final:compile
[INFO]    +- io.netty:netty-resolver:jar:4.1.51.Final:compile
[INFO]    +- io.netty:netty-buffer:jar:4.1.51.Final:compile
[INFO]    +- io.netty:netty-transport:jar:4.1.51.Final:compile
[INFO]    \- io.netty:netty-codec:jar:4.1.51.Final:compile
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

我们可以看到选择了正确的版本，并且 `netty-handler` 的每个直接依赖关系也被解析为传递依赖关系。

`<dependencyManagement>` 部分的第三个和最后一个目的在于当 POM 之间存在父子关系时发挥作用。POM 格式定义了一个 `<parent>` 部分，任何 POM 都可以使用它来与另一个被视为父级的 POM 建立关联。父 POM 提供的配置可以被子 POM 继承，其中父 `<dependencyManagement>` 部分就是其中之一。Maven 沿着父链接向上进行，直到无法找到父定义，然后沿着链条向下处理解析配置，位于较低级别的 POM 覆盖由较高级别 POM 设置的配置。

这意味着子 POM 总是有选择权来覆盖父 POM 声明的配置。因此，父 POM 中找到的 `<dependencyManagement>` 部分将对子 POM 可见，就像它是在子 POM 上定义的一样。我们仍然可以从此部分前两个目的中获得相同的好处，这意味着我们可以为传递依赖项固定版本并导入 BOM 依赖项。以下是一个父 POM 的示例，它在自己的 `<dependencyManagement>` 部分中声明了 `helidon-dependencies` 和 `commons-lang3`：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
 http://maven.apache.org/xsd/maven-4.0.0.xsd"
   xmlns="http://maven.apache.org/POM/4.0.0"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.acme</groupId>
  <artifactId>parent</artifactId>
  <version>1.0.0-SNAPSHOT</version>
  <packaging>pom</packaging>

  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>io.helidon</groupId>
        <artifactId>helidon-dependencies</artifactId>
        <version>2.2.0</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.11</version>
      </dependency>
    </dependencies>
  </dependencyManagement>
</project>
```

鉴于此 POM 文件未关联任何 JAR 文件，我们还必须显式定义 `<packaging>` 元素的值为 `pom`。子 POM 需要使用 `<parent>` 元素来引用此 POM，如下例所示：

```java
<?xml version="1.0" encoding="UTF-8"?>
<project
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
 http://maven.apache.org/xsd/maven-4.0.0.xsd"
  xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>com.acme</groupId>
    <artifactId>parent</artifactId>
    <version>1.0.0-SNAPSHOT</version>
  </parent>
  <artifactId>example</artifactId>

  <dependencies>
    <dependency>
      <groupId>io.netty</groupId>
      <artifactId>netty-handler</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-lang3</artifactId>
    </dependency>
  </dependencies>
</project>
```

完美！准备好这个设置后，现在是时候再次解析依赖图并检查其内容了：

```java
$ mvn dependency:tree
[INFO] Scanning for projects...
[INFO]
[INFO] --------------------------< com.acme:example >--------------------------
[INFO] Building example 1.0.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ example ---
[INFO] com.acme:example:jar:1.0.0-SNAPSHOT
[INFO] +- io.netty:netty-handler:jar:4.1.51.Final:compile
[INFO] |  +- io.netty:netty-common:jar:4.1.51.Final:compile
[INFO] |  +- io.netty:netty-resolver:jar:4.1.51.Final:compile
[INFO] |  +- io.netty:netty-buffer:jar:4.1.51.Final:compile
[INFO] |  +- io.netty:netty-transport:jar:4.1.51.Final:compile
[INFO] |  \- io.netty:netty-codec:jar:4.1.51.Final:compile
[INFO] \- org.apache.commons:commons-lang3:jar:3.11:compile
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

正如预期的那样，我们有两个直接依赖项，具有正确的 GAV 坐标，以及前面看到的传递依赖项。还有一些与依赖管理和解析相关的附加项，如依赖项排除（通过其 GA 坐标排除传递依赖项）和在依赖冲突中失败构建（在图中找到相同 GA 坐标的不同版本）。然而，最好在这里停下来，看看 Gradle 在依赖管理方面提供了什么。

## 使用 Gradle 进行依赖管理

正如前面提到的，Gradle 建立在从 Maven 学到的教训之上，并理解 POM 格式，使其能够提供类似于 Maven 的依赖解析能力。Gradle 还提供了额外的能力和更精细的控制。本节涉及已经讨论过的主题，因此建议您首先阅读前面的部分，以防您跳过了它，或者如果您需要回顾 Maven 提供的依赖管理的内容。让我们看看 Gradle 提供了什么。

首先，必须选择用于编写构建文件的 DSL。您的选项是 Apache Groovy DSL 或 Kotlin DSL。我们将继续使用前者，因为在实际应用中有更多的示例。从 Groovy 转换到 Kotlin 比反向转换更容易，这意味着可以直接使用 Groovy 编写的代码片段（可能需要 IDE 建议的一些更改），而反向转换需要掌握两种 DSL。下一步是选择依赖项记录的格式，有多种常见格式，如单个 GAV 坐标文字，例如：

```java
'org.apache.commons:commons-collections4:4.4'
```

和将每个 GAV 坐标成员分割为自己元素的 Map 文字，例如：

```java
group: 'org.apache.commons', name: 'commons-collections4', version: '4.4'
```

注意，Gradle 选择使用`group`而不是`groupId`，以及`name`而不是`artifactId`，尽管语义相同。

接下来的任务是声明特定范围（在 Maven 术语中）的依赖项，虽然 Gradle 称其为*配置*，其行为超出了范围的能力。假设将`java-library`插件应用于 Gradle 构建文件，则默认情况下可以访问以下配置：

`api`

定义了编译生产代码所需的依赖项，并影响编译类路径。它等同于`compile`范围，因此在生成 POM 时被映射为这样。

`implementation`

定义了编译所需但被视为实现细节的依赖项；它们比`api`配置中的依赖项更灵活。此配置影响编译类路径，但在生成 POM 时将映射到`runtime`范围。

`compileOnly`

定义了编译所需但不执行的依赖项。此配置影响编译类路径，但这些依赖项不与其他类路径共享。此外，在生成的 POM 中它们也不被映射。

`runtimeOnly`

此配置中的依赖项仅用于执行，并仅影响运行时类路径。在生成 POM 时，它们映射到`runtime`范围。

`testImplementation`

定义了编译测试代码所需的依赖项，并影响`testCompile`类路径。在生成 POM 时，它们映射到`test`范围。

`testCompileOnly`

定义了编译测试代码但不执行的依赖项。此配置影响`testCompile`类路径，但这些依赖项不与`testRuntime`类路径共享。此外，在生成的 POM 中它们也不被映射。

`testRuntimeOnly`

具有此配置的依赖项用于执行测试代码，仅影响`testRuntime`类路径。在生成 POM 时，它们映射到`test`范围。

可能会根据使用的 Gradle 版本看到其他配置，包括以下已弃用的（在 Gradle 6 中已弃用，在 Gradle 7 中移除）：

`compile`

此配置已拆分为`api`和`implementation`。

`runtime`

Deprecated in favor of `runtimeOnly`.

`testCompile`

Deprecated in favor of `testImplementation` to align with the `implementation` configuration name.

`testRuntime`

Deprecated in favor of `testRuntimeOnly` to be consistent with `runtimeOnly`.

Similarly to Maven, the classpaths follow a hierarchy. The compile classpath can be consumed by the runtime classpath, thus every dependency set in either the `api` or `implementation` configurations is also available for execution. This classpath can also be consumed by the test compile classpath, enabling production code to be seen by test code. The runtime and test classpaths are consumed by the test runtime classpath, allowing test execution access to all dependencies defined in all the configurations so far mentioned.

As with Maven, dependencies can be resolved from repositories. Unlike Maven, in which both Maven local and Maven Central repositories are always available, in Gradle we must explicitly define the repositories from which dependencies may be consumed. Gradle lets you define repositories that follow the standard Maven layout, the Ivy layout, and even local directories with a flat layout. It also provides conventional options to configure the most commonly known repository, Maven Central. We’ll use `mavenCentral` for now as our only repository. Putting together everything we have seen so far, we can produce a build file like the following:

```java
plugins {
  id 'java-library'
}

repositories {
  mavenCentral()
}

dependencies {
  api 'org.apache.commons:commons-collections4:4.4'
}
```

We can print the resolved dependency graph by invoking the `dependencies` task. However, this will print the graph for every single configuration, so we’ll print only the resolved dependency graph for the compile classpath for the sake of keeping the output short, as well as to showcase an extra setting that can be defined for this task:

```java
$ gradle dependencies --configuration compileClasspath

> Task :dependencies

------------------------------------------------------------
Root project
------------------------------------------------------------

compileClasspath - Compile classpath for source set 'main'.
\--- org.apache.commons:commons-collections4:4.4
```

As you can see, only a single dependency is printed out, because `commons-collections` does not have any direct dependencies of its own that are visible to consumers.

Let’s see what happens when we configure another dependency that brings in additional transitive dependencies, but this time using the `implementation` configuration that will show that both `api` and `implementation` contribute to the compile classpath. The updated build file looks like this:

```java
plugins {
  id 'java-library'
}

repositories {
  mavenCentral()
}

dependencies {
  api 'org.apache.commons:commons-collections4:4.4'
  implementation 'commons-beanutils:commons-beanutils:1.9.4'
}
```

Running the `dependencies` task with the same configuration as before now yields the following result:

```java
$ gradle dependencies --configuration compileClasspath

> Task :dependencies

------------------------------------------------------------
Root project
------------------------------------------------------------

compileClasspath - Compile classpath for source set 'main'.
+--- org.apache.commons:commons-collections4:4.4
\--- commons-beanutils:commons-beanutils:1.9.4
     +--- commons-logging:commons-logging:1.2
     \--- commons-collections:commons-collections:3.2.2
```

This tells us our consumer project has two direct dependencies contributing to the compile classpath, and that one of those dependencies brings two additional dependencies seen as transitive from our consumer’s point of view. If for some reason you’d like to skip bringing those transitive dependencies into your dependency graph, you can add an extra block of configuration on the direct dependency that declares them, like this:

```java
plugins {
  id 'java-library'
}

repositories {
  mavenCentral()
}

dependencies {
  api 'org.apache.commons:commons-collections4:4.4'
  implementation('commons-beanutils:commons-beanutils:1.9.4') {
    transitive = false
  }
}
```

Running the `dependencies` task once more now shows only the direct dependencies and no transitive dependencies:

```java
$ gradle dependencies --configuration compileClasspath

> Task :dependencies

------------------------------------------------------------
Root project
------------------------------------------------------------

compileClasspath - Compile classpath for source set 'main'.
+--- org.apache.commons:commons-collections4:4.4
\--- commons-beanutils:commons-beanutils:1.9.4
```

在继续之前，我想要讨论的最后一个方面是，与 Maven 不同，Gradle 理解语义化版本，并将在依赖解析过程中相应地选择最高版本号。我们可以通过配置两个版本的相同依赖来验证这一点，无论它们是直接的还是传递的，如下面的片段所示：

```java
plugins {
  id 'java-library'
}

repositories {
  mavenCentral()
}

dependencies {
  api 'org.apache.commons:commons-collections4:4.4'
  implementation 'commons-collections:commons-collections:3.2.1'
  implementation 'commons-beanutils:commons-beanutils:1.9.4'
}
```

在这种情况下，我们声明了对`commons-collections`版本`3.2.1`的直接依赖。我们从之前的运行中知道，`commons-beanutils:1.9.4`引入了`commons-collections`的`3.2.2`版本。鉴于`3.2.2`被认为比`3.2.1`更新，我们期望`3.2.2`将被解析。调用`dependencies`任务将产生以下结果：

```java
$ gradle dependencies --configuration compileClasspath

> Task :dependencies

------------------------------------------------------------
Root project
------------------------------------------------------------

compileClasspath - Compile classpath for source set 'main'.
+--- org.apache.commons:commons-collections4:4.4
+--- commons-collections:commons-collections:3.2.1 -> 3.2.2
\--- commons-beanutils:commons-beanutils:1.9.4
     +--- commons-logging:commons-logging:1.2
     \--- commons-collections:commons-collections:3.2.2

(*) - dependencies omitted (listed previously)
```

如预期的那样，选择了版本 3.2.2。输出甚至包含一个指示器，告诉我们何时将依赖版本设置为与请求的版本不同的值。版本也可以配置为固定的，而不考虑它们的语义化版本方案，甚至可以设置为较低的值。这是因为 Gradle 提供了更灵活的依赖解析策略选项。然而，这属于高级主题的范畴，与依赖锁定、严格与建议的版本、依赖重定位、平台和强制平台（Gradle 与 BOM 工件交互的方式）以及其他内容相关。

# 容器的依赖管理基础

在软件开发周期的进一步过程中，您可能会遇到一个步骤，需要将您的 Maven 和 Gradle 项目打包成容器镜像。就像项目中的其他依赖项一样，您的容器镜像也必须适当地管理，并与其他所需的工件协调一致。容器在第三章中有详细讨论，但本节主要关注容器镜像管理的一些微妙之处。与自动化构建工具 Maven 和 Gradle 中的依赖管理一样，可能还会遇到更多问题。

正如您在第三章中学到的那样，容器是使用容器镜像启动的，这些镜像通常使用 Dockerfile 定义。Dockerfile 的作用是定义镜像的每一层，这些层将用于构建运行的容器。从这个定义中，您将获得基础分发层、代码库和框架，以及运行软件所需的任何其他文件或工件。在这里，您还将定义任何必要的配置（例如开放端口、数据库凭据和消息服务器的引用），以及任何需要的用户和权限。

在本节中首先讨论的是 Dockerfile 的第 1 行，或者在多阶段构建的 Dockerfile 中，以 `FROM` 指令开头的行。与 Maven POM 类似，一个镜像可以从一个`父`镜像构建，该父镜像可能来自另一个父镜像，一直到`基础`镜像，即最初的祖先。在这里，我们必须特别注意我们的镜像组合方式。

正如您可能还记得的来自 第三章 的内容，Docker 镜像的版本控制旨在在软件开发阶段提供灵活性，并在需要时使用最新的维护更新来增强信心。大多数情况下，这是通过引用特殊的镜像版本 `latest`（如果未指定版本，则为默认版本）来实现的，请求该版本将检索到被假定为正在开发中的镜像的最新版本。虽然不是完美的比较，但这最像使用 Java 依赖的 `快照` 版本。

在开发过程中一切正常，但是当在生产环境中出现新 bug 需要进行故障排除时，生产镜像工件中的这种版本控制可能会增加故障排除的难度。一旦一个镜像已经使用默认的 `latest` 版本构建完成，重现构建可能会很困难甚至不可能。就像您希望在生产发布中避免使用快照依赖一样，我建议锁定您的镜像版本并避免使用默认的 `latest` 版本，以限制不必要的变动部分。

仅仅锁定您的镜像版本并不足以确保安全。构建容器时，请只使用可信任的基础镜像。这个建议看起来可能很明显，但第三方注册表通常没有对其存储的镜像制定治理政策。了解哪些镜像可以在 Docker 主机上使用，了解它们的来源并审查其内容非常重要。您还应该为镜像验证启用 Docker 内容信任（DCT），并且只安装经过验证的软件包到镜像中。

使用尽可能少的基础镜像，不包含可能导致攻击面变大的不必要软件包。在您的容器中减少组件数量可以减少可用的攻击向量，而且最小化的镜像还能提升性能，因为磁盘上的字节数更少，拷贝镜像时网络流量也较少。BusyBox 和 Alpine 是构建最小基础镜像的两个选择。在您验证的基础镜像上构建任何额外的层时，一定要特别注意明确指定所有软件包的版本或者您拉取到镜像中的任何其他工件。

# 发布工件

到目前为止，我已经讨论了如何解析构件和依赖项，通常从称为仓库的位置获取，但仓库究竟是什么，以及如何向其发布构件？从最基本的角度来看，*构件仓库*是文件存储，用于跟踪构件。仓库收集每个发布构件的元数据，并利用该元数据提供额外功能，如搜索、归档、访问控制列表（ACL）等。工具可以利用这些元数据提供其他高级功能，如漏洞扫描、指标、分类等。

我们可以使用两种类型的仓库来管理 Maven 依赖项，这些依赖项可以通过 GAV 坐标解析：本地和远程。Maven 使用本地文件系统中的可配置目录来跟踪已解析的依赖项。这些依赖项可以从远程仓库下载，也可以直接由 Maven 工具放置在那里。这个目录通常被称为*Maven 本地*，其默认位置为当前用户的主目录下的*.m2/repository*。这个位置是可配置的。另一端是远程仓库，由仓库软件如 Sonatype Nexus Repository、JFrog Artifactory 等处理。最著名的远程仓库是 Maven 中央仓库，用于解析构件的标准仓库。

现在让我们讨论如何将构件发布到本地和远程仓库。

## 发布到 Maven 本地

Maven 提供了三种方式将构件发布到 Maven 本地仓库。其中两种是显式的，一种是隐式的。我们已经涵盖了隐式的方式——每当 Maven 从远程仓库解析依赖项时都会发生这种情况，结果是构件的副本及其元数据（相关的*pom.xml*）将被放置在 Maven 本地仓库中。这种行为默认发生，因为 Maven 使用 Maven 本地作为缓存，以避免再次通过网络请求构件。

发布构件到 Maven 本地的另外两种方式是显式地“安装”文件到仓库中。Maven 有一组生命周期阶段，其中*install*是其中之一。这个阶段对于 Java 开发者来说非常熟悉，因为它用于（有时滥用于）编译、测试、打包和将构件安装到 Maven 本地。Maven 生命周期阶段遵循预定的顺序：

```java
Available lifecycle phases are: validate, initialize, generate-sources,
process-sources, generate-resources, process-resources, compile,
process-classes, generate-test-sources, process-test-sources,
generate-test-resources, process-test-resources, test-compile,
process-test-classes, test, prepare-package, package, pre-integration-test,
integration-test, post-integration-test, verify, install, deploy,
pre-clean, clean, post-clean, pre-site, site, post-site, site-deploy.
```

阶段按顺序执行直至找到终端阶段。因此，通常调用*install*将导致几乎完整的构建（除了*deploy*和*site*）。我提到*install*被滥用了，因为大多数情况下只需调用*verify*即可，后者位于*install*之前的阶段，它会强制编译、测试、打包和集成测试，但如果不需要，则不会将构件污染到 Maven 本地。这绝不是建议始终放弃*install*而选择*verify*，因为有时测试需要从 Maven 本地解析构件。关键是要意识到每个阶段的输入/输出及其后果。

回到安装。将构件安装到 Maven 本地的第一种方法是简单地调用*install*阶段，就像这样：

```java
$ mvn install
```

这将会复制所有的*pom.xml*文件并按照约定重命名为`*artifactId*`-`*version*.pom`，同时将每个附加的构件放入 Maven 本地。附加的构件通常是构建生成的二进制 JAR 文件，也可以包括其他 JAR 文件，比如`-sources`和`-javadoc` JAR 文件。第二种安装构件的方法是手动调用`install:install-file`目标并提供一组参数。假设你有一个 JAR 文件（*artifact.jar*）和一个匹配的 POM 文件（*artifact.pom*），安装它们可以这样做：

```java
$ mvn install:install-file -Dfile=artifact.jar -DpomFile=artifact.pom
```

Maven 将读取 POM 文件中的元数据，并根据解析的 GAV 坐标将文件放置在对应的位置。可以覆盖 GAV 坐标，动态生成 POM，甚至可以省略显式的 POM 文件（如果 JAR 文件内部包含一份副本的话，这通常是 Maven 构建的情况；而 Gradle 默认情况下不包含 POM 文件）。

Gradle 有一种方式可以将构件发布到 Maven 本地，那就是应用`maven-publish`插件。该插件为项目添加了新的功能，比如`publishToMavenLocal`任务；正如其名称所示，它将构建的构件和生成的 POM 文件复制到 Maven 本地。与 Maven 不同，Gradle 不将 Maven 本地作为缓存，因为它有自己的缓存基础设施。因此，当 Gradle 解析依赖关系时，文件会被放置在另一个位置，通常位于当前用户的主目录下的`.gradle/caches/modules-2/files-2.1`。

这就涵盖了发布到 Maven 本地的内容。现在让我们看一下远程仓库。

## 发布到 Maven 中央仓库

*Maven Central* 仓库是支撑 Java 项目日常构建的支柱。运行 Maven Central 的软件是 *Sonatype Nexus Repository*，这是由 Sonatype 提供的一个存储库。由于在 Java 生态系统中的重要角色，Maven Central 已经制定了一系列在发布构件时必须遵循的规则；Sonatype 已经发布了一份[指南](https://oreil.ly/xfhNd)，详细解释了先决条件和规则。我强烈建议在阅读本书之后阅读该指南，以确保要求是否已自出版之日起更新。简而言之，您必须确保以下几点：

+   您必须证明拥有目标 `groupId` 的反向域的所有权。如果您的 `groupId` 是 `com.acme.*`，则您必须拥有 `acme.com`。

+   发布二进制 JAR 时，您还必须提供 `-sources` 和 `javadoc` JAR 文件，以及匹配的 POM 文件，即至少四个单独的文件。

+   发布 POM 类型的构件时，只需要提供 POM 文件。

+   所有构件的 PGP 签名文件也必须提交。用于签名的 PGP 密钥必须发布在公钥服务器上，以便 Maven Central 验证签名。

+   可能在最初会让大多数人困惑的一点是：POM 文件必须符合最少的一组元素，如 `<license>`、`<developers>`、`<scm>` 等。这些元素在指南中有详细描述；如果省略任何一个元素，将导致在发布过程中失败，结果是构件将完全不会发布。

我们可以通过使用 [PomChecker](https://oreil.ly/E7LP1) 项目来避免最后一个问题，或者至少在开发过程中更早地检测到它。PomChecker 可以以多种方式调用：作为独立的 CLI 工具、作为 Maven 插件或作为 Gradle 插件。这种灵活性使其非常适合在本地环境或 CI/CD 流水线中验证 POM。使用 CLI 验证 *pom.xml* 文件可以这样做：

```java
$ pomchecker check-maven-central --pom-file=pom.xml
```

如果您的项目使用 Maven 构建，可以在不需要在 POM 中配置的情况下调用 PomChecker 插件，像这样：

```java
$ mvn org.kordamp.maven:pomchecker-maven-plugin:check-maven-central
```

那个命令将解析 `pomchecker-maven-plugin` 的最新版本，并立即在现有项目上执行其 `check-maven-central` 目标。使用 Gradle 时，您必须显式配置 `org.kordamp.gradle.pomchecker` 插件，因为与 Maven 不同，Gradle 不提供调用内联插件的选项。

必须应用于构建的最后一部分配置是发布机制本身。如果您使用 Maven 构建，可以通过将以下内容添加到 *pom.xml* 中来完成：

```java
<distributionManagement>
  <repository>
    <id>ossrh</id>
    <url>https://s01.oss.sonatype.org/service/local/staging/deploy/maven2/</url>
  </repository>
  <snapshotRepository>
    <id>ossrh</id>
    <url>https://s01.oss.sonatype.org/content/repositories/snapshots</url>
  </snapshotRepository>
</distributionManagement>

<build>
  <plugins>
    <plugin>
     <groupId>org.apache.maven.plugins</groupId>
     <artifactId>maven-javadoc-plugin</artifactId>
     <version>3.2.0</version>
      cutions>
       cution>
       <id>attach-javadocs</id>
       <goals>
         <goal>jar</goal>
       </goals>
       <configuration>
         <attach>true</attach>
       </configuration>
      </execution>
     </executions>
    </plugin>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-source-plugin</artifactId>
      <version>3.2.1</version>
      <executions>
        <execution>
          <id>attach-sources</id>
          <goals>
            <goal>jar</goal>
          </goals>
          <configuration>
            <attach>true</attach>
          </configuration>
        </execution>
      </executions>
    </plugin>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-gpg-plugin</artifactId>
      <version>1.6</version>
      <executions>
        <execution>
          <goals>
            <goal>sign</goal>
          </goals>
          <phase>verify</phase>
          <configuration>
            <gpgArguments>
              <arg>--pinentry-mode</arg>
              <arg>loopback</arg>
            </gpgArguments>
          </configuration>
        </execution>
      </executions>
    </plugin>
    <plugin>
      <groupId>org.sonatype.plugins</groupId>
      <artifactId>nexus-staging-maven-plugin</artifactId>
      <version>1.6.8</version>
      <extensions>true</extensions>
      <configuration>
        <serverId>central</serverId>
        <nexusUrl>https://s01.oss.sonatype.org</nexusUrl>
        <autoReleaseAfterClose>true</autoReleaseAfterClose>
      </configuration>
    </plugin>
  </plugins>
</build>
```

注意，该配置生成 `-sources` 和 `-javadoc` JAR 文件，用 PGP 签署所有附加的构件，并将所有构件上传到给定的 URL，这恰好是 Maven Central 支持的 URL 之一。`<serverId>` 元素标识了您必须在 *settings.xml* 文件中设置的凭据（否则上传将失败），或者您可以将凭据定义为命令行参数。

您可能希望将插件配置放在 `<profile>` 部分中，因为配置的插件提供的行为仅在发布时需要；在执行主要生命周期阶段序列期间生成额外的 JAR 没有必要。这样，您的构建将只执行最小的步骤，从而更快地执行。

另一方面，如果您使用 Gradle 进行发布，您必须配置一个能够发布到 Sonatype Nexus 仓库的插件，最新的此类插件是 [io.github.gradle-nexus.publish-plugin](https://oreil.ly/MdCNh)。有多种配置 Gradle 完成工作的方式。惯用法变化比您必须在 Maven 中配置的更快。我建议您查阅官方 Gradle 指南，了解在这种情况下需要做什么。

## 发布到 Sonatype Nexus 仓库

您可能还记得 Maven Central 是使用 Sonatype Nexus 仓库运行的，因此不应对前面部分中显示的配置感到意外，因此您只需更改发布 URL 以匹配 Nexus 仓库。但是有一个警告：Maven Central 经常施加的严格验证规则通常不适用于自定义 Nexus 安装。也就是说，Nexus 有选项配置管理工件发布的规则。例如，这些规则在您组织内运行的 Nexus 实例中可能会放宽，或者在其他领域可能会更严格。最好查阅您组织的文档，了解有关向其自己的 Nexus 实例发布工件的信息。

有一件事很明确：如果您将工件发布到您组织的 Nexus 仓库，而这些工件最终必须发布到 Maven Central，从一开始遵循 Maven Central 的规则是个好主意——只要这些规则不与您组织的规则冲突。

## 发布到 JFrog Artifactory

*JFrog Artifactory* 是另一个流行的工件管理选项。它提供与 Sonatype Nexus 仓库类似的功能，同时添加了其他功能，包括与 JFrog 平台的其他产品集成，例如 Xray 和 Pipelines。我特别喜欢的一个功能是，发布之前不需要在源处对工件进行签名。Artifactory 可以使用您的 PGP 密钥或站点范围的 PGP 密钥执行签名。这使您无需在本地和 CI 环境上设置密钥，也减少了发布过程中的字节数传输。与之前一样，我们在 Maven Central 中看到的发布配置也适用于 Artifactory，只需更改发布 URL 以匹配 Artifactory 实例。

与 Nexus 类似，Artifactory 允许您将工件同步到 Maven Central，并且您必须再次遵循发布到 Maven Central 的规则。因此，从一开始就发布格式良好的 POM、源代码和 Javadoc JAR 是个好主意。

# 总结

在本章中，我们涵盖了许多概念，但主要的要点是，单靠工件本身是不足以在构建软件或领先竞争中取得最佳结果的。工件通常具有与之关联的元数据，例如它们的构建时间、依赖版本和环境。这些元数据可以用于追溯特定工件的起源，帮助将其转变为可重现的工件，或者启用生成软件材料清单（SBOM），这又是另一种元数据格式。此外，通过这些元数据，还可以极大地增强构建管道健康和稳定性方面的可观察性、监控以及其他关注点。

关于依赖性，我们看到了与流行的 Java 构建工具（如 Apache Maven 和 Gradle）一起进行依赖性解析的基础知识。当然，这一章节讨论的深度还远不够；这些主题确实可以单独填满一本书。请务必关注这一领域的改进，这些改进可能会由后续版本的这些构建工具提供。

最后，我们讨论了如何将 Java 工件发布到流行的 Maven 中央仓库，这要求遵循一组特定的准则以确保成功发布。Maven 中央仓库是官方仓库，但并非唯一选择。Sonatype 提供 Sonatype Nexus 仓库，而 JFrog 提供 JFrog Artifactory，这两者也是管理内部位置（例如您自己的组织或公司）的工件的流行选择。
