# 第三章\. 编程 AWS Lambda 函数

本章内容涉及构建 Lambda 函数的含义—它们是什么样子的，如何配置它们的运行方式，以及如何指定自己的环境配置。通过检查 Lambda 执行环境的核心概念、输入和输出、超时、内存和 CPU，最后，Lambda 如何使用环境变量进行应用程序配置来学习这些主题。

首先，让我们看看 Lambda 函数是如何执行的。系好您的登山靴—是时候探索一番了。

# 核心概念：运行时模型、调用

在第二章中，您创建了一个 Java 类，将其上传到某个位于“云”中的 Lambda 服务，并神奇地能够执行该代码。您不必考虑操作系统、容器、启动脚本、代码部署到实际主机或 JVM 设置。也不必考虑那些讨厌的“服务器”。那么您的代码是如何执行的呢？

要理解这一点，您首先需要了解 Lambda 执行环境的基础知识，如图 3-1 所示。

![images/ch03_image01.png](img/awsl_0301.png)

###### 图 3-1\. Lambda 执行环境

## Lambda 执行环境

正如我们在第二章中提到的（参见“安装 AWS CLI”），AWS 的管理和函数操作（通常称为*控制平面*和*数据平面*）都广泛使用 API。Lambda 也不例外，为函数的管理和执行提供 API。

每当调用 AWS Lambda API 的`invoke`命令时，函数都会被执行或*调用*。这发生在以下时间：

+   当函数由事件源触发时

+   当您在 Web 控制台中使用测试工具包时

+   当您自己调用 Lambda API 的`invoke`命令时，通常通过 CLI 或 SDK，从您自己的代码或脚本中。

首次调用函数将启动以下一系列活动链，最终导致您的代码被执行。

首先，Lambda 服务将创建一个主机 Linux 环境—一个轻量级微虚拟机。通常您不需要担心它究竟是何种环境（什么内核，什么发行版等），但如果您关心，亚马逊会公布这些信息。但不要依赖它保持不变—亚马逊可能会频繁更改 Lambda 函数的操作系统，通常是为了您自己的利益，包括自动安全补丁。

一旦主机环境被创建，Lambda 将在其中启动语言运行时—在我们的例子中是 Java 虚拟机。在撰写本文时，JVM 版本将始终为 Java 8 或 Java 11。您必须提供与您选择的 Java 版本兼容的代码给 Lambda。JVM 是以一组环境标志启动的，我们无法更改。

当我们编写代码时，您可能已经注意到没有“main”方法——顶级 Java 应用程序是亚马逊自己的 Java 应用程序服务器，我们将其称为 *Lambda Java 运行时*；这是下一个要启动的组件。运行时负责顶级错误处理、日志记录等。

当然，Lambda Java 运行时的主要任务是执行我们的代码。调用链的最后几步是：（a）加载我们的 Java 类，并（b）调用我们在部署过程中指定的处理方法。

## 调用类型

很好——我们的代码已经运行！接下来会发生什么？

为了探索这个问题，让我们开始使用 AWS CLI。在 第二章 中，我们使用了更高级别的 SAM CLI 工具——AWS CLI 更接近 AWS 机器的内部。具体来说，我们将使用 AWS CLI 中用于调用 Lambda 函数的命令：`aws lambda invoke`。

假设您在 第二章 中运行了示例，让我们从一个小更新开始。打开 *template.yaml* 文件（我们将从现在开始偶尔称为 *SAM 模板*），在属性部分中添加一个名为 `FunctionName` 的新属性，值为 `HelloWorldJava`，以使资源部分如下所示：

```java
HelloWorldLambda:
  Type: AWS::Serverless::Function
  Properties:
    FunctionName: HelloWorldJava
    Runtime: java8
    MemorySize: 512
    Handler: book.HelloWorld::handler
    CodeUri: target/lambda.jar
```

重新从 第二章 运行 **`sam deploy`** 命令。几分钟后应该完成。如果回到 Lambda 控制台，你会看到你的奇怪命名的 Java 函数现在已经重命名为 `HelloWorldJava`。在大多数实际用例中，我们喜欢使用 AWS 提供的生成名称，但当我们学习 Lambda 时，能够引用更简洁名称的函数会更好。

###### 注意

要使用 Java 11 运行时而不是 Java 8，只需在 SAM 模板中将 `Runtime:` 属性从 `java8` 更改为 `java11`。

让我们回到调用。从终端运行以下命令：

```java
$ aws lambda invoke \
  --invocation-type RequestResponse \
  --function-name HelloWorldJava \
  --payload \"world\" outputfile.txt
```

这应该返回以下内容：

```java
{
  "StatusCode": 200,
  "ExecutedVersion": "$LATEST"
}
```

您可以通过 `StatusCode` 是 `200` 来确认一切正常。

