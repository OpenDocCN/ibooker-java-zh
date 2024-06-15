# 第十六章：附加的 Quarkus 特性

本章包含 Quarkus 的一些特性，这些特性不适合其他章节。当然，这并不会使它们变得不那么有用！在本章中，您将了解以下主题：

+   Quarkus 的模板解决方案，Qute

+   OpenAPI 集成

+   发送电子邮件

+   调度功能

+   应用数据缓存

# 16.1 使用 Qute 模板引擎创建模板

## 问题

您希望创建模板并使用特定数据渲染它们。

## 解决方案

使用 Qute 模板引擎。

Qute 是一个专门设计以满足 Quarkus 的需求，最小化反射使用并支持命令式和响应式编码风格的模板引擎。

Qute 可以作为一个独立库使用（生成报告到磁盘或生成电子邮件正文消息），也可以与 JAX-RS 结合以提供 HTML 内容。

要开始使用 JAX-RS 的 Qute，请添加 `resteasy-qute` 扩展：

```java
./mvnw quarkus:add-extension -Dextensions="quarkus-resteasy-qute"
```

默认情况下，模板存储在 *src/main/resources/templates* 目录及其子目录中。

下面可能是一个简单的文本文件模板：

```java
Hello {name}!
```

模板是一个简单的句子，使用 `name` 参数进行参数化。

要使用具体数据渲染模板，只需注入 `io.quarkus.qute.Template` 实例并提供模板参数：

```java
@Inject
io.quarkus.qute.Template hello; ![1](img/1.png) ![2](img/2.png)

@GET
@Produces(MediaType.TEXT_PLAIN)
public TemplateInstance hello() { ![3](img/3.png)
    final String name = "Alex";
    return hello.data("name", name); ![4](img/4.png)
}
```