通过执行以下命令，还可以查看 Lambda 函数返回的内容：

```java
$ cat outputfile.txt && echo
"Hello, world"
```

当我们执行 `invoke` 命令时，Lambda 函数首先被实例化，正如我们在前一节中描述的那样。实例化完成后，Lambda Java 运行时在 JVM 内部调用我们的 Lambda 函数，使用我们传递给 `payload` 参数的数据——在本例中是字符串 `"world"`。

我们的代码然后运行。作为提醒，这里是代码：

```java
public String handler(String s) {
  return "Hello, " + s;
}
```

它接受我们的输入（`"world"`），并返回 `"Hello, world"`。

这里有一个重要但微妙的点。当我们调用 `invoke` 时，我们指定了 `--invocation-type RequestResponse`——这意味着我们 *同步* 调用函数（即 Lambda 运行时调用我们的代码并等待结果）。我们在 “Lambda 应用程序是什么样子？” 中解释了这一点。*同步行为* 对于像 Web API 这样的场景非常有用。

因为我们同步调用了函数，Lambda 运行时能够将响应返回给我们的终端，这就是保存到 *outputfile.txt* 的内容。

现在让我们稍微不同地调用函数：

```java
$ aws lambda invoke \
  --invocation-type Event \
  --function-name HelloWorldJava \
  --payload \"world\" outputfile.txt
```

注意我们已将 `--invocation-type` 标志更改为 `Event`。现在的结果如下所示：

```java
{
  "StatusCode": 202
}
```

`StatusCode` 是 `202`，而不是 `200`。在 HTTP 术语中，`202` 意味着*已接受*。如果您查看 *outputfile.txt*，您会发现它是空的。

这一次我们以*异步*方式调用函数。Lambda 运行时像以前一样调用我们的代码，但它不等待或使用我们代码返回的值——该值会被丢弃。使用异步执行的关键在于我们可以对某些其他函数或服务执行“副作用”。在 “Lambda 应用程序是什么样子？” 中的异步示例中，副作用是将照片的新调整大小版本上传到亚马逊的 S3 服务中。

当您开始使用 Lambda 时，您会发现大多数 Lambda 函数类使用异步调用，支持 Lambda 是一个*事件驱动平台*的理念。我们将在本书稍后的章节中进一步探讨这一点，当我们开始研究 “Lambda 事件来源” 时。

我们在前两个示例中使用了相同的代码；然而，如果您知道您的 Lambda 函数永远不会被同步使用，则不需要返回值——该方法可以具有 `void` 返回类型。让我们看一个例子。

首先，将函数的方法更改为以下内容：

```java
public void handler(String s) {
  System.out.println("Hello, " + s);
}
```

注意我们已将返回类型更改为 `void`，并且现在正在向 `System.out` 写入消息。

现在我们需要重建和重新部署我们的代码。要做到这一点，请运行与 第二章 中相同的两个命令：

+   **`mvn package`**

+   **`sam deploy…`**

其中 `**…**` 指的是您之前使用的相同参数。您会经常运行这些命令，所以可能想把它们放入一个脚本中。

现在以 `Event` 调用类型再次调用代码，您应该会收到另一个 `"StatusCode": 202` 的响应。但是 `System.out` 中的消息去哪里了？要理解这一点，让我们快速看一下日志记录。

###### 注意

您现在已经了解足够的关于 `mvn`、`sam` 和 `aws` 命令的知识，可以运行本章剩余的示例。如果出现异常情况，请转到 AWS Web 控制台中的 *CloudFormation*，删除 `HelloWorldLambdaJava` 栈，然后重新部署。

## 日志简介

Lambda 运行时捕获我们的函数写入的任何内容，无论是标准输出还是标准错误流。在 Java 中，这些对应于 `System.out` 和 `System.err`。一旦 Lambda 运行时捕获到这些数据，它会将其发送到 CloudWatch Logs。如果您是 AWS 的新手，这需要更详细的解释！

CloudWatch Logs 由几个组件组成。其中主要的是日志捕获服务。它便宜、可靠、易于使用，并且可以处理您可能遇到的所有规模。

一旦 CloudWatch Logs 捕获到日志消息，您有几种方法可以查看或处理它们。最简单的方法是使用 AWS Web 控制台中的 CloudWatch Logs 日志查看器。

有多种方法可以实现这一点，但现在请在 AWS Web 控制台中打开您 Lambda 函数页面（如我们在 “运行 Lambda 函数” 中所示）。如果您点击该页面的监控选项卡，您应该能够看到一个 *在 CloudWatch 中查看日志* 按钮—点击它，如 图 3-2 所示。

![images/ch03_image02.png](img/awsl_0302.png)

###### 图 3-2\. 访问 Lambda 日志

接下来您将看到的内容将取决于 CloudWatch 控制台的工作方式，但如果您尚未看到日志输出，请点击蓝色的 *搜索日志组* 按钮并向下滚动至最近的日志行。然后您应该能够看到类似 图 3-3 中的内容。

![images/ch03_image03.png](img/awsl_0303.png)

###### 图 3-3\. Lambda 日志

注意第二行就是我们从 Lambda 函数中编写的输出。

没有一个优秀的、自重的 Java 程序员会真正使用 `System.out.println` 进行生产日志记录—日志记录框架提供了更多的灵活性和控制日志行为的功能。我们将在 “日志记录” 中详细讨论日志记录实践。

# 输入，输出

当执行 Lambda 函数时，它总是会传递一个输入参数，通常称为 *事件*。在 Lambda 执行环境中，此事件始终是一个 JSON 值，在我们到目前为止的示例中，我们一直在手动创建一个字符串—这个字符串本身就是有效的 JSON。

在实际使用中，Lambda 函数的输入将是一个表示来自某些其他组件或系统的事件的 JSON 对象。例如，它可能是表示 HTTP 请求的详细信息，或者是上传到 S3 存储服务的图像的一些元数据。在本书的后面章节中我们将详细讨论如何将事件源与 Lambda 函数绑定—参见 “Lambda 事件源”。

我们在测试事件中创建的 JSON，或者来自事件源的 JSON，会传递给 Lambda Java 运行时。在大多数用例中，Lambda Java 运行时会自动为我们反序列化此 JSON 负载，并且我们有几种选项来指导此过程。

正如您在前一节中看到的，当我们同步调用一个函数时，我们可以将一个有用的值返回给环境。Lambda Java 运行时会自动将此返回值序列化为 JSON。

Java 运行时如何执行这些序列化和反序列化操作取决于我们在函数签名中指定的类型，因此现在是时候深入了解使 Lambda 函数在静态上有效的因素了。

## Lambda 函数方法签名

有效的 Java Lambda 方法必须符合以下四种签名之一：

+   *`output-type handler-name`*`(`*`input-type`* ` `input`) `

+   *`output-type handler-name`*`(`*`input-type`* ` `input`, Context context) `

+   `void` *`handler-name`*`(InputStream is, OutputStream os)`

+   `void` *`handler-name`*`(InputStream is, OutputStream os, Context context)`

其中：

+   *`output-type`* 可以是 `void`、Java 原始类型或可 JSON 序列化的类型。

+   *`input-type`* 是 Java 原始类型或可 JSON 序列化的类型。

+   `Context` 指的是 `com.amazonaws.services.lambda.runtime.Context`（我们将在本章稍后详细描述）。

+   `InputStream` 和 `OutputStream` 是指 `java.io` 包中的相应类型。

+   *`handler-name`* 可以是任何有效的 Java 方法名称，并且我们在应用程序的配置中引用它。

Java Lambda 方法可以是实例方法也可以是静态方法，但必须是公共的。

包含 Lambda 函数的类不能是抽象的，并且必须具有无参数构造函数——可以是默认构造函数（即未指定任何构造函数）或显式的无参数构造函数。考虑使用构造函数的主要原因之一是在 Lambda 调用之间缓存数据，这是我们稍后会详细讨论的高级主题—参见 “Caching”。

除了这些限制外，Java Lambda 函数没有静态类型要求。您不需要实现任何接口或基类，尽管如果希望可以这样做。AWS 提供了一个 `RequestHandler` 接口，如果您想非常明确地指定 Lambda 类的类型，但我们从未发现有必要使用这个接口。此外，您可以扩展自己的类（符合构造函数规则），但我们同样发现这种能力很少有用。

您可以在一个类中定义多个具有不同名称的 Lambda 函数，但我们通常不建议这种风格。由于两个不同的 Lambda 函数永远不会在同一个执行环境中运行，我们发现将每个函数的代码清晰地分开可以让后续的工程师更容易理解。

Lambda 函数在静态上与某些其他应用程序框架相比较简单。前面列出的前两个签名是 Java Lambda 最常见的签名，接下来我们将看看它们。

## 在 SAM 模板中配置处理函数

到目前为止，我们仅对 SAM 模板文件 *template.yaml* 进行了一次更改，即更改函数的名称。在我们继续之前，我们需要查看该文件中的另一个属性：`Handler`。

打开 *template.yaml* 文件，您会看到 `Handler` 当前设置为 `book.HelloWorld::handler`。这意味着对于此 Lambda 函数，Lambda 平台将尝试在名为 `book` 的包中的名为 `HelloWorld` 的类中找到一个名为 `handler` 的方法。