![1](img/#co_additional_quarkus_features_CO1-1)

`Template` 实例定义了模板中的操作。

![2](img/#co_additional_quarkus_features_CO1-2)

默认情况下，使用字段名来定位模板；在这种情况下，模板路径为 *src/main/resources/templates/hello.txt*

![3](img/#co_additional_quarkus_features_CO1-3)

无需渲染，因为 RESTEasy 集成了 `TemplateInstance` 对象以渲染内容。

![4](img/#co_additional_quarkus_features_CO1-4)

`data` 方法用于设置模板参数。

如果运行项目，您将能够看到模板如何渲染：

```java
./mvnw compile quarkus:dev

curl http://localhost:8080/hello

Hello Alex!
```

## 讨论

Qute 支持更多语法（如 `include` 和 `insert` 片段、直接注入 CDI bean 或变体模板）以及与其他 Quarkus 部分（如电子邮件或计划任务）的集成。

## 参见

访问以下网站了解更多关于 Qute 的信息：

+   [Quarkus: Qute 参考指南](https://oreil.ly/R1A1S)

# 16.2 使用 Qute 渲染 HTML

## 问题

使用 Qute 渲染 HTML。

## 解决方案

Qute 将 HTML 渲染得如此简单，就像文本一样。唯一需要发生的是 Quarkus 找到与您的注入匹配的模板。模板的实际内容并不重要。

让我们使用模板渲染一个包含更复杂结构的 HTML 页面。在这种情况下，将渲染一个简单的 HTML 报告。创建一个包含报告参数的 POJO 类：

```java
package org.acme.quickstart;

import java.util.ArrayList;
import java.util.List;

import io.quarkus.qute.TemplateData;

@TemplateData ![1](img/1.png)
public class Movie {

    public String name;
    public int year;
    public String genre;
    public String director;
    public List<String> characters = new ArrayList<>();
    public float ratings;

    public int getStars() { ![2](img/2.png)
        return Math.round(ratings);
    }
}
```

![1](img/#co_additional_quarkus_features_CO2-1)

此注释允许 Quarkus 避免在运行时使用反射访问对象

![2](img/#co_additional_quarkus_features_CO2-2)

自定义方法

## 讨论

下面是值得解释的 HTML 模板的一些详细信息。

首先要查看的部分是一个可选的头部，你可以将其放在任何模板中，以帮助 Quarkus 在编译时验证所有表达式：

```java
{@org.acme.quickstart.Movie movie} ![1](img/1.png)
<!DOCTYPE html>
<html>
```

![1](img/#co_additional_quarkus_features_CO3-1)

参数声明；这不是强制性的，但有助于 Quarkus 验证模板的类型安全性

支持基本的语法，如条件语句或循环：

```java
<div class="col-sm-12">
    <dl> {#if movie.year == 0} ![1](img/1.png)
            <dt>Year:</dt> Not Known
        {#else} ![2](img/2.png)
            <dt>Year:</dt> {movie.year}
        {/if}
        {#if movie.genre is 'horror'} ![3](img/3.png)
        <dt>Genre:</dt> Buuuh
        {#else} <dt>Genre:</dt> {movie.genre}
        {/if} <dt>Director:</dt> {movie.director ?: 'Unknown'} ![4](img/4.png)
        <dt>Main Characters:</dt> {#for character in movie.characters} ![5](img/5.png) {character} ![6](img/6.png) {#if hasNext} ![7](img/7.png) -
            {/if}
        {/for} <dt>Rating:</dt>
        <font color="red"> {#for i in movie.stars} ![8](img/8.png)
            <span class="fas fa-xs fa-star"></span> {/for} </font>
    </dl>
</div>
```

![1](img/#co_additional_quarkus_features_CO4-1)

数字类型的条件语句

![2](img/#co_additional_quarkus_features_CO4-2)

否则部分

![3](img/#co_additional_quarkus_features_CO4-3)

使用 `is` 运算符的字符串类型的条件语句

![4](img/#co_additional_quarkus_features_CO4-4)

Elvis 操作符；如果参数是 `null`，则使用默认值

![5](img/#co_additional_quarkus_features_CO4-5)

遍历所有字符

![6](img/#co_additional_quarkus_features_CO4-6)

显示字符信息

![7](img/#co_additional_quarkus_features_CO4-7)

`hasNext` 是一个特殊属性，用于检查是否还有更多元素。

![8](img/#co_additional_quarkus_features_CO4-8)

在 POJO 中定义的方法调用；按照调用中定义的次数进行迭代

###### 提示

在循环内部，可以使用以下隐式变量：`hasNext`、`count`、`index`、`odd` 和 `even`。

###### 警告

目前只能使用 `Iterable`、`Map.Entry`、`Set`、`Integer` 和 `Stream`。

# 16.3 更改 Qute 模板的位置

## 问题

你想要改变 Qute 查找模板的位置。

## 解决方案

你可以通过使用 `io.quarkus.qute.api.ResourcePath` 注解，自定义模板位置（仍然在 *src/main/resources/templates* 中，并将输出到应用程序部署目录中的 *templates* 目录）：

```java
@ResourcePath("movies/detail.html") ![1](img/1.png)
Template movies;
```

![1](img/#co_additional_quarkus_features_CO5-1)

设置模板路径为 *src/main/resources/templates/movies/detail.html*

再次运行应用程序（或者如果已经在运行，请让实时重载完成其工作），然后打开浏览器并输入此 URL：[*http://localhost:8080/movie*](http://localhost:8080/movie)。

# 16.4 扩展 Qute 数据类

## 问题

你想要扩展 Qute 数据类的功能。

## 解决方案

模板扩展方法必须遵循以下规则：

+   必须是静态的。

+   方法不能返回 void。

+   必须包含至少一个参数。第一个参数用于匹配基本数据对象。

###### 提示

你可以使用 *模板扩展* 来添加专门用于报告目的的方法，当你无法访问数据对象源代码时。

通过使用 `@io.quarkus.qute.TemplateExtension` 注解，你可以实现 *模板扩展方法*。在这种情况下，让我们实现一个方法来对 `rating` 数字进行四舍五入：

```java
@TemplateExtension
static double roundStars(Movie movie, int decimals) { ![1](img/1.png) ![2](img/2.png)
    double scale = Math.pow(10, decimals);
    return Math.round(movie.ratings * scale) / scale;
}
```

![1](img/#co_additional_quarkus_features_CO6-1)

第一个参数是 POJO 数据对象。

![2](img/#co_additional_quarkus_features_CO6-2)

可以设置自定义参数

从模板引擎，`movie` 有一个 `roundStars` 方法，带有一个参数，该参数是要舍入的小数位数。

现在在模板中可以调用以下内容：

```java
({movie.roundStars(2)}) ![1](img/1.png)
```

![1](img/#co_additional_quarkus_features_CO7-1)

`Movie` 类没有定义 `roundStars` 方法，但因为它是模板扩展，所以可以访问它

再次运行应用程序（或者如果已经在运行，请让实时重载完成其工作），然后打开浏览器，输入以下 URL：[*http://localhost:8080/movie*](http://localhost:8080/movie)。

输出应与图 16-1 中显示的输出类似。

![qucb 1601](img/qucb_1601.png)

###### 图 16-1\. HTML 输出

# 16.5 描述带有 OpenAPI 的端点

## 问题

您希望使用 OpenAPI 描述您的 REST API。

## 解决方案

使用 SmallRye OpenAPI 扩展。

一旦您使用 Quarkus 创建了一个 RESTful API，您只需要添加 `openapi` 扩展：

```java
./mvnw quarkus:add-extension -Dextensions="openapi"
```

然后重新启动应用程序，使所有内容生效：

```java
./mvnw compile quarkus:dev
```

默认情况下，API 的规范位于 */openapi*。要更改此设置，请使用 `quarkus.smallrye-openapi.path` 配置：

```java
quarkus.smallrye-openapi.path=/rest-api
```

您可以通过[*http://localhost:8080/openapi*](http://localhost:8080/openapi)访问规范：

```java
openapi: 3.0.1
info:
  title: Generated API
  version: "1.0"
paths:
  /task:
    get:
      responses:
        200:
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/SetTask'
    post:
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Task'
      responses:
        200:
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/SetTask'
    delete:
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Task'
      responses:
        200:
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/SetTask'
components:
  schemas:
    Task:
      type: object
      properties:
        complete:
          type: boolean
        description:
          type: string
        reminder:
          format: date-time
          type: string
    SetTask:
      type: array
      items:
        type: object
        properties:
          complete:
            type: boolean
          description:
            type: string
          reminder:
            format: date-time
            type: string
```

基于之前的规范，有 GET、POST 和 DELETE 端点。您还可以看到 DELETE 和 POST 需要一个任务对象。任务需要一个布尔值、一个字符串和一个日期时间。这非常简单易懂。

## 讨论

使用 Quarkus 中的 SmallRye OpenAPI 扩展非常容易创建 OpenAPI 规范。这使您能够轻松查看和理解您的 RESTful API。

SmallRye OpenAPI 是 Eclipse MicroProfile OpenAPI 的实现。OpenAPI 规范是一种标准的、与语言无关的描述和发现 RESTful API 的方式。它既可由人类阅读，也可由机器处理。OpenAPI 文档定义为 JSON 或 YAML。

## 参见

欲了解更多信息，请访问 GitHub 上的以下页面：

+   [Eclipse MicroProfile OpenAPI](https://oreil.ly/hczN4)

+   [MicroProfile OpenAPI 规范](https://oreil.ly/ufzr6)

+   [OpenAPI 规范](https://oreil.ly/uslyb)

在示例 16.6 中，您将学习如何使用 SmallRye OpenAPI 的注解来自定义生成的规范。

# 16.6 自定义 OpenAPI 规范

## 问题

您希望自定义生成的 API 规范。

## 解决方案

使用 SmallRye OpenAPI 扩展的 OpenAPI 注解。

重用上一个示例中创建的任务 API，示例 16.5，可以轻松使用 OpenAPI 注解来添加自定义和进一步的 API 文档：

```java
package org.acme.openapi;

import java.time.LocalDateTime;
import java.util.Collections;
import java.util.LinkedHashMap;
import java.util.Set;

import javax.ws.rs.Consumes;
import javax.ws.rs.DELETE;
import javax.ws.rs.GET;
import javax.ws.rs.POST;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

import org.eclipse.microprofile.openapi.annotations.Operation;
import org.eclipse.microprofile.openapi.annotations.media.Content;
import org.eclipse.microprofile.openapi.annotations.media.Schema;
import org.eclipse.microprofile.openapi.annotations.parameters.Parameter;

  @Path("/task")
  @Produces(MediaType.APPLICATION_JSON)
  @Consumes(MediaType.APPLICATION_JSON)
  public class TaskResource {

    Set<Task> tasks = Collections.newSetFromMap(
        Collections.synchronizedMap(new LinkedHashMap<>()));

    public TaskResource() {
      tasks.add(new Task("First task",
            LocalDateTime.now().plusDays(3), false));
      tasks.add(new Task("Second task",
            LocalDateTime.now().plusDays(6), false));
    }

    @GET
    @Operation(summary = "Get all tasks",
               description = "Get the full list of tasks.")
    public Set<Task> list() {
      return tasks;
    }

    @POST
    @Operation(summary = "Create a new task")
    public Set<Task> add(
        @Parameter(required = true, content =
          @Content(schema = @Schema(implementation = Task.class))) Task task) {
      tasks.add(task);
      return tasks;
    }

    @DELETE
    @Operation(summary = "Remove the specified task")
    public Set<Task> delete(
        @Parameter(required = true,
        content = @Content(schema = @Schema(implementation = Task.class)))
        Task task) {
      tasks.removeIf(existingTask -> existingTask.equals(task));
      return tasks;
    }
  }
```

```java
package org.acme.openapi;

import java.time.LocalDateTime;
import java.util.Objects;

import javax.json.bind.annotation.JsonbDateFormat;

import org.eclipse.microprofile.openapi.annotations.enums.SchemaType;
import org.eclipse.microprofile.openapi.annotations.media.Schema;

public class Task {
    public String description;

    @Schema(description = "Flag indicating the task is complete")
    public Boolean complete;

    @JsonbDateFormat("yyyy-MM-dd'T'HH:mm")
    @Schema(example = "2019-12-25T06:30", type = SchemaType.STRING,
            implementation = LocalDateTime.class,
            pattern = "yyyy-MM-dd'T'HH:mm",
            description = "Date and time for the reminder.")
    public LocalDateTime reminder;

    public Task() {
    }

    public Task(String description,
                LocalDateTime reminder,
                Boolean complete) {
        this.description = description;
        this.reminder = reminder;
        this.complete = complete;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Task task = (Task) o;
        return Objects.equals(description, task.description) &&
                Objects.equals(reminder, task.reminder) &&
                Objects.equals(complete, task.complete);
    }

    @Override
    public int hashCode() {
        return Objects.hash(description, reminder, complete);
    }
}
```

上述代码将创建以下规范：

```java
---
openapi: 3.0.1
info:
  title: Generated API
  version: "1.0"
paths:
  /task:
    get:
      summary: Get all tasks
      description: Get the full list of tasks.
      responses:
        200:
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/SetTask'
    post:
      summary: Create a new task
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Task'
      responses:
        200:
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/SetTask'
    delete:
      summary: Remove the specified task
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Task'
      responses:
        200:
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/SetTask'
components:
  schemas:
    Task:
      type: object
      properties:
        complete:
          description: Flag indicating the task is complete
          type: boolean
        description:
          type: string
        reminder:
          format: date-time
          description: Date and time for the reminder.
          pattern: yyyy-MM-dd'T'HH:mm
          type: string
          example: 2019-12-25T06:30
    SetTask:
      type: array
      items:
        type: object
        properties:
          complete:
            description: Flag indicating the task is complete
            type: boolean
          description:
            type: string
          reminder:
            format: date-time
            description: Date and time for the reminder.
            pattern: yyyy-MM-dd'T'HH:mm
            type: string
            example: 2019-12-25T06:30
```

基于之前的规范，有 GET、POST 和 DELETE 端点。您还可以看到 DELETE 和 POST 需要一个任务对象。任务需要一个布尔值、一个字符串和一个日期时间。这非常简单易懂。

## 讨论

使用各种 OpenAPI 注解提供有关 API 的额外信息，包括描述、摘要和示例。关于这些注解的更多信息可以在规范中和“参见”部分的链接中找到。

使用 Quarkus 进行生成的 OpenAPI 规范进一步定制非常容易。对于最终定制，Quarkus 支持提供静态文件 OpenAPI 规范。为此，您需要将有效的 OpenAPI 规范文件放置在 *META-INF/openapi.yml* 或 *META-INF/openapi.json*。然后 Quarkus 将结合这两个文件并提供一个结合了静态和动态规范的服务。要禁用动态规范生成，只需在 *applications.properties* 文件中使用 `mp.openapi.scan.disable=true` 配置。

## 参见

获取更多信息，请访问 GitHub 上的以下页面：

+   [Eclipse MicroProfile OpenAPI 规范](https://oreil.ly/i47k_)

+   [Eclipse MicroProfile OpenAPI: 注解示例](https://oreil.ly/ITXQz)

+   [Swagger 2.X Annotations: OpenAPI Annotations](https://oreil.ly/ol6nb)

# 16.7 发送同步邮件

## 问题

您希望同步发送邮件。

## 解决方案

利用 Quarkus 邮件扩展。

Quarkus 使得以纯文本和 HTML 发送电子邮件以及添加附件变得非常直观。还有一个简单易用的方法来测试是否正确发送了邮件，而无需设置自己的中继。将 Email Quarkus 扩展添加到现有项目中：

```java
mvn quarkus:add-extensions -Dextensions="mailer"
```

Quarkus 使用 Vert.x Mail 客户端，虽然有两个包装器以便于使用：

```java
@Inject
Mailer mailer;

@Inject
ReactiveMailer reactiveMailer;
```

`Mailer` 类使用标准的阻塞和同步 API 调用，而 `ReactiveMailer` 则使用非阻塞和异步 API 调用，如预期那样。`ReactiveMailer` 将在下一个示例中讨论；这两个类都提供相同的功能。要发送电子邮件，只需使用 `withText` 或 `withHtml` 方法。您需要提供收件人、主题和正文。如果需要添加 CC、BCC 和附件等内容，可以在实际的 `Mail` 实例上进行操作。

您还需要配置 SMTP 提供者（在本例中，我们使用 Gmail TLS）：

```java
quarkus.mailer.from=quarkus-test@gmail.com
quarkus.mailer.host=smtp.gmail.com
quarkus.mailer.port=587
quarkus.mailer.start-tls=REQUIRED

![1](img/1.png)
quarkus.mailer.username=YOUREMAIL@gmail.com
quarkus.mailer.password=YOURGENERATEDAPPLICATIONPASSWORD
```

![1](img/#co_additional_quarkus_features_CO8-1)

这些也可以通过系统属性和/或环境属性进行设置

通过使用 `MockMailbox` 组件可以轻松地测试邮件组件。它是一个简单的组件，包括三个方法：

+   `getMessagesSentTo`

+   `clear`

+   `getTotalMessagesSent`

以下测试演示了这三种方法的全部内容：

```java
package org.acme.email;

import java.util.List;

import javax.inject.Inject;

import io.quarkus.mailer.Mail;
import io.quarkus.mailer.Mailer;
import io.quarkus.mailer.MockMailbox;
import io.quarkus.test.junit.QuarkusTest;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import static org.assertj.core.api.Assertions.assertThat;

@QuarkusTest
public class MailerTest {
    @Inject
    Mailer mailer;

    @Inject
    MockMailbox mbox;

    @BeforeEach
    void clearMBox() {
        mbox.clear(); ![1](img/1.png)
    }

    @Test
    public void assertBasicTextEmailSent() {
        final String mailTo = "test@example.org";
        final String testingSubject = "Testing email";
        final String testingBody = "Hello World!";

        mailer.send(Mail.withText(mailTo,
                testingSubject,
                testingBody));

        assertThat(mbox.getTotalMessagesSent()).isEqualTo(1); ![2](img/2.png)
        List<Mail> emails = mbox.getMessagesSentTo(mailTo); ![3](img/3.png)

        assertThat(emails).hasSize(1);
        Mail email = emails.get(0);

        assertThat(email.getSubject()).isEqualTo(testingSubject);
        assertThat(email.getText()).isEqualTo(testingBody);
    }
}
```

![1](img/#co_additional_quarkus_features_CO9-1)

我们在每次测试开始之前清空邮箱

![2](img/#co_additional_quarkus_features_CO9-2)

使用 `getTotalMessagesSent` 来验证 Quarkus 发送了多少封消息

![3](img/#co_additional_quarkus_features_CO9-3)

验证发送到特定地址的消息

## 讨论

支持常规附件和内联附件。以下是内联附件的简单示例：

```java
    @Test
    void attachmentTest() throws Exception {
        final String mailTo = "test@example.org";
        final String testingSubject = "email with Attachment";
        final String html = "<strong>E-mail by:</strong>" + "\n" +
                "<p><img src=\"cid:logo@quarkus.io\"/></p>";    ![1](img/1.png)

        sendEmail(mailTo, testingSubject, html);

        Mail email = mbox.getMessagesSentTo(mailTo).get(0);
        List<Attachment> attachments = email.getAttachments();

        assertThat(email.getHtml()).isEqualTo(html);
        assertThat(attachments).hasSize(1);
        assertThat(attachments.get(0).getFile())
                .isEqualTo(new File(getAttachmentURI()));
    }

    private void sendEmail(String to, String subject, String body)
          throws URISyntaxException {
        final File logo = new File(getAttachmentURI());

        Mail email = Mail.withHtml(to, subject, body)
                .addInlineAttachment("quarkus-logo.svg",
                        logo,
                        "image/svg+xml",
                        "<logo@quarkus.io>");   ![2](img/2.png)

        mailer.send(email);
    }
```

![1](img/#co_additional_quarkus_features_CO10-1)

请确保通过`content-id`引用内联附件。

![2](img/#co_additional_quarkus_features_CO10-2)

附件的`content-id`

## 参见

欲了解更多信息，请参阅以下内容：

+   16.8 配方

+   [Vert.x 邮件客户端（SMTP 客户端实现）](https://oreil.ly/aqGZU)

# 16.8 响应式发送电子邮件

## 问题

您希望以非阻塞、响应式的方式发送电子邮件。

## 解决方案

利用 Quarkus 邮件扩展。

前一节详细介绍了基础知识。要以响应式方式执行此操作，只需注入`ReactiveMailer`组件并使用它即可。方法是相同的；它们只是返回响应式对应项而不是同步对应项：

```java
package org.acme.email;

import java.util.List;
import java.util.concurrent.CountDownLatch;

import javax.inject.Inject;

import io.quarkus.mailer.Mail;
import io.quarkus.mailer.MockMailbox;
import io.quarkus.mailer.reactive.ReactiveMailer;
import io.quarkus.test.junit.QuarkusTest;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import static org.assertj.core.api.Assertions.assertThat;

@QuarkusTest
public class ReactiveMailerTest {
    @Inject
    ReactiveMailer reactiveMailer;

    @Inject
    MockMailbox mbox;

    @BeforeEach
    void clearMbox() {
        mbox.clear();
    }

    @Test
    public void testReactiveEmail() throws Exception {
        final String mailTo = "test@example.org";
        final String testingSubject = "Testing email";
        final String testingBody = "Hello World!";
        final CountDownLatch latch = new CountDownLatch(1);

        reactiveMailer.send(Mail.withText(mailTo,
                testingSubject,
                testingBody)).subscribeAsCompletionStage().join();

        assertThat(mbox.getTotalMessagesSent()).isEqualTo(1);
        List<Mail> emails = mbox.getMessagesSentTo(mailTo);

        assertThat(emails).hasSize(1);
        Mail email = emails.get(0);

        assertThat(email.getSubject()).isEqualTo(testingSubject);
        assertThat(email.getText()).isEqualTo(testingBody);
    }
}
```

这个测试与上一节中的测试完全相同；唯一的区别是将`CompletionStage`转换为`CompletableFuture`，并调用`join`以返回测试的命令式样式。

## 讨论

Qute 与 Mailer 扩展集成，因此消息的主体内容是从模板中呈现的。

这次您只需要`qute`扩展，因为不需要 RESTEasy 集成：

```java
mvn quarkus:add-extensions -Dextensions="quarkus-qute"
```

主类是`io.quarkus.mailer.MailTemplate`，使用方式与`io.quarkus.qute.Template`相同，但前者包含特定于邮件逻辑的方法：

```java
@ResourcePath("mail/welcome.txt") ![1](img/1.png)
MailTemplate mailTemplate;

CompletionStage<Void> c = hello.to("to@acme.org")
     .subject("Hello from Qute template")
     .data("name", "Alex")
     .send(); ![2](img/2.png)
```

![1](img/#co_additional_quarkus_features_CO11-1)

模板位于*src/main/resources/templates/mail/welcome.txt*

![2](img/#co_additional_quarkus_features_CO11-2)

使用提供的数据从模板文件渲染正文发送电子邮件

以响应式方式发送电子邮件遵循完全相同的方法名称和用法，只是使用了响应式类。这使得切换和理解非常容易。

## 参见

欲了解更多信息，请参阅以下内容：

+   16.7 配方

+   16.1 配方

# 16.9 创建定时任务

## 问题

您希望某些任务按计划运行。

## 解决方案

在 Quarkus 中安排任务快速简单，但提供了高度的控制和定制。Quarkus 有一个与 Quartz 集成的`scheduler`扩展。

创建定时任务非常简单：只需将`@io.quarkus.scheduler.Scheduled`注解添加到应用范围的 bean 中即可。有两个属性可用于指定任务的调度时间：`cron`和`every`。

`cron`属性使用 Quartz cron 语法。如果您不熟悉 Quartz，请注意与标准 cron 语法存在一些差异。您可以在“参见”的链接中了解更多信息。

`every`属性可能是最容易使用的，尽管它有一些微妙之处。`every`使用`Duration#parse`解析字符串。如果表达式以数字开头，则会自动添加`PT`前缀。

`every`和`cron`都会查找以`{`开始和以`}`结束的表达式进行配置。

有一个 `delay` 属性，它接受一个长整型，还有一个 `delayUnit` 属性，它接受一个 `TimeUnit`。这两者一起使用时，将指定一个延迟时间，在此时间后触发器启动。默认情况下，触发器在注册时开始。

这里演示了一个非常简单的用法：

```java
package org.acme.scheduling;

import java.util.concurrent.atomic.AtomicInteger;

import javax.enterprise.context.ApplicationScoped;

import io.quarkus.scheduler.Scheduled;
import io.quarkus.scheduler.ScheduledExecution;

@ApplicationScoped
public class Scheduler {

    private AtomicInteger count = new AtomicInteger();

    int get() {
        return count.get();
    }

    @Scheduled(every = "5s")
    void fiveSeconds(ScheduledExecution execution) {
        count.incrementAndGet();
        System.out.println("Running counter: 'fiveSeconds'. Next fire: "
                + execution.getTrigger().getNextFireTime());
    }
}
```

## 讨论

Qute 可用于定期生成报告。

您只需要 `qute` 扩展，因为不需要 RESTEasy 集成：

```java
mvn quarkus:add-extensions -Dextensions="quarkus-qute"
```

现在必须手动调用 `render()` 方法以获取结果：

```java
@ResourcePath("reports/report_01.html")
Template report;

@Scheduled(cron="0 30 * * * ?")
void generate() {
    final String reportContent = report
        .data("sales", listOfSales)
        .data("now", java.time.LocalDateTime.now())
        .render();
    Files.write(reportOuput, reportContent.getBytes());
}
```

## 参见

欲了解更多信息，请参阅以下内容：

+   [Quartz：Cron 触发器教程](https://oreil.ly/XdQ7r)

+   Recipe 16.1

# 16.10 使用应用程序数据缓存

## 问题

您希望在方法响应时间较长时避免等待时间。

## 解决方案

使用应用程序数据缓存。

有些情况下，方法可能比预期花费更长的时间来响应，可能是因为正在向外部系统发出请求，或者因为执行的逻辑需要较长时间来执行。

改善此情况的一种方法是使用应用程序数据缓存。其思想是保存方法调用的结果，以便对于那些使用相同*输入*调用的方法，返回先前计算的结果。

Quarkus 与 [Caffeine](https://oreil.ly/1NjlX) 集成作为缓存提供者。

要开始使用应用程序数据缓存，请添加 `cache` 扩展：

```java
./mvnw quarkus:add-extension -Dextensions="cache"
```

这里是一个模拟长时间执行的方法：

```java
@GET
@Produces(MediaType.TEXT_PLAIN)
public String hello() {
    long initial = System.currentTimeMillis();
    String msg = greetingProducer.getMessage(); ![1](img/1.png)
    long end = System.currentTimeMillis();
    return msg + " " + (end - initial) + "ms";
}
```

![1](img/#co_additional_quarkus_features_CO12-1)

此逻辑具有随机的休眠时间

如果运行项目，您将能够看到这种延迟：

```java
./mvnw compile quarkus:dev

curl http://localhost:8080/hello
Hello World 4009ms

curl http://localhost:8080/hello
Hello World 3003ms
```

让我们通过使用 `@io.quarkus.cache.CacheResult` 注解来缓存 `getMessage()` 方法调用：

```java
@CacheResult(cacheName = "greeting-cache") ![1](img/1.png)
public String getMessage() {
    try {
        TimeUnit.SECONDS.sleep(random.nextInt(4) + 1);
        return "Hello World";
    } catch (InterruptedException e) {
        throw new IllegalStateException(e);
    }

}
```

![1](img/#co_additional_quarkus_features_CO13-1)

为此方法调用创建一个新的缓存

再次运行应用程序（或者如果已经在运行，请让实时重载完成其工作），并重复调用 *[*http://localhost:8080/hello*](http://localhost:8080/hello)*：

```java
curl http://localhost:8080/hello
Hello World 2004ms

curl http://localhost:8080/hello
Hello World 0ms
```

第二次调用方法时，不会调用方法，而是从缓存中返回。Quarkus 为每个调用计算一个缓存键，并检查缓存系统中是否有命中。

要计算缓存键，Quarkus 默认使用所有参数值。如果没有参数方法，则从缓存名称派生键。

## 讨论

`@io.quarkus.cache.CacheKey` 注解可用于方法参数中，以指定确切用于缓存键计算的参数。例如 `public String myMethod(@CacheKey String keyElement1, String notPartOfTheKey)`。

###### 重要提示

`@io.quarkus.cache.CacheKey` 注解不能用于返回 `void` 的方法。

`@io.quarkus.cache.CacheInvalidate` 注解可用于使缓存中的条目失效。调用带有 `@CacheInvalidate` 注解的方法时，将计算缓存键并用于从缓存中移除现有条目。

`@io.quarkus.cache.CacheInvalidateAll` 注解用于使所有缓存条目无效。

每个数据缓存选项都可以在*application.properties*文件中单独配置：

```java
quarkus.cache.caffeine."greeting-cache".initial-capacity=10 ![1](img/1.png)
quarkus.cache.caffeine."greeting-cache".expire-after-write=5S ![2](img/2.png)
```

![1](img/#co_additional_quarkus_features_CO14-1)

`greeting-cache`缓存的内部数据结构的最小总大小

![2](img/#co_additional_quarkus_features_CO14-2)

设置过期时间，在对`greeting-cache`缓存进行写操作后开始计时

再次运行应用程序（或者如果已经运行，请让实时重新加载完成其工作），并重复调用*[*http://localhost:8080/hello*](http://localhost:8080/hello)*：

```java
curl http://localhost:8080/hello
Hello World 2004ms

curl http://localhost:8080/hello
Hello World 0ms

// Wait 5 seconds

curl http://localhost:8080/hello
Hello World 1011ms
```

###### 提示

`quarkus.cache.caffeine."greeting-cache".expire-after-access`属性可用于将缓存的过期时间设置为最近一次读取或写入缓存值后的一段时间。