如果你在名为`old.macdonald.farm`的包中创建了一个名为`Cow`的新类，并且你有一个名为`moomoo`的方法作为你的 Lambda 函数，那么你应该将`Handler`设置为`old.macdonald.farm.Cow::moomoo`。

有了这些信息，你就可以创建一些新的 Lambda 处理程序了！

## 基本类型

示例 3-1 展示了一个具有三个不同 Lambda 处理函数的类（是的，我们刚才说过在实际使用中我们不倾向于在一个类中使用多个 Lambda 函数—这里为了简洁起见这样做了！）

##### 示例 3-1\. 基本类型的序列化和反序列化

```java
package book;

public class StringIntegerBooleanLambda {
  public void handlerString(String s) {
    System.out.println("Hello, " + s);
  }

  public boolean handlerBoolean(boolean input) {
    return !input;
  }

  public boolean handlerInt(int input) {
    return input > 100;
  }
}
```

要尝试这段代码，请将新的类`StringIntegerBooleanLambda`添加到你的源代码树中，更改*template.yaml*文件中的`Handler`（例如，改为`book.StringIntegerBooleanLambda::handlerString`），然后运行你的打包和部署命令。

第一个函数与我们在前一节中描述的相同。我们可以通过使用 JSON 对象`"world"`调用它进行测试，由于它有一个`void`返回类型，它适用于异步使用。

###### 提示

从现在开始，当我们在示例中说要调用一个函数时，假设我们是指在没有特别指定的情况下以*同步*方式调用它。你可以通过在终端调用时使用`--invocation-type RequestResponse`标志或在 AWS Web 控制台中使用*测试*功能来实现这一点。

第二个函数可以用布尔值调用—任何 JSON 值`true`、`false`、`"true"`或`"false"`—它也会返回一个布尔值，在这种情况下是输入的反向。

最后一个函数接受一个整数（可以是 JSON 整数或 JSON 字符串中的数字，例如`5`或`"5"`），并返回一个布尔值。

在第二和第三个示例中，我们使用了一个原始类型，但如果你愿意，你可以使用装箱类型。例如，你可以使用`java.lang.Integer`而不是简单的`int`。

在所有这些情况中发生的情况是 Lambda Java 运行时代表我们将 JSON 输入反序列化为简单类型。如果传递的事件无法反序列化为指定的参数类型，你将收到一个失败消息，消息开头如下：

```java
An error occurred during JSON parsing: java.lang.RuntimeException
```

字符串、整数和布尔值是唯一明确记录为支持的基本类型，但通过一些实验，我们发现其他基本类型，如双精度和浮点数，也包括在内。

## 列表和映射

JSON 还包括数组和对象/属性（参见示例 3-2）。Lambda Java 运行时将自动将其反序列化为 Java `List`和`Map`，并且还会将输出的`List`和`Map`序列化为 JSON 数组和对象。

##### 示例 3-2\. 列表和映射的序列化和反序列化

```java
package book;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.stream.IntStream;

public class ListMapLambda {
  public List<Integer> handlerList(List<Integer> input) {
    List<Integer> newList = new ArrayList<>();
    input.forEach(x -> newList.add(100 + x));
    return newList;
  }

  public Map<String,String> handlerMap(Map<String,String> input) {
    Map<String, String> newMap = new HashMap<>();
    input.forEach((k, v) -> newMap.put("New Map -> " + k, v));
    return newMap;
  }

  public Map<String,Map<String, Integer>>
    handlerNestedCollection(List<Map<String, Integer>> input) {
    Map<String, Map<String, Integer>> newMap = new HashMap<>();
    IntStream.range(0, input.size())
          .forEach(i -> newMap.put("Nested at position " + i, input.get(i)));
    return newMap;
  }
}
```

使用 JSON 数组`[ 1, 2, 3 ]`调用函数`handlerList()`返回`[ 101, 102, 103 ]`。使用 JSON 对象`{ "a" : "x", "b" : "y"}`调用函数`handlerMap()`返回`{ "New Map → a" : "x", "New Map → b" : "y" }`。

此外，您可以如预期般使用嵌套的集合；例如，通过调用 `handlerNestedCollection()` 来执行

```java
[
  { "m" : 1, "n" : 2 },
  { "x" : 8, "y" : 9 }
]
```

返回

```java
{
  "Nested at position 0": { "m" : 1, "n" : 2},
  "Nested at position 1": { "x": 8, "y" : 9}
}
```

最后，您也可以只使用 `java.lang.Object` 作为输入参数的类型。虽然在生产中很少有用（除非您不关心输入参数的值，有时这是一个有效的用途），但在开发时，如果不知道事件的确切格式，这可能很方便。例如，您可以在参数上使用 `.getClass()` 来查找它的实际类型，打印出 `.toString()` 的值等等。稍后我们会展示另一种更好的方法来获取事件的 JSON 结构。

## POJO 和生态系统类型

对于非常简单的输入类型，前面的输入类型效果很好。对于更复杂的类型，另一种选择是使用 Lambda Java 运行时的自动 POJO（普通的 Java 对象）序列化。示例 3-3 展示了我们如何同时用于输入和输出。

##### 示例 3-3\. POJO 序列化和反序列化

```java
package book;

public class PojoLambda {
  public PojoResponse handlerPojo(PojoInput input) {
    return new PojoResponse("Input was " + input.getA());
  }

  public static class PojoInput {
    private String a;

    public String getA() {
      return a;
    }

    public void setA(String a) {
      this.a = a;
    }
  }

  public static class PojoResponse {
    private final String b;

    PojoResponse(String b) {
      this.b = b;
    }

    public String getB() {
      return b;
    }
  }
}
```

显然，这只是一个非常简单的案例，但它展示了 POJO 序列化的实际效果。我们可以使用输入 `{ "a" : "Hello Lambda" }` 执行此 Lambda，并返回 `{ "b" : "Input was Hello Lambda" }`。让我们仔细看一下代码。

首先，我们有我们的处理函数 `handlerPojo()`。它以 `PojoInput` 类型作为输入，这是我们定义的 POJO 类。POJO 输入类可以是静态嵌套类，就像我们在这里写的一样，也可以是常规（外部）类。重要的是，它们需要有一个空的构造函数，并且必须有字段的 setter，这些 setter 遵循从输入 JSON 中反序列化预期字段的命名规则。如果找不到与 setter 同名的 JSON 字段，则 POJO 字段将保持为 null。输入的 POJO 对象需要是可变的，因为运行时会在实例化后修改它们。

我们的处理函数会检查 POJO 对象，并创建 `PojoResponse` 类的新实例，然后将其传回 Lambda 运行时。Lambda 运行时通过反射所有的 `get…` 方法将其序列化为 JSON。POJO 输出类的限制较少——由于它们不是由 Lambda 运行时创建或变异的，因此您可以根据自己的意愿构建它们，也可以使它们成为不可变对象。与输入类一样，POJO 输出类可以是静态嵌套类或常规（外部）类。

对于 POJO 输入和输出类，您可以进一步嵌套 POJO 类，使用相同的规则来序列化/反序列化嵌套的 JSON 对象。此外，您可以在输入和输出中混合使用我们讨论过的 POJO 和集合类型（`List` 和 `Map`）。

我们之前给出的示例基本上遵循了您在线上看到的大部分文档：为字段使用 *JavaBean* 约定。然而，如果您不想在输入类中使用 setter 或在输出类中使用 getter，您也可以使用公共字段。例如，示例 3-4 展示了另一个例子。

##### 示例 3-4\. POJO 序列化和反序列化的备选定义

```java
package book;

public class PojoLambda {
  public PojoResponse handlerPojo(PojoInput input) {
    return new PojoResponse("Input was " + input.c);
  }

  public static class PojoInput {
    public String c;
  }

  public static class PojoResponse {
    public final String d;

    PojoResponse(String d) {
      this.d = d;
    }
  }
}
```

我们可以使用输入 `{ "c" : "Hello Lambda" }` 来执行这个 Lambda 函数，它将返回 `{ "d" : "Input was Hello Lambda" }`。

POJO 输入反序列化的主要用途之一是将 Lambda 函数与 AWS 生态系统 Lambda 事件源之一绑定。以下是一个示例，展示了如何处理上传到 S3 存储服务的对象事件的处理函数：

```java
public void handler(S3Event input) {
  // …
}
```

`S3Event`是你可以从 AWS 库依赖中访问的一种类型——我们将在“示例：构建无服务器数据流水线”中进一步讨论此问题。你也可以自由构建自己的 POJO 类来处理 AWS 事件。

## 流

到目前为止，我们讨论的输入/输出类型在实际使用 Lambda 中将对你非常有用，甚至可能涵盖所有场景。但是如果你有一个相当动态和/或复杂的结构，而你不能或不想使用先前的反序列化方法的话，该怎么办？

答案是使用前述列表的选项 3 或 4 的有效签名，利用`java.io.InputStream`作为事件参数。这使你可以访问传递给 Lambda 函数的原始字节。

使用`InputStream`的 Lambda 的签名略有不同，因为它始终具有`void`返回类型。如果将`InputStream`作为参数，你还必须将`java.io.OutputStream`作为第二个参数。要从这样的处理函数中返回结果，你需要向`OutputStream`写入内容。

示例 3-5 展示了一个可以处理流的处理程序。

##### 示例 3-5\. 使用流作为处理参数

```java
package book;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;

public class StreamLambda {
  public void handlerStream(InputStream inputStream, OutputStream outputStream)
    throws IOException {
    int letter;
    while((letter = inputStream.read()) != -1)
    {
      outputStream.write(Character.toUpperCase(letter));
    }
  }
}
```

如果我们使用输入 `"Hello World"` 执行这个处理程序，它将将 `"HELLO WORLD"` 写入输出流，这将成为函数的结果。

如果你正在使用`InputStream`，你可能会想要使用自己的 JSON 操作代码，但我们会将这留给读者作为练习。此外，你应该保持流的良好处理习惯——错误检查、关闭等。

想要了解更多相关内容，请参阅官方文档中关于[在处理函数中使用流](https://oreil.ly/oXm39)的说明。

一种特别方便的 Lambda 函数使用场景是在开发时，当你不知道编写代码的事件结构时。示例 3-6 将事件记录到 CloudWatch Logs 中，以便你查看其内容。

##### 示例 3-6\. 记录接收到的事件到 CloudWatch Logs 中

```java
package book;

import java.io.InputStream;
import java.io.OutputStream;

public class WhatIsMyLambdaEvent {
  public void handler(InputStream is, OutputStream os) {
    java.util.Scanner s = new java.util.Scanner(is).useDelimiter("\\A");
    System.out.println(s.hasNext() ? s.next() : "No input detected");
  }
}
```

## 上下文

到目前为止，我们已经涵盖了前述列表中的签名格式 1 和 3，那么 2 和 4 是什么呢？`Context`对象又是什么？

在我们到目前为止的所有示例中，Lambda 处理程序函数所接收的唯一输入是发生的事件。但这并不是处理程序在处理时唯一能接收的信息。此外，您可以在任何处理程序参数列表的末尾添加一个`com.amazonaws.services.lambda.runtime.Context`参数，运行时将传入一个您可以使用的有趣对象。让我们看一个例子（示例 3-7）。

##### 示例 3-7\. 检查 Context 对象

```java
package book;

import com.amazonaws.services.lambda.runtime.Context;

import java.util.HashMap;
import java.util.Map;

public class ContextLambda {
  public Map<String,Object> handler (Object input, Context context) {
    Map<String, Object> toReturn = new HashMap<>();
    toReturn.put("getMemoryLimitInMB", context.getMemoryLimitInMB() + "");
    toReturn.put("getFunctionName",context.getFunctionName());
    toReturn.put("getFunctionVersion",context.getFunctionVersion());
    toReturn.put("getInvokedFunctionArn",context.getInvokedFunctionArn());
    toReturn.put("getAwsRequestId",context.getAwsRequestId());
    toReturn.put("getLogStreamName",context.getLogStreamName());
    toReturn.put("getLogGroupName",context.getLogGroupName());
    toReturn.put("getClientContext",context.getClientContext());
    toReturn.put("getIdentity",context.getIdentity());
    toReturn.put("getRemainingTimeInMillis",
                   context.getRemainingTimeInMillis() + "");
    return toReturn;
  }
}
```

这是我们第一个需要使用 Java 标准库之外类型的完整示例。我们将在下一章节更详细地讨论依赖关系和打包，但现在请在您的*pom.xml*文件的根元素下的任何位置添加以下部分：

```java
<dependencies>
  <dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-lambda-java-core</artifactId>
    <version>1.2.0</version>
    <scope>provided</scope>
  </dependency>
</dependencies>
```

当您现在运行`mvn package`时，它将使用 AWS 提供的核心 Lambda 库来编译您的代码，使您能够使用`Context`接口。

`Context`对象为我们提供了有关当前 Lambda 调用的信息。在 Lambda 事件处理过程中，我们可以使用这些信息。当我们调用该示例（传入任何输入事件——它不会被使用）时，我们将得到类似以下结果：

```java
{
  "getFunctionName": "ContextLambda",
  "getLogStreamName": "2019/07/24/[$LATEST]0f1b1111111111111111111111111111",
  "getInvokedFunctionArn":
    "arn:aws:lambda:us-west-2:181111111111:function:ContextLambda",
  "getIdentity": {
    "identityId": "",
    "identityPoolId": ""
  },
  "getRemainingTimeInMillis": "2967",
  "getLogGroupName": "/aws/lambda/ContextLambda",
  "getLogger": {},
  "getFunctionVersion": "$LATEST",
  "getMemoryLimitInMB": "512",
  "getClientContext": null,
  "getAwsRequestId": "2108d0a2-a271-11e8-8e33-cdbf63de49d2"
}
```

所有不同的`Context`字段都在[AWS 文档](https://oreil.ly/oE2hP)中有描述。

在特定事件处理期间，这些字段中的大多数都将保持不变，但`getRemainingTimeInMillis()`是一个显著的例外。它与*超时*相关，这是我们接下来要看的内容。

# 超时

Lambda 函数受可配置的超时限制。您可以在创建函数时指定此超时时间，或者稍后在函数的配置中更新它。

在撰写本文时，*最大*超时时间为 15 分钟。这意味着单次 Lambda 函数调用的最长运行时间为 15 分钟。这个限制是 AWS 未来可能会增加的，之前也曾增加过——长期以来，最大超时时间为 5 分钟。

到目前为止，我们的示例中没有指定超时设置，因此默认为 3 秒。这意味着如果我们的函数在 3 秒内没有完成执行，则 Lambda Java 运行时将中止它。稍后您将在一个示例中看到这个情况。

在前面的部分中，我们查看了`Context`对象。调用`context.getRemainingTimeInMillis()`将告诉您在执行期间的任何给定点剩余多少时间可以运行，然后函数将由运行时中止。后续调用将提供更新的持续时间。如果您正在编写一个相当长寿的 Lambda 并希望在超时发生之前保存任何状态，则这将非常有用。

您可能会问自己一个问题——为什么不总是将超时配置为最大的 900 秒？正如我们将在下一节中进一步探讨的那样，Lambda 的成本主要基于函数运行的时间——如果您的函数最多只能运行 10 秒，那么您不希望十亿次调用花费 90 倍的时间，因为您将被收取 90 倍的费用。

超时 *不* 包括函数实例化时的时间——换句话说，超时期间不会在函数的 *冷启动* 期间启动。或者更准确地说，超时仅适用于 Lambda 调用我们的 `handler` 方法后的时间。我们在 “冷启动” 中进一步讨论了冷启动。

15 分钟的最大超时对 Lambda 函数来说是一个重要的约束——如果您正在编写需要超过 15 分钟的功能，您需要将其拆分为多个协调的 Lambda 函数，或者根本不使用 Lambda。

理论足够了，让我们看看超时是如何发挥作用的。

示例 3-8 展示了一个 Lambda 函数，该函数将查询剩余时间，最终由于超时而失败。

##### 示例 3-8\. 使用 Context.getRemainingTimeInMillis() 查看超时

```java
package book;

import com.amazonaws.services.lambda.runtime.Context;

public class TimeoutLambda {
  public void handler (Object input, Context context) throws InterruptedException {
    while(true) {
      Thread.sleep(100);
      System.out.println("Context.getRemainingTimeInMillis() : " +
        context.getRemainingTimeInMillis());
    }
  }
}
```

更新您的 *template.yaml* 文件，在函数的 `Properties` 部分添加一个名为 `Timeout` 的新属性。将值设置为 `2`——这意味着函数的超时现在是两秒。还要记得更新您的 `Handler` 属性。

然后按照通常步骤运行您的包和部署步骤。

如果我们在 Web 控制台中使用测试功能执行此操作，它将因“任务在 2.00 秒后超时”而失败。日志输出将如下所示：

```java
START RequestId: 6127fe67-a406-11e8-9030-69649c02a345 Version: $LATEST
Context.getRemainingTimeInMillis() : 1857
Context.getRemainingTimeInMillis() : 1756
... Cut for brevity ...
Context.getRemainingTimeInMillis() : 252
Context.getRemainingTimeInMillis() : 152
Context.getRemainingTimeInMillis() : 51
END RequestId: 6127fe67-a406-11e8-9030-69649c02a345
REPORT RequestId: 6127fe67-a406-11e8-9030-69649c02a345	Duration: 2001.52 ms
  Billed Duration: 2000 ms 	Memory Size: 512 MB	Max Memory Used: 51 MB
2019-07-24T21:22:30.076Z 444e6ae0-9217-4cd2-8568-7585ca3fafee
  Task timed out after 2.00 seconds
```

这里我们可以看到 `getRemainingTimeInMillis()` 方法被查询，正如我们预期的那样，然后函数最终由于 Lambda 的超时而失败。

# 内存和 CPU

Lambda 函数没有无限量的 RAM，实际上每个函数都配置有一个 `memory-size` 设置。默认设置为 128MB，但对于生产环境的 Java Lambda 函数来说很少足够，因此您应该将 `memory-size` 视为每个函数都需要认真考虑的内容。

`memory-size` 可以小至 64MB，尽管对于 Java Lambda 函数，您可能应该至少使用 256MB。`memory-size` 必须是 64MB 的倍数。

`memory-size` 设置非常重要，不仅决定函数可以使用多少内存——*它也指定了函数可以获取多少 CPU 力量*。实际上，Lambda 函数的 CPU 力量从 64MB 线性扩展到 1792MB。因此，配置为 1024MB RAM 的 Lambda 函数比配置为 512MB RAM 的函数具有两倍的 CPU 力量。

1792MB RAM 的 Lambda 函数获得一个完整的虚拟 CPU 核心——比该设置大的 RAM 设置允许秒级虚拟核心的分数。这值得知道，如果您的代码根本没有多线程，您可能在这种情况下看不到内存设置高于 1792MB 时的 CPU 改进。

###### 注意

我们将讨论 Lambda 执行环境如何与多个线程交互在 “Lambda and Threading”。

但为什么你应该关心这个——为什么不总是将`memory-size`设置为其最大值 3008MB？原因在于成本。AWS 根据两个主要因素收取 Lambda 函数的费用：

+   函数运行时间，四舍五入到最接近的 100 毫秒

+   函数指定使用的内存量

换句话说，给定相同的执行时间，具有 2GB RAM 的 Lambda 函数的执行成本是具有 1GB RAM 的两倍。或者，具有 512MB RAM 的 Lambda 函数的成本是具有 3008MB 的 17％。这在规模上可能会有很大的差异。

这意味着您应该尽可能使用最少的内存吗？不，那并不总是最好的选择。由于具有两倍于较小函数的内存的函数也具有两倍的 CPU 功率，因此它可能需要一半的时间来执行，这意味着成本是相同的，并且可以更快地完成工作。

调整 Lambda 函数的大小有点艺术性。我们建议您从 512MB 到 1GB 之间进行选择，然后随着函数的增大或需要扩展它们而开始调整。

# 环境变量

前两节都是关于 Lambda 自己的系统配置——如果您想为自己的应用程序使用配置，该怎么办？

我们可以为我们的 Lambda 函数指定 *环境变量*。这允许我们在相同代码的不同上下文中更改函数运行方式。例如，通过环境变量指定外部进程的连接设置或安全配置是非常典型的。

让我们试试这个。示例 3-9 显示了一个使用 Java 标准方法读取环境的函数。

##### 示例 3-9\. 使用环境变量

```java
package book;

public class EnvVarLambda {
  public void handler(Object event) {
    String databaseUrl = System.getenv("DATABASE_URL");
    if (databaseUrl == null || databaseUrl.isEmpty())
      System.out.println("DATABASE_URL is not set");
    else
      System.out.println("DATABASE_URL is set to: " + databaseUrl);
  }
}
```

更新 *template.yaml* 文件以指向此新类并执行打包和部署过程。

如果我们运行此函数（使用我们喜欢的任何测试输入），日志输出将包括以下内容：

```java
DATABASE_URL is not set
```

现在再次更新 *template.yaml* 文件，使 `HelloWorldLambda` 部分如下所示（注意你的 YAML 缩进！）：

```java
HelloWorldLambda:
  Type: AWS::Serverless::Function
  Properties:
    FunctionName: HelloWorldJava
    Runtime: java8
    MemorySize: 512
    Handler: book.EnvVarLambda::handler
    CodeUri: target/lambda.jar
    Environment:
      Variables:
        DATABASE_URL: my-database-url
```

打包和部署后，如果我们现在测试函数，日志输出将包括这个代替：

```java
DATABASE_URL is set to: my-database-url
```

我们可以随意更新环境配置。

在使用环境变量时，通常希望存储敏感数据，例如远程服务的访问密钥。有许多安全的 Lambda 使用方式，并在亚马逊的文档中有所解释。

# 概要

AWS Lambda 的编程模型与您可能习惯的其他模型显着不同。

在本章中，您探讨了编写 Lambda 函数的含义——运行环境是什么，函数如何被调用，以及您可以输入和输出函数的不同方式。

然后，您了解了 Lambda 函数的一些配置方面——超时和内存——以及这些设置的含义。最后，您看到了如何通过环境变量应用自己的应用程序配置。

现在您已经知道如何编程 Lambda 函数，在下一章中，我们将研究 Lambda 操作——打包、部署、安全、监控等内容。

# 练习

1.  花些时间逐步完成本章中的描述——Lambda 与您以前构建和运行 Java 应用程序的方式非常不同。

1.  尝试使用 `System.err`——标准错误流——而不是 `System.out` 记录一些内容。日志输出与 `System.out` 有何不同？它是否改变了调用函数的结果，无论是异步还是同步？

1.  故意使用无效输入调用一个函数，以查看前面描述的解析异常：`An error occurred during JSON parsing`。您在哪里看到这个错误？它如何影响调用函数的结果，无论是异步还是同步？

1.  尝试构建自己的 POJO 类型，并使用它们的 JSON 版本调用 Lambda。您更喜欢*JavaBean*风格，还是公共字段？

1.  尝试使用之前描述的 `StreamLambda` 在 Lambda web 控制台中输出整个输入事件与提供的测试事件模板对象之一。

1.  尝试将您的一个类转换为使用静态处理器方法，而不是实例方法，以确认它是否同样有效。
