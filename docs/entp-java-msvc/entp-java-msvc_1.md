## 第二部分\. 实施企业 Java 微服务

在第二部分中，我们通过涵盖诸如消费其他微服务、服务注册和发现、容错和安全性等主题，更深入地探讨了微服务开发。

这六章也涵盖了从 Cayambe 单体应用中开发微服务混合体，使用你在整本书中开发的微服务。最后，当你学习到在微服务及其混合体之间共享数据时，你将添加 Kafka 的数据流。

## 第六章\. 消费微服务

*本章涵盖*

+   如何消费微服务

+   消费微服务时的选择

消费微服务对许多人来说意味着很多不同的事情。微服务的客户端可以是脚本、网页、其他微服务，或者几乎所有可以发起 HTTP 请求的东西。如果我们涵盖了所有这些，那么这一章将是一本完整的书！

开发一个微服务很有趣，但直到你引入许多相互交互的微服务之前，它并不能让你走得很远。为了使两个服务能够相互交互，你需要一种方法，使得一个服务可以调用另一个服务。

本章提供了使用基于 Java 的库来消费另一个微服务的示例，但所示方法同样适用于任何通用 Java 客户端消费微服务。

在企业 Java 中，两个服务将直接通过服务调用进行交互，如图 6.1 所示。

##### 图 6.1\. 企业 Java 业务服务调用

![图片](img/06fig01_alt.jpg)

服务调用可以通过以下方式实现：

+   使用 EJB 时的`@EJB`注入

+   `@Inject`与 CDI

+   通过`static`方法或变量获取服务实例

+   Spring 依赖注入，无论是基于 XML 还是基于注解

所有这些选项都需要你的两个服务位于同一个 JVM 和运行时环境中，如图 6.1 所示。

在图中，一个微服务正在调用另一个微服务。在图中，它们位于相同的微服务环境中，但不必如此。回顾图 1.4，图 6.2 突出了本章的重点，即解决两个在不同运行时环境中的微服务如何进行通信的问题。

##### 图 6.2\. 消费微服务

![图片](img/06fig02.jpg)

在我们的特定情况下，你将使用第二章中的新 Cayambe 管理微服务，并使用不同的库为其开发客户端。图 6.3 说明了微服务客户端的位置；你使用一种临时方式检索分类数据，直到最终解决方案到位。

##### 图 6.3\. Cayambe 管理微服务客户端

![图片](img/06fig03_alt.jpg)

你将首先通过使用直接处理 HTTP 请求的低级库来了解如何消费微服务。因为它们处理 HTTP 请求，所以可以与不暴露 RESTful 端点的微服务一起使用。然后，你将学习专门设计来简化调用 RESTful 端点所需代码的客户端库。它们在 HTTP 请求上提供了更高层次的抽象，这显著简化了客户端代码，正如你将在我们的示例中看到的那样。本节代码可以在书籍示例代码的/chapter6 目录中找到。对于每个客户端库，将实现一个服务，该服务调用你在第二章中创建的`CategoryResource` RESTful 端点，然后将接收到的数据作为对调用者的响应返回。

| |
| --- |

##### 提示

你可以将`CategoryResource`启动的端口设置为防止与其他微服务发生端口冲突。你将`swarm.port.offset`属性在 Maven 插件中设置为`1`。

| |
| --- |

这些服务中的每一个都需要一个对象来表示从管理服务接收到的类别 JSON。为了方便起见，每个客户端库 Maven 模块都将有自己的`Category`对象，该对象将在从管理服务反序列化响应时使用。

##### 列表 6.1\. `Category`模型类

```
@JsonIdentityInfo(generator = ObjectIdGenerators.PropertyGenerator.class,
property = "id")                                                        *1*
public class Category {

    protected Integer id;

    protected String name;

    protected String header;

    protected Boolean visible;

    protected String imagePath;

    protected Category parent;

    private Collection<Category> children = new HashSet<>();               *2*

    protected LocalDateTime created = LocalDateTime.now();

    protected LocalDateTime updated;

    protected Integer version;

    ...
}
```

+   ***1*** 将键定义为 Category 的 ID，该 ID 用于反序列化作为 JSON 接收到的子集合

+   ***2*** 初始化子集合，以确保即使它是空的，你也能始终拥有一个有效的集合

为了简洁，省略了获取器和设置器方法，但`Category`类的完整源代码可以在书籍的源代码中找到。

此外，这些服务中的每一个都需要访问`ExecutorService`以在新的线程上提交工作。你希望使用 Java EE 提供的，以便所有服务都以相同的方式检索：

```
private ManagedExecutorService executorService() throws Exception {
    InitialContext ctx = new InitialContext();
    return (ManagedExecutorService)
      ctx.lookup("java:jboss/ee/concurrency/executor/default");
}
```

这通过名称进行简单的 JNDI 查找以获取服务，并返回你可以用于提交工作的实例。

| |
| --- |

##### 备注

`ExecutorService`是由 WildFly 定义的。你不需要做任何事情就可以通过 JNDI 检索它。

| |
| --- |

你的服务也可以直接创建一个新的`Thread`来执行任何所需的工作，但这样你的新线程就会在 Java EE 线程池管理之外。这是问题吗？不一定，但如果运行时的线程池大小几乎与可用的 JVM 线程一样大，你可能会遇到问题。在这种情况下，当在 Java EE 线程池之外创建线程时，你可能会耗尽所有可用的 JVM 线程。一般来说，最好不直接创建线程，而是使用`ExecutorService`。

因为你想展示同步和异步使用场景如何导致消费微服务的不同客户端代码，每个使用客户端库的资源将包含两个端点：

+   `/sync`—同步处理调用者的请求

+   `/async`—异步处理调用者的请求

传统上，服务被开发出来以同步方式与完成响应所需的任何其他资源进行通信。随着企业对提高性能和可扩展性的需求不断增加，我们在服务中转向了更大的异步行为。在本章和本书的剩余部分，你将了解同步和异步的使用模式。增强微服务的优势也需要一定程度的异步行为；否则，你将最小化它们分布式特性的优势。如果你选择那条路线，那么你不妨坚持使用单体架构！


##### 注意

你的每个微服务定义了一个名为`categoryUrl`的字段，该字段硬编码为 http://localhost:8081/admin/categorytree。这不是你投入生产时会做的事情，但它有助于简化示例。在后面的章节中，你将看到如何使用服务发现来连接到其他服务。


### 6.1\. 使用 Java 客户端库消费微服务

在本节中，你将看到如何消费使用较低级别库直接处理 HTTP 请求的微服务的示例。尽管这会导致更冗长和额外的数据处理，但它确实提供了最大的灵活性，关于如何进行调用。例如，如果微服务需要与许多类型的 HTTP 资源通信，那么使用这些库可能是一个更好的选择，因为为 RESTful 端点交互添加另一个库是没有意义的。

#### 6.1.1\. java.net

java.net 包中的类从一开始就是 JDK 的一部分。尽管这些年来它们得到了增强和更新，但它们专注于低级 HTTP 交互。它们绝不是为 RESTful 端点消费而设计的，因此需要一定程度的繁琐代码。

让我们看看`DisplayResource`的第一个方法。

##### 列表 6.2\. 使用 java.net 的`DisplayResource`

```
@GET
@Path("/sync")
@Produces(MediaType.APPLICATION_JSON)
public Category getCategoryTreeSync() throws Exception {
    HttpURLConnection connection = null;

    try {
        URL url = new URL(this.categoryUrl);                                *1*
        connection = (HttpURLConnection) url.openConnection();

        connection.setRequestMethod("GET");                                 *2*
        connection.setRequestProperty("Accept", MediaType.APPLICATION_JSON);*3*

        if (connection.getResponseCode() != HttpURLConnection.HTTP_OK) {    *4*
            throw new RuntimeException("Request Failed: HTTP Error code: " +
connection.getResponseCode());
        }

        return new ObjectMapper()                                           *5*
                .registerModule(new JavaTimeModule())                       *6*
                .readValue(connection.getInputStream(), Category.class);    *7*

    } finally {
        assert connection != null;
        connection.disconnect()                                             *8*
    }
}
```

+   ***1*** 创建一个指向你的 CategoryResource 的 URL。

+   ***2*** 将 HTTP GET 设置为连接的请求方法。

+   ***3*** 将“application/json”设置为响应中你将接受的媒体类型。

+   ***4*** 检查非 OK 响应代码。

+   ***5*** 创建一个新的 ObjectMapper 来执行 JSON 反序列化。

+   ***6*** 注册 JavaTimeModule 以处理 JSON 到 LocalDateTime 实例的转换。

+   ***7*** 将你在响应中收到的 InputStream 传递给 ObjectMapper 进行反序列化，到一个 Category 实例中。

+   ***8*** 关闭与 CategoryResource 的连接。

尽管你处理的是一个简单的 RESTful 端点，但与之通信的客户端代码肯定不是，这只是一个同步操作！

下一个列表显示了如果你想要异步处理对微服务的客户端请求，前面的代码将如何改变。

##### 列表 6.3\. 使用 java.net 异步的`DisplayResource`

```
@GET
@Path("/async")
@Produces(MediaType.APPLICATION_JSON)
public void getCategoryTreeAsync(
    @Suspended final AsyncResponse asyncResponse)                 *1*
    throws Exception {

    executorService().execute(() -> {                             *2*
        HttpURLConnection connection = null;

        try {
            // The code to open the connection, check the status code,
            // and process the response is identical to the synchronous
            // example and has been removed.

            asyncResponse.resume(category);                       *3*
        } catch (IOException e) {
            asyncResponse.resume(e);                              *4*
        } finally {
            assert connection != null;
            connection.disconnect();
        }
    });
}
```

+   ***1*** 请求处理将异步进行。

+   ***2*** 将 lambda 表达式传递给执行器进行处理。

+   ***3*** 使用反序列化的 Category 实例恢复 AsyncResponse。

+   ***4*** 使用异常恢复 AsyncResponse。

列表中介绍了你之前未见过的新概念——即 `@Suspended` 和 `AsyncResponse`。这两个部分是 JAX-RS 处理客户端请求异步的核心。`@Suspended` 通知 JAX-RS 运行时，客户端的 HTTP 请求应挂起，直到有响应准备就绪。`AsyncResponse` 指示开发者如何通知运行时响应已准备就绪或未能完成。

它看起来是什么样子？请查看 图 6.4。

下面是 图 6.4 中每个步骤所发生的情况：

1.  从浏览器或其他客户端收到了 HTTP 请求。

1.  `getCategoryTreeAsync()` 触发在单独的线程中执行代码。在 `getCategoryTreeAsync()` 完成后，客户端请求被挂起，处理该请求的 HTTP 请求线程变得可用以处理其他请求。

1.  向外部微服务发出 HTTP 请求。

    ##### 图 6.4\. JAX-RS `AsyncResponse` 处理

    ![](img/06fig04_alt.jpg)

1.  从外部微服务接收到了 HTTP 响应。

1.  响应数据传递给 `asyncResponse.resume()`。

1.  客户端请求在 HTTP 请求线程中被重新激活，并构建响应。

1.  响应返回给浏览器，或返回给发起请求的任何客户端。


##### 警告

在 RESTful 端点中使用 `@Suspended` 并不能阻止调用端点的客户端阻塞。它仅通过允许更大的请求吞吐量来使请求的服务器端受益。不使用 `@Suspended`，JAX-RS 资源只能处理与可用线程数量一样多的请求，因为每个请求都会阻塞线程，直到方法完成。


现在你已经构建了服务，可以启动它们。

切换到书籍示例代码的 /chapter6/admin 目录并运行以下命令：

```
mvn thorntail:run
```

`CategoryResource` 将启动并在浏览器中可用，地址为 http://localhost:8081/admin/categorytree。

现在开始你的 `DisplayResource`。切换到 /chapter6/java-net 目录并运行以下命令：

```
mvn thorntail:run
```

现在可以通过在浏览器中访问它们来访问微服务：http://localhost:8080/sync 和 http://localhost:8080/async。在浏览器中打开上述任一 URL 都会显示当前存在于管理微服务中的类别树。

#### 6.1.2\. Apache HttpClient

使用 Apache HttpClient，你得到了对 java.net 中使用的类的抽象，最小化了与底层 HTTP 连接交互所需的代码。`DisplayResource` 中的代码与之前的代码没有太大不同，但它提高了代码的可读性。

例如，让我们看看你的第一个 `DisplayResource` 方法。

##### 列表 6.4\. 使用 HttpClient 的 `DisplayResource`

```
@GET
@Path("/sync")
@Produces(MediaType.APPLICATION_JSON)
public Category getCategoryTreeSync() throws Exception {
    try (CloseableHttpClient httpclient = HttpClients.createDefault()) {  *1*

        HttpGet get = new HttpGet(this.categoryUrl);                      *2*
        get.addHeader("Accept", MediaType.APPLICATION_JSON);              *3*

        return httpclient.execute(get, response -> {                      *4*
            int status = response.getStatusLine().getStatusCode();
            if (status >= 200 && status < 300) {                          *5*
                return new ObjectMapper()                                 *6*
                        .registerModule(new JavaTimeModule())
                        .readValue(response.getEntity().getContent(),
Category.class);
            } else {
                throw new ClientProtocolException("Unexpected response
status: " + status);
            }
        });
    }
}
```

+   ***1*** 在 try-with-resources 语句内创建一个 HTTP 客户端。

+   ***2*** 使用 CategoryResource URL 端点创建一个 HttpGet 实例。

+   ***3*** 指定你将接受 JSON 响应。

+   ***4*** 执行 HttpGet，传递一个用于响应的处理程序。

+   ***5*** 验证响应代码是否为 OK。

+   ***6*** 从响应中提取 HttpEntity。使用 ObjectMapper 将实体转换为 Category 实例。

即使是这样一个简短的示例，你也能看到在发起 HTTP 请求时客户端代码有多简单。现在让我们看看使用 `@Suspended` 时代码会变得多么简单。

##### 列表 6.5\. 使用 HttpClient 和 `@Suspended` 的 `DisplayResource`

```
@GET
@Path("/async")
@Produces(MediaType.APPLICATION_JSON)
public void getCategoryTreeAsync(@Suspended final AsyncResponse
asyncResponse) throws Exception {
    executorService().execute(() -> {                                       *1*
        try (CloseableHttpClient httpclient = HttpClients.createDefault()) {
            HttpGet get = new HttpGet(this.categoryUrl);

            // The code to initiate the HTTP GET request and convert the
HttpEntity
            // is identical to the synchronous example and has been removed.

            asyncResponse.resume(category);                                 *2*
        } catch (IOException e) {
            asyncResponse.resume(e);
        }
    });
}
```

+   ***1*** 在单独的线程中执行你的调用代码。

+   ***2*** 使用接收到的类别恢复 AsyncResponse。

再次强调，这种方法与我们的同步示例类似，但你使用 `@Suspended` 和 `AsyncResponse` 来告诉 JAX-RS 你希望在调用外部微服务时挂起 HTTP 请求。

如果你已经在 http://localhost:8081 上运行了你的 `CategoryResource` 微服务，你现在可以使用 Apache HttpClient 启动你的新微服务。


##### 警告

在运行此微服务之前，你需要停止任何之前运行的微服务，因为它们使用相同的端口。


切换到 /chapter6/apache-httpclient 目录并运行以下命令：

```
mvn thorntail:run
```

现在可以通过浏览器访问微服务：http://localhost:8080/sync 和 http://localhost:8080/async。就像你之前使用的微服务一样，你将看到一个当前存在于管理微服务中的类别树。

在本节中，你查看了一些专注于直接使用 URL 和 HTTP 请求方法的客户端库。它们在与 HTTP 资源交互时非常出色，但在处理 RESTful 端点时则较为冗长。你能找到进一步简化客户端代码的客户端库吗？

### 6.2\. 使用 JAX-RS 客户端库消费微服务

本节介绍了将抽象级别提升到 HTTP 以上的客户端库。这两个库都提供了专门为与 JAX-RS 端点通信而设计的 API。

#### 6.2.1\. JAX-RS 客户端

JAX-RS 在多年的时间里被定义为 Java EE 的 JSR 311 和 JSR 339 规范的一部分。作为这些规范的一部分，JAX-RS 有一个客户端 API，它为开发者提供了一种更干净的方式来从 JAX-RS 资源调用 RESTful 端点。

那么，使用 JAX-RS 客户端库的好处是什么？它允许你忘记连接到 RESTful 微服务所需的低级 HTTP 连接，并专注于所需的元数据，如下所示：

+   HTTP 方法

+   要传递的参数

+   参数和返回类型的 MediaType 格式

+   必要的 Cookie

+   消费 RESTful 微服务所需的任何其他元数据

当使用 JAX-RS 客户端库时，你需要注册一个提供者来处理响应中 JSON 到 `LocalDateTime` 实例的反序列化。为此，你需要以下列表，你将在我们后续的示例中使用它。

##### 列表 6.6\. `ClientJacksonProvider`

```
public class ClientJacksonProvider implements ContextResolver<ObjectMapper> { *1*

    private final ObjectMapper mapper = new ObjectMapper()                    *2*
            .registerModule(new JavaTimeModule());                            *3*

    @Override
    public ObjectMapper getContext(Class<?> type) {
        return mapper;                                                        *4*
    }
}
```

+   ***1*** 为 ObjectMapper 实例提供 ContextResolver。

+   ***2*** 创建一个新的 ObjectMapper 实例。

+   ***3*** 注册 JavaTimeModule 以处理 LocalDateTime 转换。

+   ***4*** 当请求时返回你创建的 ObjectMapper 实例。

再次，你从你的同步示例端点开始。

##### 列表 6.7\. 使用 JAX-RS 客户端的 `DisplayResource`

```
@GET
@Path("/sync")
@Produces(MediaType.APPLICATION_JSON)
public Category getCategoryTreeSync() {
    Client client = ClientBuilder.newClient();         *1*

    return client
            .register(ClientJacksonProvider.class)     *2*
            .target(this.categoryUrl)                  *3*
            .request(MediaType.APPLICATION_JSON)       *4*
            .get(Category.class);                      *5*
}
```

+   ***1*** 创建一个 JAX-RS 客户端。

+   ***2*** 注册你在 列表 6.6 中定义的提供者。

+   ***3*** 将客户端的目标设置为 CategoryResource URL。

+   ***4*** 指定响应应返回 JSON。

+   ***5*** 发送 HTTP GET 请求并将响应体转换为 Category。

当将前面的列表与任何纯 Java 客户端库进行比较时，你将有一个显著简化且更连贯的代码片段，用于调用外部微服务。

这重要吗？从执行请求和处理响应所需的功能角度来看，一点也不重要。但这一点与开发者理解现有代码或开发新代码的容易程度相比，远没有那么关键。我会把它留给你来判断，但我知道我宁愿看到之前的例子，也不愿看到我们迄今为止看到的任何其他东西。

JAX-RS 客户端库同样可以改善你代码的异步可读性吗？参见下一列表。

##### 列表 6.8\. 使用 JAX-RS 客户端和 `@Suspended`

```
@GET
@Path("/async")
@Produces(MediaType.APPLICATION_JSON)
public void getCategoryTreeAsync(@Suspended final AsyncResponse
asyncResponse) throws Exception {
    executorService().execute(() -> {
        Client client = ClientBuilder.newClient();

        try {
            Category category = client.target(this.categoryUrl)
                    .register(ClientJacksonProvider.class)
                    .request(MediaType.APPLICATION_JSON)
                    .get(Category.class);

            asyncResponse.resume(category);
        } catch (Exception e) {
            asyncResponse.resume(Response                          *1*
                                         .serverError()
                                         .entity(e.getMessage())
                                         .build());
        }
    });
}
```

+   ***1*** 返回你构造的响应，包括异常消息，而不是仅仅传递异常。

与所有异步使用一样，你指定 `@Suspended` 和 `AsyncResponse`。你还使用 `ManagedExecutorService` 为处理你的调用提供一个新线程，并通过 `asyncResponse.resume()` 设置响应。

你还可以使用 JAX-RS 客户端库本身的异步功能。

##### 列表 6.9\. 使用 JAX-RS 客户端和 `InvocationCallback` 的 `DisplayResource`

```
@GET
@Path("/asyncAlt")
@Produces(MediaType.APPLICATION_JSON)
public void getCategoryTreeAsyncAlt(@Suspended final AsyncResponse
asyncResponse) {
    Client client = ClientBuilder.newClient();
    WebTarget target = client.target(this.categoryUrl)
            .register(ClientJacksonProvider.class);
    target.request(MediaType.APPLICATION_JSON)
            .async()                                         *1*
            .get(new InvocationCallback<Category>() {        *2*
                @Override
                public void completed(Category result) {
                    asyncResponse.resume(result);
                }

                @Override
                public void failed(Throwable throwable) {
                    throwable.printStackTrace();
                    asyncResponse.resume(Response
                         .serverError()
                         .entity(throwable.getMessage())
                         .build());
                }
            });
}
```

+   ***1*** 指示你希望调用是异步的。

+   ***2*** 传递带有完成和失败处理方法的 InvocationCallback。

第二个异步版本改变了你的代码中哪些部分在新线程中执行，但它并没有改变最终结果。在 `getCategoryTreeAsync()` 中，你将所有 RESTful 端点代码传递到一个新线程，以便 HTTP 请求线程可以几乎与处理一样快地被解除阻塞。`getCategoryTreeAsyncAlt()` 的不同之处在于它只在新的线程中执行对外部微服务的 HTTP 请求。所有用于发送 HTTP 请求的设置代码都发生在客户端请求相同的线程中。

由于 `getCategoryTreeAsyncAlt()` 使用了为客户端打开的最长时间 HTTP 请求线程，它通过使每个客户端在比必要更长的时间内阻塞线程来降低 RESTful 端点的吞吐量。尽管影响可能很小，但如果请求数量足够大，这种影响是存在的。

为什么展示一个 *较差* 的方法，它会对吞吐量产生负面影响？首先，作为一种展示有多种方式可以实现类似目标的方式。其次，许多微服务可能没有足够多的并发客户端请求，以至于这种性能影响是明显的并导致问题。在这种情况下，开发者可能更愿意选择回调而不是任何其他替代方案——因为当某个选项不会影响性能时，这是一个合理的选择。

在切换到使用 JAX-RS 客户端库时，您简化了调用代码，并使其更容易理解。这当然比使用底层库更令人愉快，但它确实在如何灵活使用方面付出了代价。

会失去什么样的灵活性？对于大多数用例来说，JAX-RS 客户端库不会造成任何影响，但调用使用二进制协议的微服务可能会更困难。根据协议的不同，可能需要开发自定义处理程序和提供者，或者集成提供此类功能的额外第三方库。

切换到 /chapter6/jaxrs-client 目录并运行以下命令：

```
mvn thorntail:run
```

现在可以通过在浏览器中访问它们来访问微服务：http://localhost:8080/sync 和 http://localhost:8080/async。就像我们之前的示例一样，您将看到当前存在于管理微服务中的类别树。

#### 6.2.2\. RESTEasy 客户端

RESTEasy 是 JAX-RS 规范的实现，它不仅可在 WildFly 中使用，还可以单独使用。尽管其客户端库的许多部分与 JAX-RS 客户端 API 提供的相同，但 RESTEasy 提供了一个特别有趣的功能，值得注意。

使用 JAX-RS 客户端库，您可以通过链式调用方法来指定想要调用的 RESTful 端点，从而构建端点、URL 路径、参数、返回类型、媒体类型等的画面。这样做并没有什么问题，但对于更熟悉使用 JAX-RS 创建 RESTful 端点的开发者来说，这并不十分自然。

使用 RESTEasy，您可以重新创建想要与之通信的 RESTful 端点作为接口，并为您生成该接口的代理。这个过程允许您将外部微服务的接口用作如果它存在于您的代码库中一样。

对于您的外部 `CategoryResource` 微服务，您会创建如下所示的接口。

##### 列表 6.10\. `CategoryService`

```
@Path("/admin/categorytree")
public interface CategoryService {
    @GET
    @Produces(MediaType.APPLICATION_JSON)
    Category getCategoryTree();
}
```

这里的代码没有什么特别之处。它看起来像任何其他的 JAX-RS 端点类，除了它是一个接口并且没有方法实现。另一个好处是只需要在接口上定义你的微服务需要的方法。例如，如果一个外部微服务有五个端点，而你的微服务只需要使用一个，那么定义该外部微服务的接口只需要一个方法。你不需要定义整个外部微服务。

这有什么优势吗？当然有！它允许你有一个专注于定义你需要消费的外部微服务的明确定义。如果该微服务上的方法被更新而你不需要，那么你不需要更新你的接口，因为你没有使用那些端点。


##### 注意

采用这种方法，可以在服务和客户端之间共享相同的接口。服务将为实际的端点代码提供接口的实现。

|  |

##### 警告

虽然这种做法是可能的，但不推荐在微服务中使用，因为它变成了两个微服务都依赖的独立库，这会引入发布时间和顺序问题。这是一条危险的道路，最终只会给企业带来持续的痛苦。因此，最好是复制你需要调用的方法。


现在你已经定义了一个映射到你的外部微服务的接口，它该如何使用呢？

##### 列表 6.11\. 使用 RESTEasy 的 `DisplayResource`

```
@GET
@Path("/sync")
@Produces(MediaType.APPLICATION_JSON)
public Category getCategoryTreeSync() {
    ResteasyClient client = new ResteasyClientBuilder().build();           *1*
    ResteasyWebTarget target = client.target(this.categoryUrl)             *2*
            .register(ClientJacksonProvider.class);

    CategoryService categoryService = target.proxy(CategoryService.class); *3*
    return categoryService.getCategoryTree();                              *4*
}
```

+   ***1*** 使用 RESTEasy 创建客户端。

+   ***2*** 设置请求的目标 URL 基路径。

+   ***3*** 生成你的 CategoryService 的代理实现。

+   ***4*** 通过你的代理调用 CategoryResource。

采用这种方法，你将设置所有请求参数（如 URL 路径、媒体类型和返回类型）的操作转移到你的 `CategoryService` 接口上。现在与代理交互的客户端代码表现得就像是一个本地方法调用。通过将常见的请求参数值集中到一个地方，你在代码中获得了进一步的简化。这在微服务可能需要在不同的 RESTful 端点调用相同的对外部微服务时尤为重要，因为你不希望在不改变信息的地方重复信息。

让我们看看一些使用你的代理接口的异步示例。

##### 列表 6.12\. 使用 RESTEasy 和 `@Suspended` 的 `DisplayResource`

```
@GET
@Path("/async")
@Produces(MediaType.APPLICATION_JSON)
public void getCategoryTreeAsync(@Suspended final AsyncResponse
asyncResponse) throws Exception {
    executorService().execute(() -> {
        ResteasyClient client = new ResteasyClientBuilder().build();

        try {
            ResteasyWebTarget target = client.target(this.categoryUrl)
                    .register(ClientJacksonProvider.class);

            CategoryService categoryService =
     target.proxy(CategoryService.class);
            Category category = categoryService.getCategoryTree();
            asyncResponse.resume(category);
        } catch (Exception e) {
            asyncResponse.resume(Response
                                         .serverError()
                                         .entity(e.getMessage())
                                         .build());
        }
    });
}
```

在同步和异步 RESTful 端点之间，你需要做的唯一改变是 JAX-RS 异步要求的 `@Suspended` 和 `@AsyncResponse`，将客户端代码提交到单独的线程进行处理，并在 `asyncResponse.resume()` 上设置成功或失败。

你在使用 RESTEasy 客户端库时采用的代理方法的一个缺点是，它不支持在调用外部微服务时执行回调。因此，使用 RESTEasy 的 `getCategoryTreeAsyncAlt()` 将与使用 JAX-RS 客户端库时相同。

切换到 /chapter6/resteasy-client 目录并运行以下命令：

```
mvn thorntail:run
```

现在可以通过 http://localhost:8080/sync 和 http://localhost:8080/async 访问微服务。每个 URL 都会返回当前存在于管理微服务中的分类树，作为结果。

现在我们已经介绍了几种提供更高层次抽象以与 RESTful 端点交互的客户端库。示例展示了这些库在减少代码复杂性和提高可读性方面的好处。

### 摘要

+   基于 Java 的客户端库，如 java.net 和 Apache HttpClient，提供了对 Java 网络的低级别访问，但比必要的代码更冗长。

+   基于 JAX-RS 的客户端库提供了一个抽象，使得消费微服务变得更加容易。

## 第八章\. 容错和监控策略

*本章涵盖*

+   什么是延迟？

+   为什么微服务需要具备容错能力？

+   电路断路器是如何工作的？

+   有哪些工具可以减轻分布式故障的影响？

你将使用前几章中的示例来扩展 Stripe 和 Payment 的功能，包括在探索容错和监控概念时加入故障缓解。当你的 Payment 微服务通过网络与外部系统通信时，容错尤为重要。你需要预料到在网络通信中可能会出现故障和超时。

### 8.1\. 分布式架构中的微服务故障

图 8.1 回顾了你的微服务分布式架构的外观。

##### 图 8.1\. 分布式架构中的微服务

![](img/08fig01_alt.jpg)

这种分布式架构与故障有何关联？由于你的微服务包含更小的业务逻辑块，而不是包含所有内容的单体，你最终会拥有显著更多的服务需要维护。你不再处理可能只与单个后端服务通信的 UI，该服务处理所有需求。更有可能的是，相同的 UI 现在正在与数十个甚至更多的微服务集成，这些微服务需要与之前的单体一样可靠。

但你的微服务在生产环境中不会失败，对吧？在生产环境中，什么都不会失败！我们可能都曾在某个时刻说过类似的话，通常是在我们被生产环境中的重大故障咬了一口之后！一朝被蛇咬，十年怕井绳！

为什么在没有生产故障的先前经验的情况下，我们倾向于对生产系统的可靠性做出宏伟的声明？部分原因可能是因为我们天生乐观，但主要还是缺乏经验。如果你从未不得不在深夜处理应用程序的生产问题，那么就很难理解关于系统可靠性的合理担忧。

| |
| --- |

**呼叫器噩梦**

我记得在 20 世纪 90 年代末——是的，那时候我在 IT 行业——新手最可怕的经历就是被分配到*值班*。没有什么比凌晨 2 点左右接到故障作业需要修复的通知更糟糕的了，然后试图在早上 8 点员工到达办公室之前完成它们！这些只是夜间的批量作业，但接到电话的焦虑感是可怕的。

我只能想象收到一个生产故障的呼叫（如果呼叫器今天仍然存在）来处理一个需要解决的实时应用程序，因为它正在影响业务的 24/7 运行！

| |
| --- |

这里有一些关于生产系统，尤其是分布式架构，你可能会错误地相信的声明：

+   ***计算设备网络是可靠的**。如果不考虑网络故障的可能性，应用程序在等待不会到达的响应时可能会停滞。更糟糕的是，当网络再次可用时，应用程序将无法重试任何失败的操作。

+   ***请求得到响应没有延迟（称为零延迟）**。忽略网络延迟以及相关的网络数据包丢失，可能会导致带宽浪费和网络数据包丢失增加，而网络流量在无限制增长的情况下。

+   ***网络上的可用带宽没有限制**。如果客户端发送的数据太多，或者请求太多，可用的网络带宽可能会缩小到出现瓶颈并降低应用程序吞吐量的程度。延迟对网络吞吐量的影响可能持续几秒钟或持续存在。

+   ***整个网络对可能的攻击，无论是外部还是内部，都是安全的**。忽视恶意用户，如不满的员工，可能试图对应用程序造成损害的可能性是幼稚的。同样，一个曾经内部的应用程序，如果没有经过适当的安全审查就公开提供，很容易受到外部威胁。即使是对端口的防火墙规则的无害更改也可能无意中使其对外部可访问。

+   ***计算设备在网络上的位置和布局永远不会改变**。当网络发生变化，设备被移动到不同的位置时，可用的带宽和延迟可能会降低。

+   ***一切只有一个管理员**。在企业内部，如果不同网络有不同的管理员，可能会实施冲突的安全策略。在这种情况下，需要跨不同安全网络通信的客户端需要了解双方的要求才能成功通信。

+   ***零传输成本**。尽管通过网络传输物理数据可能没有成本，但在网络建成后维护网络是有成本的。

+   ***整个网络是同质的**。在同质网络中，网络上的每个设备都使用类似的配置和协议。异质网络可能导致本列表前三个点中描述的问题。

所有这些陈述都被称为分布式计算的谬误([www.rgoarchitects.com/Files/fallacies.pdf](http://www.rgoarchitects.com/Files/fallacies.pdf))。

### 8.2. 网络故障

尽管网络可能以许多方式失败，但在这个部分，你将专注于网络延迟和超时。之前，我提到零延迟是分布式计算的谬误之一，这等于在请求和执行请求时没有延迟。

为什么延迟对你的微服务很重要？它几乎影响你的微服务可能想要做的任何事情：

+   调用另一个微服务

+   等待异步消息

+   从数据库读取

+   向数据库写入

如果不考虑网络中存在延迟的事实，你会假设所有消息和数据通信都是几乎瞬时的，假设参与通信的网络设备足够接近。

超时是你在开发微服务时需要留意的另一个关键网络故障来源。超时可能与高延迟相关；请求没有及时得到响应，不仅因为网络延迟，还因为消费微服务的问题。如果你调用的微服务已经宕机，正在经历高负载，或者因为其他任何原因失败，当你尝试消费它时，你将注意到问题，最常见的形式是超时。无法预测超时何时发生，因此你的代码需要意识到超时会发生，以及当你收到超时时你想要如何处理这种情况。

你是立即重试还是稍后延迟重试？你假设标准响应并继续进行吗？

你特别希望减轻这些网络故障。否则，你将你的微服务，以及整个应用程序，暴露在无法恢复的意外网络故障中，除了重新启动服务外没有其他恢复手段。因为你不能每次网络问题发生时都重启服务，你需要开发你的代码以防止重启成为你唯一的选项。

### 8.3. 防御故障

在考虑如何减轻失败时，你当然可以自己实现所需的功能。但你可能不是所有最佳实现方式的专家。即使你是，完成这种实现也需要超过短期的开发周期。你更愿意开发更多的应用程序！尽管你可能能够使用许多不同的库，但在这个案例中，你将使用 Netflix 开源软件中的 Hystrix。

#### 8.3.1\. 什么是 Hystrix？

Hystrix 是一个旨在隔离远程系统、服务和库的访问点、阻止级联故障以及在分布式系统中提供弹性的延迟和容错库。无论在何处失败是不可避免的，例如在分布式系统中，Hystrix 库提高了这些环境中微服务的弹性。

Hystrix 有很多功能，那么这个库是如何做到的呢？我们无法在本章中涵盖 Hystrix 的所有内容；那需要一本完全属于自己的书。但本节提供了 Hystrix 如何执行隔离的高级视图。

图 8.2 展示了一个微服务处理多个用户请求负载的视图。这个微服务需要与外部服务进行通信。在这种情况下，你开发的微服务很容易因为等待外部服务 2 响应而被阻塞。更糟糕的是，你可能会使外部服务过载到完全停止工作的程度。

##### 图 8.2\. 不使用 Hystrix 处理的微服务用户请求

![](img/08fig02_alt.jpg)

正是在这里，Hystrix 发挥作用，作为中间人并调解你的外部通信，以减轻各种故障。图 8.3 通过将`HystrixCommand`实例包装在外部服务调用中，并将配置用于定义其行为（例如，可用的线程数）来将 Hystrix 添加到图中。

在图 8.3 中，每个外部服务都有不同数量的线程可供相应的`HystrixCommand`使用。这表明某些服务可能比其他服务更容易过载，你需要限制你发送的并发请求数量。

##### 图 8.3\. 使用 Hystrix 处理用户请求的微服务

![](img/08fig03_alt.jpg)

通过将外部服务 2 包装到`HystrixCommand`中，你限制了从你的微服务并发调用它的请求数量。尽管你为与该特定外部服务的交互增加了缓解措施，但你只是增加了请求在微服务中失败的可能性，因为你拒绝了发送到外部服务的额外请求！这种情况可能很好，也可能不好；结果取决于外部请求处理你的请求的速度。

这确实提出了一个重要观点。在整个生态系统中的单个微服务中添加故障缓解并不那么有益。使您的微服务成为分布式网络中的更好公民是很好的，但如果网络中的其他人没有相同的缓解措施来与您的微服务交互，您只是将瓶颈和故障点移动了位置。因此，故障缓解成为企业级关注的问题，或者至少在所有相互通信的微服务组中是至关重要的。

您可以在图 8.3 中看到的 Hystrix 的另一个优点是它提供的外部服务之间的隔离。如果对外部服务 2 的调用没有限制，那么它很可能消耗掉 JVM 中所有可用的线程，从而阻止您的微服务处理不需要与外部服务 2 交互的请求！

在本章的剩余部分，我们的方法将是概述故障缓解策略背后的理论，然后展示该策略如何在 Hystrix 中实现。您知道您需要在代码中缓解网络故障，那么您有哪些策略可供选择？

#### 8.3.2. 电路断路器

如果您对家中电气面板中保险丝的工作方式有所了解，您就会理解电路断路器的原理。图 8.4 显示，除非保险丝被跳闸打开，否则电流会无阻碍地通过保险丝流动。

##### 图 8.4. 电气电路断路器状态

![图片](img/08fig04_alt.jpg)

电气面板和软件之间唯一的区别是，软件电路断路器将根据已定义的阈值自动关闭，这些阈值指示电路变得不健康时的水平。

图 8.5 显示了缓解调用外部服务时故障的更大流程的初始部分。随着您进入本章，流程中还将添加更多部分，提供额外的功能以帮助缓解。这一部分主要关注提供电路断路器。

##### 图 8.5. 基本电路的故障缓解流程

![图片](img/08fig05.jpg)

当电路断路器处于闭合状态时，所有请求将继续通过流程。当电路断路器处于开路状态时，请求将提前退出流程。您可以在图 8.5 中看到，您的电路断路器需要电路健康数据，这些数据用于确定电路应该开启还是关闭。除了图 8.5 中的状态外，电路断路器还可以处于半开状态。参见图 8.6。

##### 图 8.6. 电路断路器状态

![图片](img/08fig06_alt.jpg)

这里是电路断路器状态之间的转换：

1.  所有请求都能无阻碍地通过，因为电路处于闭合状态。

1.  当达到故障阈值时，电路变为开路状态。

1.  当电路处于开启状态时，所有请求都会被拒绝，快速失败。

1.  电路的开启超时时间到期。电路移动到半开启状态，以允许单个请求通过。

1.  请求失败或成功：

    1.  单个请求失败，将电路状态恢复为开启。

    1.  单个请求成功，将电路状态恢复为关闭。

在半开启状态下，断路器仍然是官方关闭的。但一旦达到睡眠超时时间，将允许单个请求通过。这个单个请求的成功或失败将决定状态是否回到关闭（单个请求成功），或者是否保持开启状态，直到下一次超时间隔到达时再次尝试。

断路器只是允许或阻止请求通过的一种方式。使其按您期望的方式运行的关键部分是电路健康数据。如果没有捕获任何电路健康数据，无论有多少请求失败或是什么原因，断路器都将始终保持关闭状态。

Hystrix 为断路器提供了合理的默认设置，以处理任何请求的超时、网络拥塞和延迟。让我们看看一个简单的 Hystrix 断路器。

##### 列表 8.1\. `StockCommand`

```
public class StockCommand extends HystrixCommand<String> {           *1*
    private final String stockCode;

    public StockCommand(String stockCode) {
        super(HystrixCommandGroupKey.Factory.asKey("StockGroup"));   *2*
        this.stockCode = stockCode;
    }

    @Override
    protected String run() throws Exception {                        *3*
        // Execute HTTP request to retrieve current stock price
    }
}
```

+   ***1*** 将`String`指定为`HystrixCommand`类型。

+   ***2*** Hystrix 仪表板中用于分组数据的唯一键

+   ***3*** 调用外部服务的执行

您可以使用以下代码同步调用此命令：

```
String result = new StockCommand("AAPL").execute();
```

如果您更喜欢异步执行，您可以使用以下方法：

```
Future<String> fr = new StockCommand("AAPL").queue();
String result = fr.get();
```

在每个示例中，您都期望从执行请求中只得到一个结果，无论您是同步调用还是异步调用。因此，您选择扩展`HystrixCommand`，它适用于单响应执行。

如果您期望多个响应而不是一个，会发生什么？股票价格变化非常频繁，所以每次您想要更新时，不是不断地执行另一个调用不是很好吗？

您需要修改您的断路器以支持返回一个可以发出多个响应的`Observable`的命令。您将订阅此`Observable`以处理接收到的每个响应。将每个响应的处理识别为**反应式**执行。

| |
| --- |

##### 定义

`Reactive`是一个形容词，意为**对情况做出反应，而不是创造或控制它**。当您使用`Observable`并监听从中发出的结果时，您是在对每个发出的结果做出**反应**。这种方法的优点是，您在等待每个结果发出时不会阻塞。

| |
| --- |

让我们修改您的命令以提供一个`Observable`。

##### 列表 8.2\. `StockObservableCommand`

```
public class StockObservableCommand extends HystrixObservableCommand<String> { *1*
    private final String stockCode;

    public StockObservableCommand(String stockCode) {
        super(HystrixCommandGroupKey.Factory.asKey("StockGroup"));             *2*
        this.stockCode = stockCode;
    }

    @Override
    protected Observable<String> construct() {                                 *3*
        // Return an Observable that executes an HTTP Request
    }
}
```

+   ***1*** 将`String`指定为`HystrixObservableCommand`类型。

+   ***2*** Hystrix 仪表板中用于分组数据的唯一键

+   ***3*** 返回一个执行调用外部服务的`Observable`。

如果你希望命令在创建 `Observable` 时立即执行，你可以请求一个热 `Observable`：

```
Observable<String> stockObservable =
new StockObservableCommand(stockCode).observe();
```

通常，热 `Observable` 会发出响应，无论是否有订阅者，这使得在没有订阅者的情况下，响应可能会完全丢失。但 Hystrix 使用 `ReplaySubject` 来为你捕获这些响应，允许它们在你订阅 `Observable` 时回放给你的监听器。

你也可以使用一个 *冷* `Observable`：

```
Observable<String> stockObservable =
new StockObservableCommand(stockCode).toObservable();
```

对于冷 `Observable`，执行不会在订阅者订阅之前触发。这保证了任何订阅者都将接收到 `Observable` 所产生的所有通知。

应该使用哪种类型的 `Observable` 取决于你的情况。如果一个监听器可以承受错过一些初始数据，特别是如果他们不是 `Observable` 的第一个订阅者，那么热 `Observable` 是合适的。然而，如果你希望监听器接收所有数据，那么冷 `Observable` 是更好的选择。


##### 注意

虽然 `HystrixCommand` 支持从其非响应式方法 `execute()` 和 `queue()` 返回一个 `Observable`，但它们总是只发出单个值。


#### 8.3.3\. 隔舱

软件中的隔舱提供了与船舶中类似的策略，通过隔离不同的部分来防止一个部分的故障影响其他部分。对于船舶来说，单个水密舱的故障不会蔓延到其他部分，因为它们由隔舱分开。

软件隔舱是如何达到相同的效果的？通过减轻微服务正在经历或即将经历的压力。*隔舱* 允许你限制对组件或服务的并发调用数量，以防止网络因请求而饱和，这会增加系统中所有请求的延迟。图 8.7 将隔舱策略作为你流程中的下一步添加。

##### 图 8.7\. 带有隔舱的故障缓解流程

![](img/08fig07_alt.jpg)

你可以在任何断路器之后添加隔舱。如果断路器是开启的，不需要检查隔舱，因为你处于错误状态。当你处于关闭状态时，隔舱可以防止执行过多的请求，这可能会创建网络瓶颈。

你可能需要调用数据库服务来执行非常密集和耗时的计算，例如。如果你知道外部服务可能需要 10 秒来响应，你不想向该服务发送超过每分钟六个请求。如果你发送超过六个，你的请求将被排队等待后续处理，这会导致你的微服务无法释放对客户端请求的控制。这是一个难以打破的恶性循环，可能会导致微服务中的级联故障。图 8.7 中的隔舱执行其检查并指示你是否可以继续处理请求或是否需要拒绝。

你会如何实现软件舱壁？两种最常见的方法是计数器和线程池。*计数器*允许你设置在任何时候可以同时活跃的最大并行请求数量。*线程池*也限制了同时活跃的并行请求数量，但通过限制池中可用于执行请求的线程数量。对于线程池舱壁，会创建一个特定的池来处理对特定外部服务的请求，允许不同的外部服务相互隔离，同时也与执行你的微服务的线程隔离。

被拒绝的请求的详细信息会提供给断路器健康数据，以便更新计数器，以便在下一次需要计算断路器状态时使用。

作为软件舱壁，Hystrix 为线程池(`THREAD`)和计数器(`SEMAPHORE`)提供了执行策略。默认情况下，`HystrixCommand`使用`THREAD`，而`HystrixObservableCommand`使用`SEMAPHORE`。

`HystrixObservableCommand`不需要通过线程进行舱壁处理，因为它已经通过`Observable`在单独的线程中执行。你可以使用`THREAD`与`HystrixObservableCommand`一起使用，但这样做并不增加安全性。如果你想在`SEMAPHORE`中运行`StockCommand`，它看起来会像以下列表。

##### 列表 8.3\. 使用`SEMAPHORE`的`StockCommand`

```
public class StockCommand extends HystrixCommand<String> {
    private final String stockCode;

    public StockCommand(String stockCode) {
        super(Setter                                                   *1*

    .withGroupKey(HystrixCommandGroupKey.Factory.asKey("StockGroup"))
             .andCommandPropertiesDefaults(
                 HystrixCommandProperties.Setter()
                     .withExecutionIsolationStrategy(

    HystrixCommandProperties.ExecutionIsolationStrategy.SEMAPHORE      *2*
                      )
              )
        );

        this.stockCode = stockCode;
    }

    ...
}
```

+   ***1*** 使用 Setter 作为流畅接口来为 Hystrix 定义额外的配置

+   ***2*** 将执行隔离策略设置为 SEMAPHORE。

列表说明了如何为 Hystrix 设置额外的配置以自定义特定命令的行为。在实践中，你不会使用`SEMAPHORE`与`HystrixCommand`一起使用，因为它无法设置执行应该持续多长时间的超时。没有超时，如果消费的服务未能及时提供响应，你很容易发现自己陷入死锁系统。

#### 8.3.4\. 回退

目前，当你的断路器或舱壁不执行请求时，会返回一个错误响应。虽然这并不理想，但总比你的微服务处于等待超时状态要好。

如果你能提供一个简单的响应来代替失败，那岂不是很好？在某些情况下，可能确实无法为这些情况提供通用的响应，但通常这是可能的，并且是有益的。

在图 8.8 中，你可以看到在断路器和舱壁失败路径上的回退处理。如果你想要消费的微服务有一个已注册的回退处理器，它的响应会返回给你。如果没有，则返回原始错误。

##### 图 8.8\. 带有回退处理的故障缓解流程

![图片](img/08fig08_alt.jpg)

让我们看看如何为`StockCommand`实现回退处理器。

##### 列表 8.4\. 带有回退的`StockCommand`

```
public class StockCommand extends HystrixCommand<String> {
    ...

    @Override
    protected String getFallback() {                                   *1*
        // Return previous days cached stock price, no network call.
    }
}
```

+   ***1*** 覆盖默认的抛出失败异常的回退。

当你处理`HystrixObservableCommand`时，实现回退处理程序会有所不同，但并不太多。

##### 列表 8.5\. 带有回退的`StockObservableCommand`

```
public class StockObservableCommand extends HystrixObservableCommand<String> {
    ...

    @Override
    protected Observable<String> resumeWithFallback() {                 *1*
        // Return previous days cached stock price as an Observable,
no network call.
    }
}
```

+   ***1*** 返回 Observable<String>而不是 String 以匹配命令响应类型

#### 8.3.5\. 请求缓存

虽然它并不能直接减轻失败，但*请求缓存*可以通过减少你对其他微服务发出的请求数量来防止 bulkhead 和其他失败的发生。

它是如何做到这一点的呢？通过请求缓存，之前的请求及其响应可以被缓存，这样你就可以匹配未来的请求，并从缓存中返回响应。图 8.9 显示了请求缓存位于其他缓解策略之前，因为它减少了需要通过流程后续阶段的请求数量。

##### 图 8.9\. 带有请求缓存的失败缓解流程

![](img/08fig09_alt.jpg)

请求缓存提供了减少通过你的缓解流程的请求数量和提高返回响应速度的双重好处。启用请求缓存并不适用于所有情况，但在返回的数据根本不会改变或在你微服务完成任务期间不太可能改变的情况下是有益的。

这种解决方案特别适用于参考数据或用于检索用户账户等情况，因为它允许你的微服务在无需担心增加网络流量的情况下，根据需要多次调用外部微服务。这种方法还简化了微服务内部方法和服务接口，因为你不再需要在调用中传递数据以防止额外的调用。有了请求缓存，你就没有额外的调用风险。

要在 Hystrix 中启用请求缓存，你需要做两件事。首先，你需要激活`HystrixRequestContext`，这样你就有缓存响应的手段：

```
HystrixRequestContext context = HystrixRequestContext.initializeContext();
```

这个调用需要在执行任何 Hystrix 命令之前发生。对于我们的情况，你将在稍后看到的 JAX-RS 端点方法中进行第一次调用。其次，你需要定义用于缓存请求及其响应的键。

##### 列表 8.6\. 带有请求缓存的`StockCommand`

```
public class StockCommand extends HystrixCommand<String> {
    private final String stockCode;

    ...

    @Override
    protected String getCacheKey() {       *1*
        return this.stockCode;
    }
}
```

+   ***1*** 使用你在请求中使用的股票代码覆盖请求缓存的键

#### 8.3.6\. 将所有内容整合在一起

在你目前的流程中，你已经有了请求缓存、断路器、bulkhead 和回退。图 8.10 显示了它们在实际调用中的位置。

##### 图 8.10\. 整个失败缓解流程

![](img/08fig10_alt.jpg)

在这里，你添加“执行”来表示你正在调用外部服务。执行过程中遇到的任何失败或超时都会反馈到回退处理中，同时也提供失败数据到电路健康数据。然后，断路器使用这些信息来确定是否达到了错误阈值，电路应该切换到开启状态。

图 8.11 进一步展示了 Hystrix 如何在你的微服务、服务 A 和你要消费的服务 B 之间集成时提供这些功能。

##### 图 8.11\. 具有故障缓解的微服务调用

![图片](img/08fig11_alt.jpg)

当请求进入你的服务 A 方法或端点时，你创建一个请求并将其传递给 Hystrix。请求通过之前启用的任何检查，然后才在服务 B 上执行。服务 B 的响应返回到你的服务 A 方法，进行任何必要的处理，然后你为客户端构造响应。

如你所见，在许多点上，Hystrix 可以提供不同的或缓存的响应，而无需直接调用服务 B。这样的流程在直接减少失败的同时，也减少了导致失败的因素。一个例子是使用请求缓存来减少微服务负载。

虽然你已经看到了 Hystrix 如何实现这些故障缓解功能，但提供相同功能的其他库或框架应该以类似的方式运行。但其他库或框架实现所需缓解的方式可能差异很大。

#### 8.3.7\. Hystrix 仪表板

太棒了——你现在可以改善分布式架构中微服务的可靠性。但你怎么确定特定的微服务是否持续导致失败？或者你是否需要调整设置以减少错误并处理额外的负载？

听起来你需要一种方法来监控你的容错库的性能。恰好 Hystrix 提供了 SSEs（服务器端事件），提供了关于特定微服务的许多详细信息。你可以看到并分析一切——运行微服务的宿主数量、处理的请求、失败、超时等等。

Hystrix 还提供了一种可视化所有这些事件的方法：Hystrix 仪表板，如图 8.12 所示。Hystrix 仪表板提供了它从每个注册流接收到的 SSEs 的视觉表示。你将很快看到流是什么。

图 8.12 显示了`StockCommand`的信息。在如此小的 UI 中有很多数据点，但其中一些最重要的如下：

##### 图 8.12\. Hystrix 仪表板的单个电路

![图片](img/08fig12.jpg)

+   过去 10 秒内的错误百分比—100%

+   运行微服务的宿主数量—1

+   过去 10 秒内的成功请求—0

+   在过去 10 秒内被拒绝的短路请求—40

+   过去 10 秒内的失败—0

+   电路是开启还是关闭——开启


##### 提示

您可以在 [`github.com/Netflix/Hystrix/wiki/Dashboard`](https://github.com/Netflix/Hystrix/wiki/Dashboard) 找到每个电路的每个度量指标的详细信息。


让我们看看仪表板的实际运行情况。切换到 /hystrix-dashboard 目录并构建项目：

```
mvn clean package
```

然后运行仪表板：

```
java -jar target/hystrix-dashboard-thorntail.jar
```

启动仪表板后，打开浏览器并导航到 http://localhost:8090/。为了使仪表板能够可视化度量数据，它需要从您的断路器获取这些数据！对于单个电路，您可以通过将 http://localhost:8080/hystrix.stream 直接添加到主输入框中，来直接添加 SSE 流，如图 8.13 所示。点击添加流按钮，然后点击监控流。主页面将加载，但直到您启动微服务，流中不会接收到任何 SSE，因此可视化尚未出现。

##### 图 8.13\. Hystrix 仪表板主页

![](img/08fig13_alt.jpg)

切换到 /chapter8/stock-client 目录并启动微服务：

```
mvn thorntail:run
```

在另一个浏览器窗口中，您可以访问 http://localhost:8080/single/AAPL 以请求由代码 AAPL 表示的当前股票价格详情。在 URL 路径中可以使用任何有效的股票代码。

如果您刷新页面或以其他方式多次请求，您可以通过切换回 Hystrix 仪表板来查看您电路上的数据。

您的 stock-client 内置了处理特定 Hystrix 功能的机制。例如，每第十个请求将抛出异常返回给您的消费微服务，而每第二个请求将被暂停 10 秒以触发超时。这允许您看到失败如何在仪表板上表示。

要查看请求缓存的工作方式，您可以访问 http://localhost:8080/single/AAPL/4。注意在控制台中，只对外部服务发出了单个请求，并且浏览器收到的每个响应都有一个相同的请求编号。

要完全看到电路的实际运行情况，您需要多次调用该服务：

```
curl http://localhost:8080/single/AAPL/?[1-100]
```

这将连续对您的服务进行 100 次调用，允许您在看到请求进入时监控仪表板上的电路。您会注意到错误过多导致断路器打开的点。然后您会立即看到所有剩余的请求通过不调用微服务而是返回回退来短路。如果您在通过浏览器访问服务之前等待几秒钟，然后您会看到断路器尝试请求，成功，并再次切换到关闭状态。

在 `StockCommand` 中调整设置以查看电路行为如何变化。书中示例代码中的一个例子是将 `StockCommand` 修改为设置可用于消费微服务的线程数量。

##### 列表 8.7\. 带有线程配置的 `StockCommand`

```
super(Setter
    .withGroupKey(HystrixCommandGroupKey.Factory.asKey("StockGroup"))
    .andCommandPropertiesDefaults(
            HystrixCommandProperties.Setter()
                    .withCircuitBreakerRequestVolumeThreshold(10)
                    .withCircuitBreakerSleepWindowInMilliseconds(10000)
                    .withCircuitBreakerErrorThresholdPercentage(50)
    )
    .andThreadPoolPropertiesDefaults(
            HystrixThreadPoolProperties.Setter()
                    .withCoreSize(1)                *1*
    )
);
```

+   ***1*** 指定必须使用单个线程

使用`StockCommand`的列表 8.7 构造函数重新运行你的测试，显示请求被`ThreadPool`拒绝。

在查看 Hystrix 仪表板后，我们应该都认识到这样的工具在我们的工具箱中是多么关键。将 Hystrix 添加到你的外部调用中为这些执行提供了容错级别，但它并不是万无一失的。你需要持续实时监控你的微服务，以跟踪即将出现的问题并观察可以通过调整断路器设置来解决的问题。

如果你没有充分利用 Hystrix 仪表板提供的内容，尤其是在实时监控方面，你将无法获得使用容错库的所有好处。

### 8.4\. 将 Hystrix 添加到你的支付微服务

你已经看到了如何实现 Hystrix 以及从仪表板查看其指标。你的 Stripe 微服务并不非常可靠，所以让我们在支付中使用 Hystrix 来确保你不会过度受到其失败或超时的负面影响！

前面的章节已经介绍了 Hystrix 提供的各种功能，以帮助进行故障缓解。当将 Hystrix 添加到支付时，你将充分利用 Hystrix 提供的完整流程。

在接下来的每个部分中，你需要你的 Stripe 微服务正在运行，所以让我们现在开始。首先，你需要确保 Minishift 环境正在运行，并且你已经使用 OpenShift 客户端登录。然后切换到/chapter8/stripe 目录并运行以下命令：

```
mvn clean fabric8:deploy -Popenshift -DskipTests
```

#### 8.4.1\. 使用 RESTEasy 客户端的 Hystrix

让我们修改第七章中的 Payment，使用`HystrixCommand`与 Stripe 交互。

##### 列表 8.8\. `StripeCommand`

```
public class StripeCommand extends HystrixCommand<ChargeResponse> {
    private URI serviceURI;

    private final ChargeRequest chargeRequest;

    public StripeCommand(URI serviceURI, ChargeRequest chargeRequest) {    *1*
        super(Setter
          .withGroupKey(HystrixCommandGroupKey.Factory.asKey("StripeGroup"))
          .andCommandPropertiesDefaults(
              HystrixCommandProperties.Setter()
                  .withCircuitBreakerRequestVolumeThreshold(10)
                  .withCircuitBreakerSleepWindowInMilliseconds(10000)
                  .withCircuitBreakerErrorThresholdPercentage(50)
          )
        );

        this.serviceURI = serviceURI;
        this.chargeRequest = chargeRequest;
    }

    public StripeCommand(URI serviceURI,                                   *2*
            ChargeRequest chargeRequest, HystrixCommandProperties.Setter
commandProperties) {
        super(Setter

     .withGroupKey(HystrixCommandGroupKey.Factory.asKey("StripeGroup"))
                      .andCommandPropertiesDefaults(commandProperties)
        );

        this.serviceURI = serviceURI;
        this.chargeRequest = chargeRequest;
    }

    @Override
    protected ChargeResponse run() throws Exception {                      *3*
        ResteasyClient client = new ResteasyClientBuilder().build();
        ResteasyWebTarget target = client.target(serviceURI);

        StripeService stripeService = target.proxy(StripeService.class);
        return stripeService.charge(chargeRequest);
    }

    @Override
    protected ChargeResponse getFallback() {
        return new ChargeResponse();                                       *4*
    }
}
```

+   ***1*** 将 Stripe URL 和 ChargeRequest 传递到命令中并设置属性。

+   ***2*** 允许调用者设置 Hystrix 属性的重载构造函数

+   ***3*** 等同于第七章中`PaymentServiceResource`的方法，因为不再在 JAX-RS Resource 中进行调用

+   ***4*** 如果有问题，回退到空的 ChargeResponse。

现在你已经有了你的`StripeCommand`，`PaymentServiceResource`与第七章相比有何不同？

##### 列表 8.9\. `PaymentServiceResource`

```
@Path("/")
@ApplicationScoped
public class PaymentServiceResource {
    ....

    @POST
    @Path("/sync")
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON)
    @Transactional
    public PaymentResponse chargeSync(PaymentRequest paymentRequest) throws
Exception {
        Payment payment = setupPayment(paymentRequest);
        ChargeResponse response = new ChargeResponse();

        try {
            URI url = getService("chapter8-stripe");

            StripeCommand stripeCommand = new StripeCommand(               *1*
                url,
                paymentRequest.getStripeRequest(),
                HystrixCommandProperties.Setter()
                    .withExecutionIsolationStrategy(
        HystrixCommandProperties.ExecutionIsolationStrategy.SEMAPHORE
                       )
                      .withExecutionIsolationSemaphoreMaxConcurrentRequests(1)
                      .withCircuitBreakerRequestVolumeThreshold(5)
            );

            response = stripeCommand.execute();                            *2*
            payment.chargeId(response.getChargeId());
        } catch (Exception e) {
            payment.chargeStatus(ChargeStatus.FAILED);
        }

        em.persist(payment);
        return PaymentResponse.newInstance(payment, response);
    }

    @POST
    @Path("/async")
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON)
    public void chargeAsync(@Suspended final AsyncResponse asyncResponse,
            PaymentRequest paymentRequest) throws Exception {
        Payment payment = setupPayment(paymentRequest);

        URI url = getService("chapter8-stripe");
        StripeCommand stripeCommand =
            new StripeCommand(url, paymentRequest.getStripeRequest());     *3*

        stripeCommand
            .toObservable()                                                *4*
            .subscribe(                                                    *5*
                (result) -> {
                    payment.chargeId(result.getChargeId());
                    storePayment(payment);
                    asyncResponse.resume(PaymentResponse.newInstance(payment,
    result));
                },
                (error) -> {
                    payment.chargeStatus(ChargeStatus.FAILED);
                    storePayment(payment);
                    asyncResponse.resume(error);
                }
            );
    }
    ....
}
```

+   ***1*** 实例化命令并设置 Hystrix 属性。

+   ***2*** 在命令执行()上阻塞。

+   ***3*** 使用默认的 Hystrix 属性实例化命令。

+   ***4*** 获取命令的 Observable。

+   ***5*** 订阅 Observable，传递成功和失败方法。

你的`PaymentServiceResource`已经表明，当期望只有一个响应时，你可以轻松地在同步和异步执行模式之间切换，使用相同的`HystrixCommand`实现。

从你的第七章版本到这个版本，重构并不多，主要是将消耗外部微服务的代码提取到一个新的方法和类`StripeCommand`中。

现在您已经重构了资源，让我们运行它！切换到 /chapter8/resteasy-client 目录并运行以下命令：

```
mvn clean fabric8:deploy -Popenshift
```

如果 Hystrix Dashboard 仍在运行，请返回主页以便您可以添加新的流。如果它尚未运行，请像本章早期那样重新启动它。

从 OpenShift 控制台复制 chapter8-resteasy-client 的 URL，将其粘贴到 Hystrix Dashboard 主页上的文本框中，并添加 hystrix.stream 作为 URL 后缀。点击添加流然后监控流。

由于您尚未发出任何请求，Hystrix Dashboard 不会立即显示任何内容。为了练习 Payment 服务，您可以执行单个请求或多个请求，后者在仪表板中更容易看到结果，尤其是如果它们的执行可以自动化。

使用之前章节 8.4-resteasy-client 的 URL，您可以访问服务的同步版本 (`/sync`) 或异步版本 (`/async`)。在任一或两个端点启动一系列请求后，Hystrix Dashboard 将显示已成功和失败的请求的所有详细信息。

#### 8.4.2\. 使用 Ribbon 客户端的 Hystrix

您的 RESTEasy 客户端需要一点修改以添加 Hystrix 支持。现在您将查看 Ribbon 客户端所需的操作。

首先，您需要更新 Stripe 微服务的接口定义，以便利用 Hystrix 注解与 Ribbon 结合。

##### 列表 8.10\. `StripeService`

```
@ResourceGroup(name = "chapter8-stripe")
public interface StripeService {

    StripeService INSTANCE = Ribbon.from(StripeService.class);

    @TemplateName("charge")
    @Http(
            method = Http.HttpMethod.POST,
            uri = "/stripe/charge",
            headers = {
                    @Http.Header(
                            name = "Content-Type",
                            value = "application/json"
                    )
            }
    )
    @Hystrix(                                                       *1*
            fallbackHandler = StripeServiceFallbackHandler.class
    )
    @ContentTransformerClass(ChargeTransformer.class)
    RibbonRequest<ByteBuf> charge(@Content ChargeRequest chargeRequest);
}
```

+   ***1*** 将 Hystrix 功能添加到您的 Ribbon HTTP 请求中，并带有回退处理程序

这很简单——只需几行额外的代码！


##### 注意

Hystrix 注解仅可用于与 Netflix Ribbon 结合使用。


目前，代码无法编译，因为您没有回退处理程序的类。让我们添加它。

##### 列表 8.11\. `StripeServiceFallbackHandler`

```
public class StripeServiceFallbackHandler implements FallbackHandler<ByteBuf> {
    @Override
    public Observable<ByteBuf> getFallback(                              *1*
        HystrixInvokableInfo<?> hystrixInfo,
        Map<String, Object> requestProperties) {

        ChargeResponse response = new ChargeResponse();
        byte[] bytes = new byte[0];
        try {
            bytes = new ObjectMapper().writeValueAsBytes(response);      *2*
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
        ByteBuf byteBuf =
     UnpooledByteBufAllocator.DEFAULT.buffer(bytes.length);
        byteBuf.writeBytes(bytes);                                       *3*
        return Observable.just(byteBuf);                                 *4*
    }
}
```

+   ***1*** 实现 getFallback() 以在回退情况下返回您选择的任何内容。

+   ***2*** 创建一个空的 ChargeResponse 用于回退，并将其转换为 byte[]。

+   ***3*** 将 byte[] 写入您在上一个步骤中创建的 ByteBuf。

+   ***4*** 创建一个 Observable，它返回作为单个结果的 ByteBuf 内容。

您还需要更新 `PaymentServiceResource`，来自 第七章。但不是这样！使用 Hystrix 与 Ribbon 结合使用注解的一个优点是，您从 第七章 的 `PaymentServiceResource` 完全不需要更改。一个很大的优点是，您可以在使用 Ribbon 的现有微服务中轻松添加 Hystrix 而无需重构。只需添加一个额外的注解，如果需要，还可以添加一个回退处理程序。

是时候运行它了！切换到 /chapter8/ribbon-client 目录并运行以下命令：

```
mvn clean fabric8:deploy -Popenshift
```

就像 RESTEasy 客户端示例一样，您可以通过浏览器打开并访问服务的/sync 或/async URL，使用 OpenShift 控制台中的服务基本 URL。然后，您可以更新 Hystrix 仪表板以使用此新流，执行一些请求，并查看仪表板如何变化。

就像您部署到 Minishift 的其他示例一样，完成后，您需要取消部署以释放资源：

```
mvn fabric8:undeploy -Popenshift
```

### 摘要

+   当考虑部署到分布式架构时，延迟和容错性很重要，因为它可能会对微服务的吞吐量和速度产生不利影响。

+   您消费微服务的代码可以用 Hystrix 包装，以集成容错特性，如回退、请求缓存和隔离舱。

+   Hystrix 本身并不是最高容错性的万能药。通过像 Hystrix 仪表板这样的工具进行实时监控对于成功提高整体容错性至关重要。

## 第九章. 保护微服务

*本章涵盖*

+   理解为什么你需要安全的微服务

+   保护微服务

+   消费一个受保护的微服务

+   从 UI 与受保护的微服务交互

在本章中，您将通过向它们添加各种类型的安全性来扩展先前的示例。首先，您将了解在设计和发展微服务时可能需要考虑的不同类型的安全性。

### 9.1\. 保护微服务的重要性

保护微服务是一项至关重要的任务，需要在开发初期就考虑。如果不这样做，后期集成安全性的开发时间会更长。为什么？不设计安全性会导致代码可能需要在以后进行重大重构才能实现。

尽管在典型企业 Java 应用程序的开发之前不考虑安全性可能会轻易增加几个月的开发时间，但至少在微服务中，您通常需要重构的代码要少得多。即便如此，不是最好一开始就设计安全性并节省时间吗？

#### 9.1.1\. 为什么安全性很重要？

作为企业开发者，我们经常被要求开发各种应用程序，应用程序的最终用户可能是内部或外部，有时两者都是。图 9.1 展示了一组内部用户使用的微服务。

##### 图 9.1\. 内部用户

![](img/09fig01_alt.jpg)

有这些要求，你就可以忽略安全性，对吧？错！

即使您只为内部用户开发微服务，您能保证围绕您的微服务的安全性会保持吗？如果或当任何防止外部网络入侵的安全屏障被破坏时会发生什么？

图 9.2 展示了如果网络安全被破坏，恶意用户（网络外部）将如何无限制地访问微服务。

##### 图 9.2\. 恶意外部用户

![](img/09fig02.jpg)

安全是一个不应该被理所当然地认为存在的功能，无论可能实施哪些预防措施。一个常见的误解是安全是无可挑剔的，这当然不是事实。

再次查看图 9.2，如果你不认为内部网络是安全的，你更倾向于在自己的微服务中添加额外的安全措施，以防止未经授权的访问。如果纯粹用于内部目的的每个应用程序或微服务都不包括自己的安全预防措施，那么你已将外部网络边界的安全性变成了一个单点故障。

这还不包括你可能在内部网络中有一个恶意用户的情况，如图 9.3 所示！尽管内部恶意用户可能并不常见，但这种情况不能被忽视。这种情况可能由许多原因引起：不满的员工或企业间谍活动最有可能。

几乎没有开发的应用程序不需要安全。这些应用程序大多仅限于提供对公众已经可用的只读数据的读取。

##### 图 9.3. 恶意内部用户

![图片](img/09fig03_alt.jpg)

这是一个相当狭窄的应用定义，你可以忽略安全。你的企业每天在建造多少这样的应用？可能没有！企业在其整个生命周期中开发或将要开发的应用程序类型，数量会非常少。静态数据也是公开可用的，这不会引起企业的兴趣。

所有这些意味着什么？这意味着没有任何应用程序或微服务可以完全忽视安全，任何时候都不可以。

#### 9.1.2. 安全需要解决哪些问题？

既然你知道你需要安全，你需要解决哪些问题？这本身可能就是一本书的主题！因为你不是要为微服务重新创作《战争与和平》，所以你会关注那些最有兴趣的领域。

认证和授权是我们认为与微服务最相关的两个安全方面。在你深入探讨之前，你需要概述每个这些术语的含义。

*认证*在图 9.1、9.2 和 9.3 中展示。它仅涉及用户是否有权访问应用程序或微服务。无论该应用程序或微服务可能托管在哪里，或者用户是否属于企业或外部，认证纯粹关注用户能否访问应用程序。

如果微服务不需要区分用户是否允许访问，那么认证就是所需的全部。但如果经过认证的用户需要访问应用程序或微服务的不同部分，那么你还需要*授权*。

图 9.4 提供了一个用于微服务授权的用户角色示例。

##### 图 9.4. 多个用户角色用于授权

![](img/09fig04_alt.jpg)

你可以看到 Admin、Manager 和 User 这些角色，这些都是可能需要的典型角色。对于你的微服务可能需要的角色可能会有所不同，可能从零到多个，这取决于需求。

企业也可能拥有微服务，如图 9.5 所示。在这种情况下，你有一个由内部用户管理的微服务，其角色为 Admin。但微服务的用户对企业来说是外部的。

##### 图 9.5. 内部和外部用户角色

![](img/09fig05.jpg)

从整个可能包含许多微服务的应用程序的角度来看，你通常需要认证和授权的混合来满足安全需求。对于应用程序中的单个微服务，你可能只需要关注用户请求的认证，而无需更多。

无论你的微服务可能需要什么——无论是认证、授权还是两者兼而有之——在设计阶段就需要考虑安全性，以确保它不会成为最后一刻的担忧。

那么你是如何将安全性添加到你的微服务中的呢？你当然可以开发自己的安全解决方案，但在许多情况下这并不理想。你需要花费时间开发、维护等等。开发自己的安全解决方案不仅会导致你想要开发的微服务开发延迟，而且还会给未来的开发者增加额外的维护负担。

你想要做的是利用一个由大量开发者开发和维护的稳定项目，该项目提供了你需要的处理安全用例。尽管你可能有很多这样的项目选项，但在这本书中，我们将选择 Keycloak。

### 9.2. 使用 Keycloak

Keycloak 是一个开源项目，为现代应用程序和服务提供身份和访问管理。向应用程序添加认证和确保服务安全可以以最小的麻烦实现。

#### 9.2.1. 理解 Keycloak 的功能

Keycloak 提供了许多功能。以下是四个与微服务开发最相关的功能：

+   ***单点登录*——** 允许用户对 Keycloak 进行认证，而不是对每个单独的应用程序或服务进行认证。用户登录 Keycloak 后，可以访问任何通过 Keycloak 进行认证的应用程序或服务。

+   ***社交登录*——** 使用 Keycloak 启用社交登录非常简单！在管理控制台中配置社交网络即可。无需代码或应用程序更改。

+   ***用户联合*—** 如果您的用户在 LDAP 或 Active Directory 中注册，他们可以很容易地与 Keycloak 联合。如果用户存储在不同的类型中，例如关系数据库，您还可以开发自己的提供程序来访问您的用户。

+   ***标准协议*—** 默认情况下，Keycloak 提供了对 OpenID Connect、OAuth 2.0 和安全断言标记语言(SAML)的支持。

关于 Keycloak 及其所有功能的详细信息可以在其网站上找到，[www.keycloak.org](http://www.keycloak.org)。

#### 9.2.2\. 设置 Keycloak

您需要做的第一件事是为您的微服务和应用程序下载 Keycloak 服务器以进行集成。就我们的目的而言，您有两种方法可以做到这一点。您可以下载一个为 Keycloak 定制的完整 WildFly 发行版，或者下载一个使用 Thorntail 构建的 Keycloak 服务器。为了保持微服务的方式，请选择 Thorntail 版本。我们示例所需的版本可以从[`mng.bz/s6r9`](http://mng.bz/s6r9)下载。

下载后，在单独的端口上启动此版本，以免干扰您自己的微服务：

```
java -Dswarm.http.port=9090 -jar keycloak-2018.1.0-swarm.jar
```

当服务器启动后，在浏览器中导航到 http://localhost:9090/auth/。您将看到一个类似于图 9.6 的屏幕。

##### 图 9.6\. 设置 Keycloak 管理员用户

![](img/09fig06_alt.jpg)

在 Keycloak 服务器上输入管理员账户的用户名和密码。然后点击创建。接下来，点击管理控制台链接以查看图 9.7 中的登录屏幕。

##### 图 9.7\. 登录 Keycloak 管理控制台

![](img/09fig07_alt.jpg)

输入您在设置管理员账户时提供的凭据，然后点击登录按钮。

图 9.8 显示了 Keycloak 管理控制台的主屏幕。从这里，您可以修改和调整 Keycloak 的所有部分以满足您的需求。默认情况下，您会获得一个主域。

##### 图 9.8\. Keycloak 管理控制台

![](img/09fig08_alt.jpg)

由于主域包含管理员用户，因此对于使用应用程序或微服务进行身份验证的用户，不使用此域是一种良好的做法。

| |
| --- |

**Keycloak 域**

Keycloak *域*管理一组用户，包括他们的凭据、角色和组。域彼此隔离，只负责管理他们关联的用户。

域提供了一种将用户分组分离用于不同目的的方法。您可能有一个用于财务微服务的域，另一个用于人员管理微服务的域。这种分离确保了来自每个域的用户保持分离，但由单个 Keycloak 实例进行管理。

| |
| --- |

根据您的需求，Keycloak 足够灵活，可以处理应用程序或微服务所需的任何情况。对于典型的应用程序开发，尽管对于微服务仍然相关，但通常需要认证用户并在调用服务时使用其凭证。

图 9.9 展示了在 UI 内部对用户进行认证的请求路径。

##### 图 9.9\. 通过 UI 进行用户认证

![](img/09fig09_alt.jpg)

认证步骤如下：

1.  用户请求登录应用程序 UI。

1.  UI 重定向到 Keycloak 进行登录。Keycloak 返回可用于发出认证请求的令牌。

1.  用户选择加载需要认证的视图。

1.  Keycloak 提供的 *bearer 令牌* 被添加到请求的 HTTP 头部。

1.  从请求中提取令牌并将其传递给 Keycloak 进行验证。如果令牌有效，受保护的微服务能够处理请求。如果令牌无效，则返回 HTTP 401 状态码，表示未经授权的用户发出了请求。

| |
| --- |

##### 定义

*bearer 令牌* 是一种具有特殊行为属性的安全令牌。任何持有令牌的实体都可以像持有相同令牌的其他实体一样使用它。使用 bearer 令牌不需要持有人证明对加密密钥的占有。

| |
| --- |

在前面的过程中，有一个微服务对自身进行认证以向受保护的微服务发出请求的微小变化。图 9.10 说明了这种变化。

##### 图 9.10\. 微服务认证

![](img/09fig10.jpg)

这个过程的不同之处在于，调用受保护微服务的任何调用者都不包含或接收来自用户的认证令牌：

1.  请求被一个未受保护的微服务接收。

1.  未受保护的微服务对 Keycloak 进行自身认证。

1.  认证令牌通过请求的 HTTP 头部传递给受保护的微服务。

1.  从请求中提取令牌并将其传递给 Keycloak 进行验证。如果令牌有效，受保护的微服务可以处理请求。如果令牌无效，则返回 HTTP 401 状态码，表示未经授权的用户发出了请求。

本章的剩余部分将提供这两个场景的示例。让我们看看如何使用 Keycloak 保护一些微服务。

### 9.3\. 保护 Stripe 微服务

在本节中，您将了解如何在图 9.10 的场景中实现认证。来自第八章的 Stripe 和 Payment 微服务将采用与图 9.10 类似的安保措施。Payment 微服务将基于该章节中的 RESTEasy 客户端版本。让我们回顾一下之前的场景，这次是 Stripe 和 Payment；请参见图 9.11。

##### 图 9.11\. 带有 Stripe 和 Payment 的微服务认证

![](img/09fig11.jpg)

#### 9.3.1\. 配置 Keycloak

在您的 Keycloak 服务器运行后，下一步是为您的微服务定义一个区域。

登录到管理控制台后，将鼠标悬停在左上角的 Master 区域名称上，以显示添加区域按钮，如图 9.12 所示。

##### 图 9.12\. 在 Keycloak 中访问添加区域按钮

![](img/09fig12.jpg)

点击添加区域按钮，打开用于创建区域的屏幕。图 9.13 显示此屏幕。

##### 图 9.13\. 创建区域

![](img/09fig13_alt.jpg)

点击选择文件选项，从书籍代码存储库的 /chapter9/keycloak 目录中定位 cayambe-realm.json。然后点击打开。

图 9.14 显示您将在 Keycloak 中创建的区域。要执行导入，您需要点击创建，以便 cayambe-realm.json 的内容被导入，并存在一个 Cayambe 区域。

##### 图 9.14\. 导入 Cayambe 区域

![](img/09fig14_alt.jpg)

使用 Cayambe 区域，您正在利用 Keycloak 服务账户功能。此功能允许客户端通过 Keycloak 进行身份验证，而无需用户任何交互。此功能对于直接由用户触发的管理任务非常有用，例如需要身份验证的预定作业。

现在您的区域已创建，让我们看看您导入的 JSON 的部分，以便您可以看到 Keycloak 设置了什么。

##### 列表 9.1\. cayambe-realm.json

```
  "realm": "cayambe",                                            *1*
  "enabled": true,                                               *2*
  ...
  "users": [
    {
      "username": "service-account-payment-service",             *3*
      "enabled": true,
      "serviceAccountClientId": "payment-authz-service",         *4*
      "realmRoles": [
        "stripe-service-access"                                  *5*
      ]
    }
  ],
  "roles": {
    "realm": [
      {
        "name": "stripe-service-access",                         *6*
        "description": "Stripe service access privileges"
      }
    ]
  },
  "clients": [
    {
      "clientId": "payment-authz-service",                       *7*
      "secret": "secret",                                        *8*
      "enabled": true,
      "standardFlowEnabled": false,
      "serviceAccountsEnabled": true                             *9*
    },
    {
      "clientId": "stripe-service",                              *10*
      "enabled": true,
      "bearerOnly": true                                         *11*
    }
  ]
```

+   ***1*** 指定区域名称为 cayambe

+   ***2*** 确保在加载后您的区域已启用

+   ***3*** 服务账户用户的唯一用户名

+   ***4*** 定义将使用服务账户进行身份验证的 clientId

+   ***5*** 应分配给服务账户用户的角色。

+   ***6*** 定义 stripe-service-access 区域角色

+   ***7*** 为您的支付微服务分配的唯一 clientId。

+   ***8*** 用于身份验证服务账户用户的密钥

+   ***9*** 为客户端启用 Keycloak 的服务账户功能

+   ***10*** 将被保护的 Stripe 微服务的客户端 ID。

+   ***11*** 标识客户端仅验证 bearer 令牌，但无法检索它们

您在此处定义的所有名称和 ID 在您创建的区域中都是唯一的，但它们本身没有意义。它们只是文本。

重要的是，区域中某个服务的客户端 ID 与服务配置中的规范相匹配（下一节将介绍）。有了这个，您的 Keycloak 服务器就准备好处理 Stripe 和支付的认证了。

#### 9.3.2\. 保护 Stripe 资源

第一步是保护 Stripe 微服务，以确保您在访问 Stripe API 时没有适当的身份验证。一旦您知道您已正确连接到服务，您将添加必要的身份验证。

如果您从 第八章 中的代码开始，您不需要修改 `StripeResource` 来添加安全性。非常酷，对吧？您可以在不修改其代码的情况下向现有的 RESTful 端点添加安全性！这是怎么做到的？

立即让 Maven 知道您想要使用 Keycloak 与您的 Thorntail 微服务一起使用。为此，您需要在您的 pom.xml 中添加一个依赖项：

```
    <dependency>
      <groupId>io.thorntail</groupId>
      <artifactId>keycloak</artifactId>
    </dependency>
```

唯一的其他任务是定义 Keycloak 的位置、其配置以及需要保护的内容。幸运的是，您可以使用 Thorntail 在一个文件中完成所有这些操作！您将一个 project-defaults.yml 文件添加到 Stripe 微服务的 src/main/resources 目录中，其内容如下所示。

##### 列表 9.2\. project-defaults.yml

```
swarm:
  keycloak:
    secure-deployments:
      chapter9-stripe.war:                                  *1*
        realm: cayambe                                      *2*
        bearer-only: true                                   *3*
        auth-server-url: http://192.168.1.13:9090/auth      *4*
        ssl-required: external
        resource: stripe-service                            *5*
        enable-cors: true
  deployment:
    chapter9-stripe.war:                                    *6*
      web:
        security-constraints:
          - url-pattern: /stripe/charge/*                   *7*
            roles: [ stripe-service-access ]                *8*
```

+   ***1*** 定义 chapter9-stripe.war 部署的 Keycloak 配置章节

+   ***2*** 您的部署用于身份验证的领域——在本例中为 cayambe

+   ***3*** 将您的微服务标识为仅带 bearer。

+   ***4*** 服务域所在的服务器上的 Keycloak 服务器 URL。当服务部署到 Minishift 时，您不使用 localhost。

+   ***5*** 将此资源标识为 stripe-service，它对应于 cayambe-realm.json 中的客户端 ID。

+   ***6*** 定义 chapter9-stripe.war 部署特定配置的章节。这相当于可以在 web.xml 中提供的内容。

+   ***7*** 保护来自此微服务的 /stripe/charge URL 模式。

+   ***8*** 只有具有 stripe-service-access 角色的用户才能成功执行对这一微服务的请求。

现在您的 Stripe 微服务已经从未经身份验证的访问中得到了保护！让我们试一试。切换到 /chapter9/serviceauth/stripe 目录并运行以下命令：

```
mvn thorntail:run
```

尝试打开浏览器到 http://localhost:8080/stripe/charge，它将指示 *未授权*。从浏览器发送不带 bearer 令牌的 HTTP 请求会导致您的请求被拒绝，因为您没有进行适当的身份验证。

要查看更多细节，您可以使用显示 HTTP 网络调用的浏览器插件，或者从终端使用 `curl`。

##### 列表 9.3\. `curl` 的 Stripe 输出

```
$ curl -v http://localhost:8080/stripe/charge

*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> GET /stripe/charge HTTP/1.1                                            *1*
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 401 Unauthorized                                              *2*
< Expires: 0
< Connection: keep-alive
< WWW-Authenticate: Bearer realm="cayambe"
< Cache-Control: no-cache, no-store, must-revalidate
< Pragma: no-cache
< Content-Type: text/html;charset=UTF-8
< Content-Length: 71
< Date: Sun, 25 Feb 2018 03:22:53 GMT
<
* Connection #0 to host localhost left intact
<html><head><title>Error</title></head><body>Unauthorized</body></html>  *3*
```

+   ***1*** HTTP 请求头

+   ***2*** HTTP 响应头

+   ***3*** HTTP 响应体

现在更容易看出您收到了一个 401 HTTP 响应代码，表示您尝试未经授权访问 URL。现在 Stripe 已经得到了适当的保护，另一个微服务如何在不接收用户凭证的情况下访问它？

您也可以按照以下方式将 Stripe 部署到 Minishift：

```
mvn clean fabric8:deploy -Popenshift
```

#### 9.3.3\. 在支付资源中进行身份验证

本章的支付微服务是从 第八章 中的 RESTEasy 客户端派生出来的。您只需进行一些小的修改，就可以使其对 Keycloak 进行身份验证。

要能够使用 Keycloak 对支付进行身份验证，您需要向 Keycloak Authz 客户端添加一个依赖项：

```
<dependency>
  <groupId>org.keycloak</groupId>
  <artifactId>keycloak-authz-client</artifactId>
  <version>3.4.0.Final</version>
</dependency>
```

这个依赖项提供了你需要用于通过 Keycloak 进行身份验证的所有实用类。现在你需要定义你正在与之交互的 Keycloak，以及哪个 Payment 微服务位于 Cayambe 领域内。为此，你需要在 src/main/resources 目录下创建一个 keycloak.json 文件。

##### 列表 9.4\. Payment 服务的 keycloak.json

```
{
  "realm": "cayambe",                                   *1*
  "auth-server-url": "http://192.168.1.13:9090/auth",   *2*
  "resource": "payment-authz-service",                  *3*
  "credentials": {
    "secret": "secret"                                  *4*
  }
}
```

+   ***1*** 你的部署用于身份验证的领域——在本例中，为 cayambe。

+   ***2*** cayambe 领域所在的 Keycloak 服务器的 URL。

+   ***3*** 将此资源标识为 payment-authz-service，它对应于 cayambe-realm.json 中的 Client ID。

+   ***4*** 需要传递给 Keycloak 以验证此客户端的凭证。

这就是你需要做的所有配置。接下来，添加用于通过 Keycloak 进行身份验证的代码。因为你现在使用 Hystrix，你需要将身份验证处理添加到`StripeCommand`中。

##### 列表 9.5\. `StripeCommand`—`getAuthzClient`方法

```
private AuthzClient getAuthzClient() {                                   *1*
    if (this.authzClient == null) {                                      *2*
        try {
            this.authzClient = AuthzClient.create();                     *3*
        } catch (Exception e) {
            throw new RuntimeException("Could not create authorization
client.", e);
        }
    }

    return this.authzClient;
}
```

+   ***1*** 添加一个用于检索 Keycloak AuthzClient 的辅助方法。

+   ***2*** 如果你还没有创建 AuthzClient，请继续。

+   ***3*** 创建 AuthzClient，它使用 keycloak.json 中的信息进行身份验证。

在拥有`AuthzClient`之后，你现在可以检索一个访问令牌，你可以将其添加到你对 Stripe 发出的任何请求中。为此，你必须修改`StripeCommand`中的`run()`方法，在你获得`Resteasy-Client`实例后添加一个请求过滤器。

##### 列表 9.6\. `StripeCommand`—`run`方法

```
protected ChargeResponse run() throws Exception {
    ResteasyClient client = new ResteasyClientBuilder().build();

    client.register((ClientRequestFilter) clientRequestContext -> {        *1*
        List<Object> list = new ArrayList<>();
        list.add("Bearer " +
     getAuthzClient().obtainAccessToken().getToken());                     *2*
        clientRequestContext.getHeaders().put(HttpHeaders.AUTHORIZATION,
list);                                                                   *3*
    });

    ResteasyWebTarget target = client.target(serviceURI);

    StripeService stripeService = target.proxy(StripeService.class);
    return stripeService.charge(chargeRequest);
}
```

+   ***1*** 注册一个匿名 ClientRequestFilter 以修改 HTTP 请求。

+   ***2*** 使用 AuthzClient 从 Keycloak 检索一个访问令牌，将 Bearer 前缀添加到令牌中，并将其添加到列表中。

+   ***3*** 将你创建的列表添加到请求的 AUTHORIZATION HTTP 头部。

你只需做这些就能在向 Stripe 发出的任何请求上传递一个 bearer 令牌。很简单，对吧？

#### 9.3.4\. 测试你的受保护微服务

现在你已经设置了 Stripe 和 Payment，是时候查看所有正在运行并相互交互的服务了。如果你还没有启动 Keycloak 服务器和 Stripe，请再次启动它们，确保将 Stripe 部署到 Minishift。

然后，你需要通过切换到/chapter9/serviceauth/payment-service 目录并运行以下命令来启动 Payment：

```
mvn clean fabric8:deploy -Popenshift
```

打开 OpenShift 控制台以检索 Payment 的 URL。然后使用你在第七章和第八章中使用的相同工具对/sync 和/async 端点执行 HTTP POST 操作。如果你直接尝试访问 Stripe 微服务，你仍然会收到 HTTP 响应代码 401，表示你未授权。

要查看从支付功能调用 Stripe 时的 HTTP 头部信息，你需要拦截请求或找到其他方式来输出它。在这种情况下，你需要修改 Stripe 以直接输出 HTTP 请求和响应头部。

让我们在/chapter9/serviceauth/stripe 的 project-defaults.yml 中取消注释以下内容：

```
  undertow:
    servers:
      default-server:
        hosts:
          default-host:
            filter-refs:
              request-dumper:
    filter-configuration:
      custom-filters:
        request-dumper:
          class-name: io.undertow.server.handlers.RequestDumpingHandler
          module: io.undertow.core
```

重启 Stripe，然后在 Payment 上发出另一个 HTTP POST 请求。在 OpenShift 控制台中，找到 Stripe 服务条目，点击 pod 状态右侧的三个点。从那里，选择查看日志，您应该会看到 Stripe 的输出，如下所示：

```
----------------------------REQUEST---------------------------
               URI=/stripe/charge
 characterEncoding=null
     contentLength=63
       contentType=[application/json]
            header=Accept=application/json
            header=Connection=Keep-Alive
            header=Authorization=Bearer

       eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJCTTRFT3FlZXU1bGowaWZw
       cHR0aWtEejdnakhsNzBjd2hreGY4c05

       NWU1NIn0.eyJqdGkiOiJmNDIyNmJlYS1hNWE2LTQ0NDgtOTBiZS1kNmI4NGUwY2FlOWUiLCJ
       leHAiOjE1MTk1MzI0MTksIm5iZiI6MC

       wiaWF0IjoxNTE5NTMyMzU5LCJpc3MiOiJodHRwOi8vMTkyLjE2OC4xLjEzOjkwOTAvYXV0aC
       9yZWFsbXMvY2F5YW1iZSIsImF1ZCI6I

       nBheW1lbnQtYXV0aHotc2VydmljZSIsInN1YiI6IjljZjAyOTQ5LTgxMzctNGM1Ny04MTY4L
       TVhMzlhMDczMTRlMCIsInR5cCI6IkJl

       YXJlciIsImF6cCI6InBheW1lbnQtYXV0aHotc2VydmljZSIsImF1dGhfdGltZSI6MCwic2Vz
       c2lvbl9zdGF0ZSI6IjI5MGM3MTJiLTJ

       kMzItNGZjMi05YWJjLTIxOGFlNTk2MjQwMiIsImFjciI6IjEiLCJhbGxvd2VkLW9yaWdpbnM
       iOltdLCJyZWFsbV9hY2Nlc3MiOnsicm

       9sZXMiOlsic3RyaXBlLXNlcnZpY2UtYWNjZXNzIl19LCJyZXNvdXJjZV9hY2Nlc3MiOnt9LC
       JwcmVmZXJyZWRfdXNlcm5hbWUiOiJzZ
              XJ2aWNlLWFjY291bnQtcGF5bWVudC1zZXJ2aWNlIn0.fO-mOqigv661fSj-
       HNtVGixm_63QYw6Yl5Yo-
              BpDy7vLNQ5uLnWXLTovkiCnOfB8K1mNlAgWM-h5Nwc7IUCy7MJtMg-
              5L0ts0OOQRknIi42QrEN2kSTvQuTwJCtuhmQqfaV23rpn5SG7hf-
       5RVFnpgq3ElfEMW2fs7Ygnv-
              FlQ1Ls7Ns_uKZ7iH7kpwHl30xvXK_Lid9NXEyZI3e-

       7DcpFZPvALRt5_xBJOZk2ZfdITBVKxKc3g7r78ndmK1rnC8ar6t8Fplba2pUv_HYrMvthGp6
       XUwALr31qQcAmBS4Oua-
              qRJr2oa7SwSPfkYBsdR_BvPO1rM2R9h8VSYb_5z-A
              header=Content-Type=application/json
              header=Content-Length=63
              header=User-Agent=Apache-HttpClient/4.5.2 (Java/1.8.0_141)
              header=Host=chapter9-stripe:8080
              locale=[]
              method=POST
            protocol=HTTP/1.1
         queryString=
          remoteAddr=/172.17.0.5:47052
          remoteHost=172.17.0.5
              scheme=http
                host=chapter9-stripe:8080
          serverPort=8080
--------------------------RESPONSE--------------------------
     contentLength=56
       contentType=application/json
            header=Expires=0
            header=Connection=keep-alive
            header=Cache-Control=no-cache, no-store, must-revalidate
            header=Pragma=no-cache
            header=Content-Type=application/json
            header=Content-Length=56
            header=Date=Sun, 25 Feb 2018 04:19:24 GMT
            status=200
==============================================================
```

### 9.4\. 捕获用户认证

为了了解您如何使用用户凭据调用受保护的微服务，让我们为 Cayambe 的新管理界面进行保护。

在这种情况下，已经决定需要一些用户能够从系统中删除类别。这看起来是合理的。但您不希望所有有权访问的人都能删除类别。这绝对不是理想的结果！

为了实现这一目标，您需要进行一些代码修改：

1.  保护 JAX-RS 资源上的 HTTP DELETE 方法。

1.  通过 Keycloak 集成以将用户登录到 UI。

1.  在树中的类别 UI 上添加一个删除按钮，但仅在用户具有管理员角色时启用它。

#### 9.4.1\. 配置 Keycloak

在您之前设置 Cayambe 领域时，我没有展示它，但该领域已经根据您进行用户认证的需求进行了设置。现在让我们具体了解与用户认证相关的部分细节。

##### 列表 9.7\. cayambe-realm.json

```
  "realm": "cayambe",                           *1*
  ...
  "users": [
    {
      "username": "ken",
      ...
      "realmRoles": [                           *2*
        "admin",
        "user",
        "offline_access"
      ],
      ...
    },
    {
      "username": "bob",
      ...
      "realmRoles": [                           *3*
        "user",
        "offline_access"
      ],
      ...
    }
  ],
  "roles": {
    "realm": [                                  *4*
      {
        "name": "user",
        "description": "User privileges"
      },
      {
        "name": "admin",
        "description": "Administrator privileges"
      }
    ]
  },
  "clients": [
    {
      "clientId": "cayambe-admin-ui",           *5*
      "enabled": true,
      "publicClient": true,                     *6*
      "baseUrl": "http://localhost:8080",       *7*
      "redirectUris": [
        "http://localhost:8080/*"
      ]
    },
    {
      "clientId": "cayambe-admin-service",      *8*
      "enabled": true,
      "bearerOnly": true
    }
  ]
```

+   ***1*** 指定要使用的领域名为 cayambe。

+   ***2*** 创建了一个名为 ken 的用户，该用户具有领域角色用户和管理员。

+   ***3*** 创建了一个名为 bob 的用户，该用户具有领域角色用户。

+   ***4*** 定义用户和管理员领域角色。

+   ***5*** 您 UI 的客户端 ID。

+   ***6*** publicClient 表示客户端具有将用户登录到 Keycloak 的能力。

+   ***7*** 应用程序的基本 URL。

+   ***8*** UI 使用的 JAX-RS 端点的客户端 ID。

现在，您已经准备好对应用程序所需的更改进行操作。

#### 9.4.2\. 保护类别删除

从第六章的 admin 目录中获取代码，您只需进行一些小的修改，就可以像对 Stripe 所做的那样进行保护。再次提醒，您需要在 Thorntail 中添加 Keycloak 的 Maven 依赖项：

```
    <dependency>
      <groupId>io.thorntail</groupId>
      <artifactId>keycloak</artifactId>
    </dependency>
```

接下来，您通过 project-defaults.yml 配置与 Keycloak 的集成。

##### 列表 9.8\. project-defaults.yml

```
swarm:
  keycloak:
    secure-deployments:
      chapter9-admin.war:
        realm: cayambe                                     *1*
        auth-server-url: http://192.168.1.13:9090/auth     *2*
        ssl-required: external
        resource: cayambe-admin-service                    *3*
        bearer-only: true
  deployment:
    chapter9-admin.war:                                    *4*
      web:
        security-constraints:
          - url-pattern: /admin/category/*                 *5*
            methods: [ DELETE ]
            roles: [ admin ]
```

+   ***1*** 您部署所使用的认证领域——在本例中，为 cayambe。

+   ***2*** cayambe 领域所在 Keycloak 服务器的 URL。

+   ***3*** 将此资源标识为 cayambe-admin-service，它对应于 cayambe-realm.json 中的客户端 ID。

+   ***4*** 定义 chapter9-admin.war 的特定部署配置部分。这相当于可以作为 web.xml 的一部分提供的内容。

+   ***5*** 请求保护从该微服务中比/admin/category 更深的所有 URL 模式，针对 HTTP DELETE 方法。只有具有管理员角色的用户才能执行具有定义的 URL 和方法的请求。

为了通过 REST 安全删除类别，你需要做到这一步，但你会更进一步，提供有关谁在进行删除的详细信息。

通过从 Thorntail 添加 Keycloak 依赖项，你可以在微服务中检索发出请求的用户的详细信息。这很好，因为能够审计谁在做什么，尽管对于我们的目的，你将打印信息到控制台。

##### 列表 9.9\. `CategoryResource`

```
@DELETE
@Produces(MediaType.APPLICATION_JSON)
@Path("/category/{categoryId}")
@Transactional
public Response remove(
        @PathParam("categoryId") Integer categoryId,
        @Context SecurityContext context) throws Exception {               *1*

    String username = "";

    if (context.getUserPrincipal() instanceof KeycloakPrincipal) {         *2*
        KeycloakPrincipal<KeycloakSecurityContext> kp =
            (KeycloakPrincipal<KeycloakSecurityContext>)
     context.getUserPrincipal();                                           *3*

        username = kp.getKeycloakSecurityContext().getToken().getName();   *4*
    }

    try {
        Category entity = em.find(Category.class, categoryId);
        em.remove(entity);
        System.out.println(username + " is deleting category with id: " +
categoryId);                                                             *5*
    } catch (Exception e) {
        return Response
                .serverError()
                .entity(e.getMessage())
                .build();
    }

    return Response
            .noContent()
            .build();
}
```

+   ***1*** 将 JAX-RS SecurityContext 注入为方法参数。这让你可以访问来自 HTTP 请求的安全信息。

+   ***2*** 检查用户主体是否为 KeycloakPrincipal 类型，这是你预期的类型。

+   ***3*** 将用户主体检索到 KeycloakPrincipal。

+   ***4*** 从 HTTP 请求的令牌中获取发起请求的用户的用户名。

+   ***5*** 打印一个简单的审计消息，说明谁正在删除哪个地址。

#### 9.4.3\. 在 UI 中认证用户

现在由于你的 RESTful 端点对类别删除是安全的，你可以从应用程序 UI 中提供该功能。要查看你对 UI 所做的更改，请查看代码中第九章的 /chapter9/admin_ui/ui 目录。

在这种情况下，你选择通过将 NPM 依赖项添加到 package.json 中的 `keycloak-js` 来包含 Keycloak 提供的 JavaScript。你也可以直接从服务器下载适当的 JavaScript，从 http://localhost:9090/auth/js/keycloak.js。

与你的基于 Java 的服务一样，你需要一个 keycloak.json 文件来配置我们与 Keycloak 服务器的连接。

##### 列表 9.10\. Admin UI 的 keycloak.json

```
{
  "realm": "cayambe",
  "auth-server-url": "http://192.168.1.13:9090/auth",
  "ssl-required": "external",
  "resource": "cayambe-admin-ui",
  "public-client": true
}
```

这段代码现在应该很熟悉了，因为它涵盖了连接到 Keycloak 的典型要求。它将 `cayambe-admin-ui` 定义为你之前作为客户端 ID 指定的资源，在导入到 Keycloak 中的 cayambe-realm.json 文件中。

在 keycloak.json 文件就绪后，你可以初始化你的 Keycloak 连接。

##### 列表 9.11\. keycloak-service.js

```
import Keycloak from 'keycloak-js';                  *1*

const keycloakAuth = Keycloak('/keycloak.json');     *2*
keycloakAuth.init({ onLoad: 'check-sso' })           *3*
  .success((authenticated) => {                      *4*
    // Handle successful initialization
  })
  .error(() => {
    // Handle failure to initialize
  });
```

+   ***1*** 从 keycloak-js NPM 模块导入 Keycloak 对象。

+   ***2*** 创建 Keycloak 对象，并告诉它配置文件 keycloak.json 的位置。

+   ***3*** 使用 check-sso 初始化 Keycloak，它只检查用户是否当前已登录。

+   ***4*** 如果你成功连接到 Keycloak，你会收到一个认证参数，告诉你用户是否已认证。

作为 列表 9.11 中的 `success()` 处理的一部分，你想要设置你稍后需要的变量。其中之一是检索登录 Keycloak 的 URL，因为你需要将 URL 添加到 UI 中：

```
this.auth.loginUrl = this.auth.authz.createLoginUrl();
```

然后，你可以将此值传递到你的 ReactJS 组件的页面标题中，以便你可以提供登录链接：

```
<li className="dropdown">
  <a className="dropdown-toggle nav-item-iconic"
href={this.props.login}>Login</a>
</li>
```

`this.props.login`被设置为 Keycloak 登录 URL 的值，您在`this.auth.loginUrl`中设置。您还希望将有关当前登录用户的信息添加到页面标题中，并提供一种让该用户注销的方式。探索 JavaScript 并了解它是如何工作的将成为您的练习。

最后一个步骤是在 UI 中提供一个按钮来删除一个类别。`CategoryListContainer`是一个 ReactJS 组件，它将为`adminRole`属性设置一个布尔值，以指示用户是否具有该角色。

然后，您只需要 HTML 代码来根据此属性启用和禁用按钮：

```
<button disabled={!this.props.adminRole} className="btn btn-danger"
onClick={() => this.props.onDelete(category.id)}>Delete</button>
```

这基本上完成了大部分 UI 工作，除了将您为认证用户拥有的令牌传递给任何需要它的请求。让我们现在就做这件事。

您需要修改 ReactJS 的`delete`动作，以便在请求中设置令牌，就像您在 Payment 微服务中之前所做的那样。在 JavaScript 中，这个过程是相似的。

##### 列表 9.12. 删除管理员类别

```
import axios from 'axios';                                  *1*
const ROOT_URL = 'http://localhost:8081';                   *2*

if (store.getState().securityState.authenticated) {         *3*
  store.getState().securityState.keycloak.getToken()        *4*
    .then(token => {
      axios.delete(`${ROOT_URL}/admin/category/${id}`, {    *5*
        headers: {
          'Authorization': 'Bearer ' + token                *6*
        }
      })
        .then(response => {
            // Handle success response
        })
        .catch(error => {
            // Handle errors
        });
    })
    .catch(error => {
      dispatch(notifyError("Error updating token", error));
    });
} else {
  dispatch(notifyError("User is not authenticated", ""));
}
```

+   ***1*** 导入一个 NPM 模块以帮助进行 HTTP 调用

+   ***2*** 定义地址微服务的 RESTful 端点的根 URL

+   ***3*** 检查是否存在认证用户

+   ***4*** 从 keycloak-service.js 检索一个认证令牌

+   ***5*** 定义您要执行的 HTTP DELETE 请求

+   ***6*** 将您从 keycloak.getToken()接收到的令牌设置到请求的授权头中

您忘记 UI 和 Keycloak 是如何交互的吗？让我们再次查看图 9.15。

##### 图 9.15. 通过 UI 进行用户认证

![图片](img/09fig15_alt.jpg)

每当 UI 在您的 RESTful 端点上调用`delete`时，如果存在，令牌将被设置在请求上。目前，UI 的其他请求不会传递令牌，但如果需要保护额外的端点或记录有关发起请求的用户的信息，则可以以类似的方式添加。 

#### 9.4.4. 测试新的 UI 和服务是否都正常工作

是时候尝试新的 UI 了。如果 Keycloak 还没有运行，请使用本章中之前使用的命令启动它。启动您的 Admin 服务的 RESTful 端点，切换到/chapter9/admin_ui/admin，然后运行以下命令：

```
mvn thorntail:run
```

最后，您可以运行 UI！您想要模拟一个生产构建，因此您需要一个单独的命令来构建并启动 UI：

```
mvn clean install
java -jar target/chapter9-ui-thorntail.jar
```

现在，您可以导航到 http://localhost:8080，您将看到应用程序的主页，如图 9.16 所示。

##### 图 9.16. Cayambe Admin 屏幕

![图片](img/09fig16_alt.jpg)

您可以看到您的类别，就像之前一样，但现在您在右上角还有一个登录链接，以及每个类别的禁用删除按钮。

点击登录，您将被重定向到 Keycloak 进行身份验证。将用户名输入为`bob`，密码输入为`password`。您将被重定向回您的应用程序，并且现在您已经认证成功，如图 9.17 所示。

##### 图 9.17. 使用用户角色登录的 Cayambe Admin

![图片](img/09fig17_alt.jpg)

虽然您已经认证，但删除按钮仍然处于禁用状态。因为 Bob 只有用户角色，您没有权限删除类别。

要了解如何删除一个类别，让我们通过点击右上角的用户详情退出 Bob，然后从选项中选择登出。

现在，让我们使用之前相同的密码以 `ken` 身份登录；参见图 9.18。

##### 图 9.18\. 使用管理员角色登录时的 Cayambe 管理员界面

![](img/09fig18_alt.jpg)

删除按钮现在呈鲜艳的红色，表示您可以使用它。点击它将删除您选择的类别，您将看到该类别被删除，以及一条消息通知您类别已成功删除。

我在本章中没有涵盖很多 ReactJS 代码，例如检查令牌有效性和在它即将过期时刷新它的代码。请查看书中源代码中为应用程序提供的所有 Java-Script 代码。

### 摘要

+   无论微服务是为内部用户还是外部用户设计的，保护您的微服务都是至关重要的。您无法预测所有可能尝试通过您的微服务造成损害的恶意用户类型。

+   Keycloak 可以接受载体令牌，提供授权客户端，并提供为保护您的微服务配置的简单方法。

+   您可以在没有用户的情况下对 Keycloak 进行身份验证，这对于当接收方受到保护时进行微服务到微服务的调用是必不可少的。

+   您可以将 Keycloak 集成到应用程序 UI 中以提供身份验证，并将令牌传递到受保护的 RESTful 端点。

## 第十章\. 构建微服务混合架构

*本章涵盖*

+   运行 Cayambe 单体

+   使用混合方法将微服务集成到 Cayambe 中

+   修改 Cayambe 以集成您的微服务

+   在混合云中运行集成的 Cayambe

本章首先向您展示旧版的 Cayambe 以及如何在本地运行它。然后，在介绍了一些关于使用混合方法集成微服务理论之后，您将重新审视您希望实现的新 Cayambe 架构。接下来，您将深入实施混合方法，使用本书迄今为止开发的微服务。最后，您将带着焕然一新的 Cayambe 单体以及所需的微服务，在云端运行它们。

### 10.1\. Cayambe 单体

图 10.1 从用户的角度提供了一个 Cayambe 主页的提醒。

##### 图 10.1\. Cayambe 主页

![](img/10fig01_alt.jpg)

Cayambe ([`sourceforge.net/projects/cayambe/`](https://sourceforge.net/projects/cayambe/)) 被描述为“使用 Java Servlets & JSP & EJB 的 J2EE 电子商务解决方案”。它是基于 JDK 1.2 构建的，并使用 Apache Struts v1。现有的代码，最后更新于 15 年前，可以在 [`cayambe.cvs.sourceforge.net/viewvc/cayambe/`](http://cayambe.cvs.sourceforge.net/viewvc/cayambe/) 找到，已被下载并导入到本书的代码仓库中的 /cayambe 目录下。

我在寻找兼容版本的 Apache Struts 以及使其在 JDK 8 上编译所需的更改方面遇到了初步挑战！我还解决了一些小错误，以确保基本 UI 尽可能地功能齐全（考虑到我没有参与 Cayambe 的创建，尽可能做到这一点）。


##### 注意

需要进行的更改以编译和运行原始 Cayambe 代码超出了本书的范围。但您可以通过查看代码的 Git 提交历史来查看这些更改 [`mng.bz/4MZ5`](http://mng.bz/4MZ5)。


图 10.2 提供了 Cayambe 当前架构的代码层的详细视图。您从用于 UI 的 JavaServer Pages (JSP) 开始；这些页面与 Struts 表单和操作交互。反过来，它们与一层代表者交互，这些代表者与被称为 *后端* 的企业 JavaBeans (EJB) 通信，因为它们不涉及面向用户的代码。最后，EJB 在提供数据库持久性的数据访问对象 (DAO) 上执行调用。

图 10.2 提供了对 Cayambe 内部众多层的深入了解，以及每一层的哪些部分相互交互。例如，您可以看到 Struts 表单和操作对于 Admin WAR 和 Cart WAR 都使用相同的代表类来处理类别和产品。尽管这种情况在旧代码中很典型，但您应该使用如 DDD（领域驱动设计）这样的设计工具，这在第一章（kindle_split_010.xhtml#ch01）中已讨论过，以将管理域模型与用户下订单分离。您可能希望某些数据（如类别和产品）仅适用于网站管理员，而不适用于试图下订单的用户。

##### 图 10.2\. Cayambe 代码结构

![](img/10fig02_alt.jpg)

### 10.2\. 运行 Cayambe 单体

在本地运行 Cayambe 需要以下先决条件：

+   您可以从 [`mng.bz/uZdC`](http://mng.bz/uZdC) 下载 WildFly 11.0.0.Final

+   您可以从 [`dev.mysql.com/downloads/connector/j/`](https://dev.mysql.com/downloads/connector/j/) 下载 MySQL Connector for Java

+   本地或 Docker 容器中的运行中的 MySQL 服务器

#### 10.2.1\. 数据库设置

在运行中的 MySQL 服务器上，您现在可以设置数据库并加载数据。

##### 列表 10.1\. 创建数据库并加载数据

```
mysql -h127.0.0.1 -P 32768 -uroot                                         *1*
create user 'cayambe'@'172.17.0.1' identified by 'cayambe';               *2*
grant all privileges on *.* to 'cayambe'@'172.17.0.1' with grant option;  *3*
create database cayambe;                                                  *4*
use cayambe;                                                              *5*
source \cayambe\sql\mysql.sql                                             *6*
source \cayambe\sql\test_data.sql                                         *7*
```

+   ***1*** 以 root 用户身份连接到运行在 localhost 上 32768 端口的 MySQL 服务器。这可能在您的环境中不同。

+   ***2*** 创建一个名为 cayambe 的用户，密码为 cayambe。

+   ***3*** 授予 MySQL 服务器中 cayambe 用户权限。

+   ***4*** 创建一个名为 cayambe 的数据库。

+   ***5*** 切换到使用您刚刚创建的数据库。

+   ***6*** 执行 mysql.sql 中的 SQL 脚本来创建 Cayambe 所需的所有表。

+   ***7*** 执行 test_data.sql 中的 SQL 脚本来将初始数据加载到表中。

通过这些步骤，您现在有一个可以用于 Cayambe 的数据库。下一个任务是配置 WildFly 以能够访问您刚刚设置的数据库。

#### 10.2.2\. WildFly 设置

在您将 WildFly 11.0.0.Final 下载文件提取到您选择的目录之后，您需要提供一些设置，以便 WildFly 知道 MySQL 驱动器的位置。为此，您需要在 WildFly 提取的位置内创建 /modules/system/layers/base/com/mysql/main。

在您刚刚创建的目录中，复制您之前下载的 MySQL 连接器 for Java JAR 文件。在同一个目录中，创建此文件。

##### 列表 10.2\. 单体模式的 MySQL 驱动器模块.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<module  name="com.mysql">             *1*

  <resources>
    <resource-root path="mysql-connector-java-5.1.43-bin.jar"/>    *2*
  </resources>
  <dependencies>                                                   *3*
    <module name="javax.api"/>
    <module name="javax.transaction.api"/>
  </dependencies>
</module>
```

+   ***1*** 将模块名称设置为 com.mysql，与您创建的目录结构相匹配

+   ***2*** MySQL 连接器 for Java JAR 的路径。您的 JAR 可能需要不同的版本号。

+   ***3*** WildFly 中 JDBC 驱动器所需的某些依赖项

您在这里所做的是创建一个 JBoss 模块定义，该定义由 WildFly 使用。JBoss Modules 是 WildFly 管理类加载器核心的开源项目，它通过在类加载器之间分离类来防止冲突。对于此示例，您不需要了解 JBoss Modules 如何工作。您需要了解的是如何创建一个新的模块，就像您在这里所做的那样，以将 JDBC 驱动器添加到 WildFly 中。

最后，您需要告诉 WildFly 关于新的数据库驱动程序，并定义一个 Cayambe 可以用来与数据库通信的新数据源。所有 WildFly 配置都位于 standalone.xml 中。您需要在 WildFly 安装目录的 /standalone/configuration/ 中找到 standalone.xml 并打开文件进行编辑。找到数据源的子系统部分，并用以下内容替换整个部分。

##### 列表 10.3\. standalone.xml 片段

```
<subsystem >
  <datasources>
    <datasource jndi-name="java:jboss/datasources/ExampleDS"
        pool-name="ExampleDS" enabled="true" use-java-context="true">       *1*
      <connection-url>jdbc:h2:mem:test;DB_CLOSE_DELAY=-
1;DB_CLOSE_ON_EXIT=FALSE</connection-url>
      <driver>h2</driver>
      <security>
        <user-name>sa</user-name>
        <password>sa</password>
      </security>
    </datasource>
    <datasource jta="true" jndi-name="java:/Climb" pool-name="MySqlDS"
enabled="true" use-ccm="true">                                            *2*
      <connection-url>jdbc:mysql://localhost:32768/cayambe</connection-url> *3*
      <driver-class>com.mysql.jdbc.Driver</driver-class>
      <driver>mysql</driver>                                                *4*
      <security>
        <user-name>cayambe</user-name>                                      *5*
        <password>cayambe</password>
      </security>
      <validation>
        <valid-connection-checker
          class-name="org.jboss.jca.adapters.jdbc.extensions.mysql
.MySQLValidConnectionChecker"/>
        <background-validation>true</background-validation>
        <exception-sorter class-
     name="org.jboss.jca.adapters.jdbc.extensions.mysql
.MySQLExceptionSorter"/>
      </validation>
    </datasource>
    <drivers>
      <driver name="h2" module="com.h2database.h2">                         *6*
        <xa-datasource-class>org.h2.jdbcx.JdbcDataSource</
xa-datasource-class>
      </driver>
      <driver name="mysql" module="com.mysql">                              *7*
        <xa-datasource-class>com.mysql.jdbc.jdbc2.optional.MysqlXADataSource</
xa-datasource-class>
      </driver>
    </drivers>
  </datasources>
</subsystem>
```

+   ***1*** WildFly 中现有的 ExampleDS 数据源。它尚未被修改。

+   ***2*** Cayambe 的数据源在 JNDI 名称 java:/Climb 下可访问

+   ***3*** 数据库的 MySQL 连接 URL。需要根据您的环境进行修改。

+   ***4*** mysql 是驱动器定义的名称，它添加在列表的末尾。

+   ***5*** 在 MySQL 中为数据库创建的安全凭证

+   ***6*** ExampleDS 的现有 h2 驱动器

+   ***7*** 指向您之前创建的 com.mysql 模块的 mysql 驱动器定义

这就是您需要做的所有配置 WildFly 以与 Cayambe 一起工作的步骤。

#### 10.2.3\. 运行 Cayambe

您几乎准备好启动 Cayambe 并看到它运行了。但首先您需要构建 EAR 部署。图 10.3 提醒您从部署角度来看 Cayambe 的样子，这是您在 第二章 中首次看到的。

##### 图 10.3\. Cayambe 单体部署

![图片 10.3](img/10fig03.jpg)

Cayambe 使用 EAR（企业应用程序存档）作为打包部署的手段。EAR 允许 Cayambe 包含多个 WAR 文件，以及可以共享的通用 JAR 库。

| |
| --- |

##### 注意

尽管 EAR 是打包 Java EE 部署的首选方法，但当前更常见的是 WAR 部署。这并不意味着 EAR 没有被使用，无论是出于选择还是遗留代码，但 EAR 的使用不如以前普遍。


要构建 Cayambe，您需要切换到书籍代码的 /cayambe 目录并运行以下命令：

```
mvn clean install
```

Maven 将构建项目代码中包含的每个 JAR 和 WAR 文件，并将其打包成 EAR 文件供您部署。构建完成后，将 /cayambe-ear/target/cayambe.ear 复制到 WildFly 安装目录的 /standalone/deployments 中。

现在从 WildFly 安装根目录运行以下命令以启动 WildFly，包括您的部署：

```
./bin/standalone.sh
```

当 WildFly 启动时，会在控制台输出大量消息，然后您的部署开始。消息停止后，WildFly 准备接受 Cayambe 的流量，您应该会看到一个包含如下内容的消息：

```
WFLYSRV0025: WildFly Full 11.0.0.Final (WildFly Core 3.0.8.Final) started in
6028ms
```

您可以通过 http://localhost:8080 访问用户站点，通过 http://localhost:8080/admin 访问管理站点。

### 10.3\. Cayambe 混合——单体与微服务

在 第一章 中，您学习了用于单体的混合模式，其中现有单体可以将现有功能迁移到微服务环境中。这种模式允许那些需要更高可扩展性或性能的单体部分被有效地分离，而无需重建整个单体以进行改进。让我们回顾一下使用混合模式的单体可能看起来是什么样子；参见 图 10.4。

##### 图 10.4\. 企业 Java 和微服务混合架构

![图片 10.4](img/10fig04_alt.jpg)

| |
| --- |

##### 注意

在这个特定实例中，您不会使用一个面向所有微服务的网关。

| |
| --- |

能够将单体应用拆分成多个部分，同时分离这些部分可能部署的位置，这确实有好处。虽然这样做会增加开销，至少在网络调用和性能方面，但优势通常大于任何缺点。当这些优势围绕关键方面，如持续交付和发布节奏时，这一点尤其正确。

在 第二章 中，您开发了一个新的管理 UI，以及用于与数据交互的 RESTful 端点。在 第七章 中，您引入了一个单独的微服务来处理卡片支付，以便更容易地与外部系统集成。最后，在 第九章 中，您为您最初在第二章 第二章 中创建的管理 UI 添加了安全性。

它们是如何结合在一起的？图 10.5 表示了 Cayambe 混合单体提出的架构。您将结合原始单体的大块内容以及您在本书中开发的新微服务。这种架构确实已经从起点走了很长的路，但您仍然有一些工作要做。

##### 图 10.5\. 提出的 Cayambe 混合单体

![](img/10fig05_alt.jpg)

那么，您在 图 10.5 中到底做了什么？您想要在结账过程中集成支付微服务以处理卡片支付，并且您希望 UI 从 Admin 而不是存储数据本身检索类别信息。除了新的微服务之外，您还用一个新的 UI 替换了管理界面，因此您可以从 Cayambe 中删除旧的界面。

让我们看看每个集成的需求。Cayambe 混合单体及其微服务的所有代码都包含在本书的代码中，位于 /chapter10。

#### 10.3.1\. 集成支付微服务

由于集成了支付微服务（特别是您正在使用外部支付提供商——在这种情况下，Stripe），您不再需要存储客户的信用卡信息。这是一个巨大的好处，因为存储信用卡信息的规则和限制可能很难执行，将这项责任转交给在该领域专门化的公司更容易。

由于您不需要存储这些信息，让我们将其从 Cayambe 的 `billing_info` 表中删除。您修改 /sql/cayambe/mysql.sql，以便删除以下列：

+   `name_on_card`

+   `card_type`

+   `card_number`

+   `card_expiration_month`

+   `card_expiration_year`

+   `authorization_code`

您将这些列替换为一个用于 `card_charge_id` 的单列。在更改数据库中存储的内容时，您还需要更新传递这些值的类。

##### 列表 10.4\. `OrderDAO`

```
public class OrderDAO {
    public void Save(OrderVO orderVO)
    {
    ...

        StringBuffer sqlBillingInfo = new StringBuffer(512);
        sqlBillingInfo.append("insert into billing_info ");
        sqlBillingInfo.append("(order_id,name,address1,address2,city,state,
zipcode,country,name_on_card,");
        sqlBillingInfo.append("card_charge_id,phone,email) ");               *1*
        sqlBillingInfo.append("values ('" );
        sqlBillingInfo.append(orderId);
        sqlBillingInfo.append("','");

    ...

        sqlBillingInfo.append(orderVO.getBillingInfoVO().getCountry());
        sqlBillingInfo.append("','");
        sqlBillingInfo.append(orderVO.getBillingInfoVO().getCardChargeId()); *2*
        sqlBillingInfo.append("','");

    ...
   }
    ...

    public OrderVO getOrderVO( OrderVO orderVO )
    {
    ...
            b.setCardChargeId( rs.getString("billing_info.card_charge_id") );*3*
    ...
   }

}
```

+   ***1*** 从选择语句中删除现有的卡片列，并添加 card_charge_id。

+   ***2*** 删除对删除列设置值的调用，并用 getCardChargeId() 替换新字段。

+   ***3*** 删除旧卡片数据列的检索，并添加一个用于 card_charge_id 的列。

在这里，您修改`OrderDAO`，该 DAO 直接与数据库交互以存储和检索订单数据。因为您已经修改了`BillingInfoVO`上的方法，所以您现在也需要在那里进行更改，如列表 10.5 所示。

##### 列表 10.5\. `BillingInfoVO`

```
public class BillingInfoVO implements Serializable {

  private Long billingId = null;
  private Long orderId =  null;
  private String name = null;
  private String address = null;
  private String address2 = null;
  private String city = null;
  private String state = null;
  private String zipCode = null;
  private String country = null;
  private String phone = null;
  private String email = null;

  private String cardToken = null;                                          *1*
  private String cardChargeId = null;

  ...

  public void setCardToken ( String _cardToken ) { cardToken = _cardToken; }*2*
  public String getCardToken () { return cardToken; }

  public void setCardChargeId ( String _cardChargeId ) { cardChargeId =
_cardChargeId; }
  public String getCardChargeId () { return cardChargeId; }
}
```

+   ***1*** 将 nameOnCard、cardType、cardExpirationMonth、cardExpirationYear 和 authorizationCode 替换为 cardToken 和 cardChargeId。cardToken 用于从 UI 传递令牌，您很快就会看到。

+   ***2*** 删除之前提到的字段的 getter 和 setter 方法，并为 cardToken 和 cardChargeId 添加它们。

现在您已经修改了传递的数据对象，让我们添加从 Cayambe 单体内部调用 Payment 客户端代理所需的代码。

为了能够发送和接收 JSON 并将其转换为对象，您需要`ChargeStatus`、`PaymentRequest`和`PaymentResponse`。为了方便起见，您已经从 Payment 微服务中复制了这些文件，因此您已经有了它们。接下来，您需要一个表示支付的接口。

##### 列表 10.6\. `PaymentService`

```
@Path("/")
public interface PaymentService {
    @POST
    @Path("/sync")                                                 *1*
    @Consumes(MediaType.APPLICATION_JSON)                          *2*
    @Produces(MediaType.APPLICATION_JSON)
    PaymentResponse charge(PaymentRequest paymentRequest);         *3*
}
```

+   ***1*** 您正在使用 Payment 微服务的/sync 端点。

+   ***2*** RESTful 端点将消费和产生 JSON。

+   ***3*** 要代理的方法将调用 Payment。

为了定义外部 Payment 微服务，您需要这些所有内容。现在让我们看看如何将其集成到现有的 Struts 代码中。在保存订单之前处理卡交易，您需要从/cayambe-hybrid/checkout 内部修改`SubmitOrderAction`。

##### 列表 10.7\. `SubmitOrderAction`

```
public class SubmitOrderAction extends Action                                   *1*
{
  public ActionForward perform( ActionMapping mapping, ActionForm form,
      HttpServletRequest request, HttpServletResponse response )
      throws IOException, ServletException
  {
    ...

    OrderActionForm oaf = (OrderActionForm)form;

    try {
      delegate = new CheckOutDelegate();
      OrderVO orderVO = new OrderVO();
      orderVO = (OrderVO)oaf.toOrderVO();
      orderVO.setCartVO( (CartVO) session.getAttribute("Cart") );

      // Call Payment Service
      ResteasyClient client = new ResteasyClientBuilder().build();
      ResteasyWebTarget target =
        client.target("http://cayambe-payment-service-
myproject.192.168.64.33.nip.io");                                             *2*
      PaymentService paymentService = target.proxy(PaymentService.class);       *3*
      PaymentResponse paymentResponse =
paymentService.charge(new PaymentRequest()                                    *4*
                          .amount((long) (orderVO.getCartVO().getTotalCost()
* 100))
                          .cardToken(oaf.getCardToken())
                          .orderId(Math.toIntExact(orderVO.getOrderId()))
      );

      orderVO.getBillingInfoVO().setCardChargeId(paymentResponse.getChargeId());*5*
      delegate.Save ( orderVO );

      CartDelegate cartDelegate = new CartDelegate();
      cartDelegate.Remove( orderVO.getCartVO() );

    } catch(Exception e) {
      forwardMapping = CayambeActionMappings.FAILURE;
      errors.add( ActionErrors.GLOBAL_ERROR, new
ActionError("error.cart.UpdateCartError") );
    }

  return mapping.findForward( forwardMapping );
  }
}
```

+   ***1*** 用于处理订单提交的现有 Struts 操作类

+   ***2*** 创建一个 ResteasyClient，在 OpenShift 中调用 Payment。

+   ***3*** 创建 PaymentService 的代理实例。

+   ***4*** 调用 Payment，传递一个包含订单金额和 Stripe 的 cardToken 的 PaymentRequest。

+   ***5*** 将从 Payment 返回的 chargeId 设置到 BillingInfoVO 上。

上述代码将熟悉于第六章、第七章和第八章，因为您在那些示例中也使用了 RESTEasy 客户端代理生成。

在创建`PaymentRequest`实例时，您调用了`oaf.getCardToken()`，它包含您处理 Stripe 请求所需的卡令牌。但您需要更新`OrderActionForm`以提供该信息。

`OrderActionForm`位于/cayambe-hybrid/web-common。您需要删除以下字段及其关联的 getter 和 setter：

+   `nameOnCard`

+   `cardNumber`

+   `cardType`

+   `cardExpirationMonth`

+   `cardExpirationYear`

最后，您添加了一个类型为`String`的`cardToken`字段，以及它的 getter 和 setter。

让我们修改结账页面以捕获信用卡详情，在调用 Stripe 检索代表信用卡的`cardToken`之前。为此，您需要更新位于/cayambe-hybrid/checkout 中的 CheckOutForm.jsp。

##### 列表 10.8\. CheckOutForm.jsp

```
...
<script src="https://js.stripe.com/v3/"></script>
<script type="text/javascript">
  var stripe = Stripe({STRIPE_PUBLISH_KEY});                            *1*
  var elements = stripe.elements();                                     *2*
...
  var card = elements.create('card', {style: style});

  function stripeTokenHandler(token) {
    // Insert the token ID into the form so it gets submitted to the server
    var cardToken = document.getElementById('cardToken');
    cardToken.value = token.id;                                         *3*

    // Submit the form
    document.getElementById('orderForm').submit();                      *4*
  };

  document.body.onload = function() {
    card.mount('#card-element');                                        *5*

    card.addEventListener('change', function(event) {                   *6*
      var displayError = document.getElementById('card-errors');
      if (event.error) {
        displayError.textContent = event.error.message;
      } else {
        displayError.textContent = '';
      }
    });

    var form = document.getElementById('orderForm');                    *7*
    form.addEventListener('submit', function(event) {
      event.preventDefault();

      stripe.createToken(card).then(function(result) {                  *8*
        if (result.error) {
          // Inform the user if there was an error.
          var errorElement = document.getElementById('card-errors');
          errorElement.textContent = result.error.message;
        } else {
          stripeTokenHandler(result.token);                             *9*
        }
      });
    });
  };
</script>

<form:form name="OrderForm" styleId="orderForm"
type="org.cayambe.web.form.OrderActionForm"
    action="SubmitOrder.do" scope="request">
...
  <tr>
    <th align="right">
      <label for="card-element">
        Enter card details
      </label>
    </th>
    <td align="left">
      <div id="card-element">                                         *10*
        <!-- A Stripe Element will be inserted here. -->
      </div>

      <!-- Used to display form errors. -->
      <div id="card-errors" role="alert"></div>
      <form:hidden property="cardToken" styleId="cardToken"/>         *11*
    </td>
  </tr>
...
</form:form>
```

+   ***1*** 创建一个 Stripe JavaScript 实例，传入一个发布者密钥。

+   ***2*** 初始化 Stripe 的预构建 UI 组件。

+   ***3*** 将您从 Stripe 收到的令牌 ID 设置到 cardToken 元素上。

+   ***4*** 获取 orderForm 并提交它。

+   ***5*** 当文档加载时，将 Stripe 卡元素挂载到 card-element div 上。

+   ***6*** 在 UI 组件上添加一个事件监听器来处理 Stripe 错误。

+   ***7*** 在 orderForm 上添加一个提交事件监听器。

+   ***8*** 提交事件监听器请求 Stripe 从 UI 中的卡元素创建令牌。

+   ***9*** 如果 Stripe 返回成功，调用 stripeTokenHandler 函数。

+   ***10*** 用于容纳 Stripe 为您创建的卡元素的 div。

+   ***11*** 添加一个隐藏表单字段，将 Stripe 卡令牌传递到 OrderActionForm。


##### 备注

如何将 Stripe 的 UI 元素集成到网站中的完整细节可以在 [`stripe.com/docs/stripe-js`](https://stripe.com/docs/stripe-js) 找到。


除了将卡捕获添加到表单的表中之外，您还删除了所有捕获每条信用卡信息的现有字段。

为了使前面的 CheckOutForm.jsp 正常工作，您还需要修改 /cayambe-hybrid/checkout 中的 struts-forms.tld，为表单和隐藏标签都添加 `styleId`。这允许您设置一个将被添加到生成的 HTML 元素的 `id` 属性中的名称。

这就是您需要为 Payment 微服务集成所做的所有更改。现在，是时候集成 Admin 了！

#### 10.3.2\. 集成 Admin 微服务

要集成 Admin，您需要做类似于 Payment 的事情——至少在这一点上，您希望提供发送和接收对象到 Admin 所需的类，以及从表示 Admin 的接口生成代理。

除了能够调用 Admin 之外，您还需要将类别检索集成到 Cayambe 单体中，无论当前在哪里使用类别。在查看 Cayambe 中类别的定义时，您会发现类别是一个独立的数据库表，而类别/父级关系存在于另一个表中。

您还会看到类别从 Cayambe 单体中的各个层被调用，并且 Category EJB 提供了许多与分布在许多 Java 类中的类别交互的方式。这种情况并不利于 Admin 的顺利集成，至少不是像 Payment 那样集成。

由于集成将需要在几乎整个堆栈中进行大量代码更改，你决定尽管这种增强很有益，但与之相关的风险太多。在希望敏捷灵活的同时，你不想因为处理问题而被拖累数周或数月来集成 Admin。这些问题可能包括实际代码集成的问题，以及在 Cayambe 中测试新更改的大量时间——除了回归测试以确保更改不会对代码的其他部分产生连锁反应。这是令人遗憾的，但有时为了生产代码的稳定性，需要做出这样的艰难决定。

难道我浪费了这本书的大部分内容，让你编写一个你不会使用的新的 Admin UI 和服务？远非如此！在第十一章中，你将通过使用事件流来最小化你的风险担忧，允许你在 Cayambe 单体中保留现有代码，同时仍然利用新的 Admin UI 和微服务。

#### 10.3.3. 新的行政 UI

你已经看到了新的行政 UI 及其相关的微服务，但在 Cayambe 单体中已经存在一个行政部分。你从/cayambe-hybrid/admin 中移除了内容，因为你不再需要现有的行政 UI。接下来，你移除了/cayambe-hybrid/cayambe-ear 中存在的所有对 Admin.war 的引用，因为这个 WAR 不再是 EAR 的依赖项，也不需要打包在其中。

#### 10.3.4. Cayambe 混合总结

图 10.6 展示了你当前所处的完整情况，以及尚未开发的剩余部分。你将在第十一章中添加剩余部分。

##### 图 10.6. Cayambe 混合单体

![](img/10fig06_alt.jpg)

### 10.4. 将一切部署到混合云

由于你已经将 Cayambe 单体转换为混合型，部署一切变得更加复杂——但你也在手动完成所有工作。在实际环境中，你希望部署自动化，以使过程更加简单。

本节涵盖了 Cayambe 混合需要设置、配置或部署以运行的所有部分。你需要做的第一件事是让 Minishift 运行。它也应该从一个干净的 OpenShift 环境开始，以移除可能存在的任何服务。你需要在本地 OpenShift 中获得尽可能多的空间！所以，让我们删除你现有的任何 Minishift 虚拟机（虚拟机），从头开始：

```
> minishift delete
> minishift start --cpus 3 --memory 4GB
```

与 Minishift 之前执行的版本相比，主要区别在于你指定了三个虚拟 CPU 和 4GB 的内存。这是必要的，以确保你有足够的空间安装本章节和下一章节所需的服务。

#### 10.4.1. 数据库

让我们创建一个 MySQL 数据库来存储你的数据！运行`minishift console`并登录到 OpenShift 控制台。

打开默认的 My Project。点击顶部附近的添加到项目菜单项并选择浏览目录。这提供了 OpenShift 可以为您安装的所有预构建镜像类型，如图 10.7 所示。

##### 图 10.7\. 在 OpenShift 控制台中选择浏览目录选项

![](img/10fig07_alt.jpg)

点击底部行中的数据存储框，以查看可用的不同数据存储。在打开的数据存储页面中，如图 10.8 所示，点击 MySQL（持久）框中的选择按钮。

您将看到一个包含 MySQL 配置的页面，其中大部分可以保留默认设置。您需要设置的唯一选项是 MySQL 连接用户名、MySQL 连接密码和 MySQL 根用户密码。输入这些字段的值，记下这些信息，然后点击创建。

| |
| --- |

##### 警告

不要使用*cayambe*作为 MySQL 连接用户名，因为这会与您稍后需要创建的用户冲突。

| |
| --- |

##### 图 10.8\. OpenShift 控制台—数据存储

![](img/10fig08_alt.jpg)

一两分钟后，OpenShift 中将有一个 MySQL 服务可用。为了能够远程访问服务并设置 Cayambe 所需的数据库、表和数据，请打开终端窗口，使用`oc login`登录到 OpenShift CLI，然后运行`oc get pods`。该命令返回一个类似以下列表：

```
NAME                                  READY     STATUS      RESTARTS   AGE
mysql-1-xq98q                         1/1       Running     0          2m
```

您需要复制 MySQL pod 的名称，在本例中为 mysql-1-xq98q，以连接到它：

```
oc rsh mysql-1-xq98q
```

在 pod 内部，您然后可以运行以下命令以打开 MySQL 实例的命令提示符：

```
mysql -u root -p$MYSQL_ROOT_PASSWORD -h $HOSTNAME
```

在 MySQL pod 内部，`$MYSQL_ROOT_PASSWORD`和`$HOSTNAME`被定义为环境变量，因此您不需要记住它们来连接到 MySQL 实例。现在您已经进入 MySQL，让我们设置您所需的数据！

##### 管理微服务数据

以下命令为管理数据库创建一个`cayambe-admin`用户，授予用户对`cayambe_admin`数据库的所有权限，创建数据库，并最终切换到使用该数据库：

```
create user 'cayambe-admin' identified by 'cayambe-admin';
grant all privileges on cayambe_admin.* to 'cayambe-admin' with grant option;
create database cayambe_admin;
use cayambe_admin;
```

在`cayambe_admin`数据库的上下文中，您现在可以执行一些 SQL 来创建表并填充初始数据。

打开/chapter10/sql/admin/mysql.sql，并将内容粘贴到您已登录 MySQL 的终端窗口中。您应该会看到 SQL 语句快速闪过，如果一切顺利，则没有错误！现在表已经存在，用/chapter10/sql/admin/data.sql 进行相同的操作以加载数据。

##### 支付微服务数据

您现在为`cayambe-payment`用户和`cayambe_payment`数据库运行一组类似的命令：

```
create user 'cayambe-payment' identified by 'cayambe-payment';
grant all privileges on cayambe_payment.* to 'cayambe-payment' with grant
option;
create database cayambe_payment;
use cayambe_payment;
```

现在打开/chapter10/sql/payment-service/mysql.sql，并将内容粘贴到您已登录 MySQL 的终端窗口中。这将创建所需的两个表，并为 JPA 需要的 ID 序列生成器设置一个初始值。

##### Cayambe 单体数据

最后，为`cayambe`用户和数据库运行一组类似的命令：

```
create user 'cayambe' identified by 'cayambe';
grant all privileges on cayambe.* to 'cayambe' with grant option;
create database cayambe;
use cayambe;
```

打开/chapter10/sql/cayambe/mysql.sql 并将其内容粘贴到终端窗口中，这将创建 Cayambe 单体所需的所有表。然后复制/chapter10/sql/cayambe/test_data.sql 的内容以加载初始测试数据。

#### 10.4.2\. 安全性

您已经有一个 Keycloak 服务器，它是作为第九章的一部分设置的，所以您将重用它：

```
/chapter9/keycloak> java -Dswarm.http.port=9090 -jar keycloak-2018.1.0-
swarm.jar
```

打开 http://localhost:9090/auth/ 并登录到管理控制台。从左侧导航菜单中选择“客户端”选项。从可用客户端列表中点击 cayambe-admin-ui 以打开其详细信息。您需要做的只是更新指定新管理 UI 运行的三个 URL，将端口从 8080 更改为 8090。

#### 10.4.3\. 微服务

现在是时候开始将微服务部署到 OpenShift 了。

##### Admin 微服务

由于 Admin 微服务是从前面的章节中引入的，所以除了部署之外，您不需要对它做任何事情！

```
/chapter10/admin> mvn clean fabric8:deploy -Popenshift
```

在此微服务部署后，您应该在 OpenShift 控制台中看到它。

##### Stripe 微服务

与 Admin 一样，你不需要在 Stripe 代码中做任何事情，所以你只需部署它：

```
/chapter10/stripe> mvn clean fabric8:deploy -Popenshift
```

##### 支付微服务

接下来，您需要部署 Payment，这与其他部署方式相同：

```
/chapter10/payment-service> mvn clean fabric8:deploy -Popenshift
```

#### 10.4.4\. Cayambe 混合

现在您已经准备好为 Cayambe 混合设置 WildFly 应用程序了。您可以使用本章前面下载的 WildFly 11 和 MySQL 连接器 JAR 文件，并将它们解压缩到一个新目录中。

在它们全部提取后，创建一个与/wildfly-11.0.0.Final/modules/system/layers/base/com/mysql/main 相匹配的目录结构。到那个目录中，复制 MySQL 连接器的 JAR 文件，并创建一个包含以下内容的 module.xml 文件。

##### 列表 10.9\. 混合版 MySQL 驱动模块.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<module  name="com.mysql">

  <resources>
    <resource-root path="mysql-connector-java-5.1.43-bin.jar"/>        *1*
  </resources>
  <dependencies>
    <module name="javax.api"/>
    <module name="javax.transaction.api"/>
  </dependencies>
</module>
```

+   ***1*** 这里引用的特定版本需要与您复制到目录中的文件相匹配。

接下来，您需要向 WildFly 提供它需要配置 Cayambe 数据源的信息。打开/wildfly-11.0.0.Final/standalone/configuration/standalone.xml，并用以下内容替换当前数据源的子系统配置。

##### 列表 10.10\. standalone.xml

```
<subsystem >
  <datasources>
    <datasource jndi-name="java:jboss/datasources/ExampleDS" pool-
name="ExampleDS"
        enabled="true" use-java-context="true">
      <connection-url>jdbc:h2:mem:test;DB_CLOSE_DELAY=-
1;DB_CLOSE_ON_EXIT=FALSE</connection-url>
      <driver>h2</driver>
      <security>
        <user-name>sa</user-name>
        <password>sa</password>
      </security>
    </datasource>
    <datasource jta="true" jndi-name="java:/Climb" pool-name="MySqlDS"
enabled="true" use-ccm="true">
      <connection-url>jdbc:mysql://localhost:53652/cayambe</connection-url>
      <driver-class>com.mysql.jdbc.Driver</driver-class>
      <driver>mysql</driver>
      <security>
        <user-name>cayambe</user-name>
        <password>cayambe</password>
      </security>
      <validation>
        <valid-connection-checker
          class-name="org.jboss.jca.adapters.jdbc.extensions.mysql
.MySQLValidConnectionChecker"/>
        <background-validation>true</background-validation>
        <exception-sorter class-name="org.jboss.jca.adapters.jdbc.extensions.mysql
.MySQLExceptionSorter"/>
      </validation>
    </datasource>
    <drivers>
      <driver name="h2" module="com.h2database.h2">
        <xa-datasource-class>org.h2.jdbcx.JdbcDataSource</
xa-datasource-class>
      </driver>
      <driver name="mysql" module="com.mysql">
        <xa-datasource-
     class>com.mysql.jdbc.jdbc2.optional.MysqlXADataSource</xa-datasource-
class>
      </driver>
    </drivers>
  </datasources>
</subsystem>
```

列表 10.10 与列表 10.3 几乎相同，除了一个区别：Cayambe 的 MySQL 实例端口号设置为 53652。您可能会想知道这个端口号的来源，因为它不是一个标准的 MySQL 端口。好吧，您将通过在 OpenShift 中转发`mysql`服务端口来定义该端口，以便您可以访问它：

```
oc port-forward {mysql-pod-name} 53652:3306
```


##### 注意

如果现有的转发端口关闭，或者您的机器重新启动，您必须在 WildFly 能够找到数据库之前重新运行此命令。


#### 10.4.5\. Cayambe EAR

现在 WildFly 已经设置好了，让我们部署修改后的 Cayambe 混合 EAR。首先您需要构建它！

```
/chapter10/cayambe-hybrid> mvn clean install
```

构建完成后，将 /cayambe-hybrid/cayambe-ear/target/cayambe.ear 复制到 /wildfly-11.0.0.Final/standalone/deployments。现在您开始启动 WildFly：

```
/wildfly-11.0.0.Final/bin/standalone.sh
```

启动 WildFly 后，现在可以通过打开 http://localhost:8080 来尝试 Cayambe UI。

#### 10.4.6. 管理界面

最后要启动的是新的管理界面，因为您已经从之前启动了 Admin 微服务。

对于大部分代码，它与您在第九章中使用的代码相同，但有两次小的修改。您调整了 UI 运行的端口为 8090，这样就不会与主 UI 冲突，并且您还修改了 /chapter10/admin-ui/app/actions/category-actioxns.js 中的 `ROOT_URL`，使其成为 OpenShift 控制台中 cayambe-admin-service 显示的 URL。

| |
| --- |

##### 注意

请确保从 URL 中删除尾随斜杠。

| |
| --- |

是时候开始启动管理界面了：

```
/chapter10/admin-ui> mvn clean package
/chapter10/admin-ui> npm start
```

### 摘要

+   您设置了并运行了 Cayambe 单体，以展示在您进行任何修改之前的代码。

+   您将本书中开发的所有微服务集成到 Cayambe 中，并对 Cayambe 进行了必要的修改以实现集成。

+   您了解到，尽管您可能想集成一个微服务（在这种情况下，是 Admin），但有时这样做可能会增加太多风险，因此需要考虑其他选项。

+   您部署了少量微服务以及 Cayambe 混合体，并使它们共同运行。

## 第十一章. 使用 Apache Kafka 进行数据流

*本章涵盖*

+   理解 Apache Kafka 的数据流

+   使用数据流简化架构

+   将数据流集成到 Cayambe 混合体中

在第十章中，您组装了 Cayambe 混合体，将精简后的单体与您的新微服务相结合。本章通过将其切换到使用数据流来简化 Cayambe 混合体中管理数据的访问。

首先，您将了解数据流及其如何使开发人员和架构师都受益。带着这些课程，您将开发一个用于上一章 Cayambe 混合体的数据流解决方案，完成从单体到混合体的旅程。

### 11.1. Apache Kafka 能为您做什么？

在深入研究 Apache Kafka、您将要用于记录和处理数据流的解决方案之前，您需要对数据流有一些背景知识。否则，作为企业 Java 开发人员，Apache Kafka 做什么以及它是如何工作的将完全陌生。

#### 11.1.1. 数据流

*数据流*不仅仅是指 Netflix 如何将电影播放到您的所有设备上。它还指来自可能成千上万个来源的持续生成数据流；每条数据或记录都很小，并且按接收顺序存储。这可能听起来有很多术语，但数据流仍然相对较新，并且总是有新的使用方式在构思中。

哪些类型的数据适用于数据流？简而言之，几乎任何类型的数据在数据流的背景下都可能是有用的。常见的例子包括来自车辆传感器的测量数据、股市的实时股价，以及来自社交网络和网站的流行话题。

数据流的一个常见用例是当你拥有大量数据或记录，并且你想分析它们以寻找模式或趋势时。可能大量数据可以完全忽略，只有关键数据片段是相关的。同一组数据记录也可能有不同的用途，这取决于可能消费它的系统！例如，一个捕捉页面访问事件流的电子商务网站可以使用相同的数据不仅记录用户在购买前访问的页面数量，还可以分析每个页面在所有用户中的查看次数。这就是数据流的美丽之处：你可以用同一组数据解决不同的问题。

图 11.1 展示了数据作为流接收的方式。

##### 图 11.1\. 数据流管道

![图片](img/11fig01_alt.jpg)

特定类型的数据可能来自许多来源，并且按照系统记录数据流的接收顺序添加到流的末尾或管道中。在流中插入特定记录的概念是不存在的。所有东西都是按照接收的顺序添加到末尾的。

虽然你有多种记录和处理数据流的方法，但本章将专注于 Apache Kafka，让我们现在看看它。

| |
| --- |

##### 定义

为了明确本章中使用的术语，术语“数据流”、“数据流”和“流式传输”都指的是同一件事：从源捕获数据的过程。

| |
| --- |

#### 11.1.2\. Apache Kafka

Apache Kafka ([`kafka.apache.org/`](https://kafka.apache.org/))最初由 LinkedIn 于 2010 年开发，作为其中心数据管道的核心。目前，该管道每天处理超过 2000 亿条消息！在 2011 年初，Apache Kafka 被提议作为 Apache 的一个开源项目，并在 2012 年底结束了孵化阶段。在短短几年内，许多企业开始使用 Apache Kafka，包括 Apple、eBay、Netflix、Spotify 和 Uber。

什么是 Apache Kafka？它是一个分布式流平台。我这里所说的“分布式”是什么意思？Apache Kafka 可以将单个数据流的数据分区到集群内的多个服务器上。此外，每个分区都在服务器之间复制，以实现该数据的容错性。

Apache Kafka 有许多配置方式，包括其分布方式和容错级别，但这些内容超出了本章的范围。对于完整详情，请参阅 Apache Kafka 文档[`kafka.apache.org/documentation/`](https://kafka.apache.org/documentation/)。

作为流媒体平台，Apache Kafka 提供的关键功能如下：

+   发布和订阅记录流。

+   以容错和持久的方式存储记录流。

+   在发生时处理记录流。

在其核心，Apache Kafka 是一个分布式提交日志：它不会在数据被记录到流中时通知源，直到数据被提交到日志中。正如我之前提到的，分布式指的是日志或流中的每个提交被分散到分区并复制。

另一种描述 Apache Kafka 的方式是将其视为没有衣服的数据库：数据处于前沿，没有隐藏。数据库在其核心使用提交日志，就像 Apache Kafka 一样，用于跟踪更改和作为从服务器故障中恢复以重建数据库的手段。在 Apache Kafka 中，数据库的衣服（表、索引等）已经被剥离，只剩下提交日志。这使得 Apache Kafka 比常规数据库更易于消费和访问。

Apache Kafka 还使用了企业 Java 开发者熟悉的语义，这些开发者已经集成了消息系统。有*生产者*，它们生成记录或事件，这些事件被添加到流中，相当于之前图 11.1 中存在的多个源。每个记录流被称为*主题*，任何从流中读取记录的东西都是一个*消费者*。图 11.2 展示了生产者和消费者如何与 Apache Kafka 集成。

##### 图 11.2\. Apache Kafka 集成（来自[`kafka.apache.org/intro.html`](https://kafka.apache.org/intro.html)）

![](img/11fig02.jpg)

此外，*连接器*使数据库或其他系统能够成为发送到 Apache Kafka 的记录的来源。最后，*流处理器*能够从一个或多个主题中流式传输记录，对数据进行某种类型的转换，然后将输出到一个或多个不同的主题。

##### 记录是什么？

现在你已经了解了一些构成 Apache Kafka 的组件，让我们定义一下*记录*的含义。流中的每个记录由一个键、一个值和一个时间戳组成。键和值在目的上很简单，但为什么需要一个时间戳呢？时间戳对于 Apache Kafka 知道记录何时被接收（当涉及到分区时将变得更加关键）是至关重要的。

在继续之前，你还需要了解关于记录的额外概念。每个记录在数据流中都是*不可变的*：一旦记录被添加到数据流中，你无法编辑、修改或从数据流中删除记录。你所能做的就是为同一个键提供更新记录，设置不同的值。

图 11.3 扩展了图 11.1 中的流，展示了实时股价流的可能记录。在图 11.3 中，你可以看到没有针对键 RHT 的单个记录。目前有三个，它们的值都不同。这就是数据流的不可变性。如果流不是不可变的，那么很可能会有一个键为 RHT 的记录，它将不断更新为新值。

##### 图 11.3。不可变数据流

![图 11.3 的替代文本](img/11fig03_alt.jpg)

不可变数据流的一个大优点是，对于同一个键，你可以有一个变化的历史记录。当然，在某些情况下，你可能只关心某个事物的当前值。但更常见的是，了解历史并能够确定随时间的变化是至关重要的。

记录也被*持久化*：日志保留在文件系统中，允许在未来任何时间点处理记录。话虽如此，记录保留的时间长度是有限制的。每个记录只保留到特定主题的保留策略允许的时间。一个主题可以被定义为无限期保留记录，前提是磁盘空间不是问题，或者几天后无论是否被消费都会被清除。

##### 主题是如何工作的？

Kafka 中的*主题*与一个类别或类型相关，这种记录可以被发布和消费。例如，你会使用一个主题来实时股价，并为车辆传感器的测量使用另一个单独的主题。

每个主题被划分为一个或多个分区，这些分区位于 Kafka 集群中的一个或多个服务器上。一个*分区*是一个单一的逻辑数据流，或主题，例如你在图 11.1 和 11.3 中看到的，它被分割成多个物理数据流。

图 11.4 显示了一个分区。主题的分区增加了在特定主题写入或读取时可以实现的并行性。该图展示了一个被分割成三个分区的单一主题。每个分区代表一个有序且不可变的记录序列，这些记录不断追加，从而在数据流内创建一个结构化的变更事件日志。给定分区中的每个记录都被分配一个称为*偏移量*的顺序 ID 号。偏移量唯一地标识了特定分区内的记录。

##### 图 11.4。主题分区（摘自[`kafka.apache.org/intro.html`](https://kafka.apache.org/intro.html))

![图 11.4 的替代文本](img/11fig04_alt.jpg)

开发者需要特别注意 Kafka 记录的一个关键点是与之关联的键的定义。如果键在业务上下文中不是真正唯一的，那么键和时间戳组合之间可能会发生重叠——特别是由于 Kafka 保证所有具有相同键的记录都放置在同一个分区上，确保所有键的记录都按顺序存储在单个分区上。

图 11.5 展示了生产者和消费者如何与主题分区交互。

##### 图 11.5. 主题生产者和消费者（摘自 [`kafka.apache.org/intro.html`](https://kafka.apache.org/intro.html))

![](img/11fig05.jpg)

如前所述，生产者总是将新记录写入分区的末尾。消费者通常按顺序处理记录，但能够指定从哪个偏移量开始处理。例如，在图 11.5 中，消费者 B 可能从偏移量 0 开始读取，现在正在处理偏移量 11。消费者 A 在偏移量 9，但可能只从该偏移量开始读取记录，而不是从 0 开始。

图 11.5 介绍了一些关于消费者的概念，值得详细阐述，以便你熟悉它们能做什么：

+   消费者可以从任何偏移量开始读取一个主题，包括从非常开始，偏移量 0。

+   通过在读取记录时指定消费者组，可以对消费者进行负载均衡。

+   *消费者组*是多个消费者的逻辑分组，确保同一消费者组内的每个记录只被单个消费者读取。

### 11.2. 使用流式传输简化单体架构

图 11.6 简要回顾了迄今为止与 Cayambe 混合架构一起开发和集成的内容。灰色部分将在本章中完成，并通过 Apache Kafka 主题将 Admin 和 Cayambe 数据库链接起来，以消除 Cayambe 数据库直接管理类别的需求。这使得能够简单地从数据库 A 将数据传输到数据库 B。

在图 11.6 中没有数据流的情况下，你有几种替代方案：

+   修改 Cayambe JAR 以从 Admin 数据库检索记录。除了不同服务与同一数据库交互是一种不良的数据设计之外，你还在第十章中发现，在这种情况下，这样的更改需要大量的代码更改才能完成。

+   开发一个计划任务以从 Admin 数据库提取所有记录，然后清除并将这些记录插入到 Cayambe 数据库中。这种方法实现起来更简单，但会导致数据不同步的时期，以及当运行该任务的作业正在执行时，Cayambe 中的数据将不可用。根据数据变化的频率，这可能是一个可接受的解决方案，尽管任何计划中的停机时间都远远不是理想的。

+   修改 Admin 微服务以更新 Cayambe 数据库中的记录。虽然这比第一个选项更容易实现，但这种解决方案容易受到事务问题和知道两个更新是否都成功的影响。它需要 Admin 微服务对成功或失败以及如何适当地处理数据库调用中的失败有更多的智能，以便回滚另一个。

为了正确支持 图 11.6 中的模型，你希望能够将数据库更改事件转换为 Kafka 中的记录以便你处理。这种解决方案对 Admin 微服务的影响最小，同时仍然允许你消费其数据。你需要的是一个可以为你做这件事的 Kafka 连接器。

##### 图 11.6\. 当前 Cayambe 混合单体

![图片](img/11fig06_alt.jpg)

有任何工具可以实现这一点吗？是的，当然有！Debezium 是一个开源项目，用于将数据库中的更改流出到 Kafka。

| |
| --- |

##### 注意

Debezium 是一个用于变更数据捕获的分布式平台。你可以启动 Debezium，将其指向你的数据库，并在完全独立的应用程序中对这些数据库上的每个插入、更新或删除做出反应。Debezium 允许你消费数据库行级更改，而不会对当前直接执行这些数据库更新的应用程序产生任何影响或更改。Debezium 的一个巨大好处是，任何消费数据库更改的应用程序或服务都可以进行维护而不会丢失任何更改。Debezium 仍然将更改记录到 Kafka 中，以便在服务再次可用时进行消费。关于 Debezium 的详细信息可以在 [`debezium.io/`](http://debezium.io/) 找到。

| |
| --- |

为了更好地理解 Apache Kafka、数据流以及它们如何集成到你的微服务中，你在这个章节中不会为 Cayambe 混合型实现 Debezium。我将把它留给你作为额外的练习。

对于 Cayambe 混合型，你将直接将事件生产到 Apache Kafka，然后在另一侧消费它们。图 11.7 展示了你要对架构进行的更改，以支持展示 Apache Kafka 的工作方式更多细节。

##### 图 11.7\. Cayambe 混合单体

![图片](img/11fig07_alt.jpg)

你正在向 Admin 微服务添加代码以产生将被发送到 Apache Kafka 主题的事件。然后你有一个 Kafka 微服务来消费这些事件并更新 Cayambe 数据库中的更改。

你仍然会使用数据流来将所需数据从一个地方移动到另一个地方，尽管不使用类似 Debezium 的方法在真实生产使用中可能效率略低，但有助于理解正在发生的事情。

### 11.3\. 部署和使用 Kafka 进行数据流

在查看如何实现与 Kafka 集成的微服务之前，让我们在 OpenShift 上启动并运行 Kafka！如果你还没有运行 Minishift，让我们现在启动它，就像你在第十章中做的那样：

```
> minishift start --cpus 3 --memory 4GB
```

#### 11.3.1\. OpenShift 上的 Kafka

Minishift 启动并运行后，启动 OpenShift 控制台并登录。在你的现有项目中，点击添加到项目，然后点击导入 YAML/JSON。

将/chapter11/resources/Kafka_OpenShift.yml 的内容粘贴到文本框中，其中一些片段如本列表所示。

##### 列表 11.1\. Kafka OpenShift 模板

```
apiVersion: v1
kind: Template
metadata:
  name: strimzi                            *1*
  annotations:
    openshift.io/display-name: "Apache Kafka (Persistent storage)"
    description: >-
      This template installs Apache Zookeeper and Apache Kafka clusters. For
      more information
       see https://github.com/strimzi/strimzi
    tags: "messaging,datastore"
    iconClass: "fa pficon-topology"
    template.openshift.io/documentation-url:
     "https://github.com/strimzi/strimzi"
message: "Use 'kafka:9092' as bootstrap server in your application"
...
objects:
- apiVersion: v1
  kind: Service
  metadata:
    name: kafka                            *2*
  spec:
    ports:
    - name: kafka
      port: 9092                           *3*
      targetPort: 9092
      protocol: TCP
    selector:
      name: kafka
    type: ClusterIP
...
- apiVersion: v1
  kind: Service
  metadata:
    name: zookeeper                        *4*
  spec:
    ports:
    - name: clientport
      port: 2181                           *5*
      targetPort: 2181
      protocol: TCP
    selector:
      name: zookeeper
    type: ClusterIP
...
```

+   ***1*** strimzi 是将在 OpenShift 中出现的应用程序的名称。

+   ***2*** 定义了 kafka 服务

+   ***3*** kafka 服务将在端口 9092 上可用。

+   ***4*** 定义了 zookeeper 服务

+   ***5*** ZooKeeper 服务在端口 2181 上可用。

稍等一下，ZooKeeper 在那里做什么？之前可没提到！没错，之前确实没有提到。ZooKeeper 是一个实现细节，因为它被 Kafka 内部用作分布式键/值存储。这不是你需要与之交互的东西。你在这里看到它是因为你正在扮演运维人员角色，为自己设置 Kafka。

/chapter11/resources/Kafka_OpenShift.yml 最初是从[`mng.bz/RqUn`](http://mng.bz/RqUn)复制的，但被修改为只有一个 Kafka 代理而不是三个。因此，它不支持主题复制，但你的 OpenShift 实例运行 Kafka 所需的资源更少！

在将修改后的文件内容粘贴到弹出窗口后，点击创建，然后继续，你将看到一个表单，你可以指定不同的默认值。现在，保持它们不变，并在页面底部点击创建。OpenShift 现在将配置一个包含单个代理的 Kafka 集群，你可以在主控制台页面下的 strimzi 应用程序中看到它。


##### 警告

完成必要的 Docker 镜像下载并启动容器可能需要一些时间。如果 ZooKeeper 尚未运行，Kafka 集群最初失败，请不要担心。给定时间，它将重新启动，一切都将按预期运行。


在所有 Pod 启动后，打开一个终端窗口并登录到 OpenShift 客户端（如果你还没有登录的话）。你需要检索所有 OpenShift 服务以找到 ZooKeeper 的 URL：

```
> oc get services
NAME                 CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kafka                172.30.225.60    <none>        9092/TCP   5h
kafka-headless       None             <none>        9092/TCP   5h
zookeeper            172.30.93.118    <none>        2181/TCP   5h
zookeeper-headless   None             <none>        2181/TCP   5h
```

从列表中，你可以看到 ZooKeeper 的 URL 是 172.30.93.118。返回 OpenShift 控制台，从菜单选项中选择应用程序和 Pod。这提供了一个 OpenShift 中运行中的 Pod 列表。对于单个代理，应该只有一个 kafka-* Pod。点击该 Pod，然后选择终端标签页，你应该会看到类似于图 11.8 的内容。

##### 图 11.8\. OpenShift pod 终端

![](img/11fig08_alt.jpg)

要使用 Kafka，你需要为你的记录创建一个主题。让我们在终端标签页中做这件事：

```
./bin/kafka-topics.sh --create --topic category_topic --replication-factor
1 --partitions 1 --zookeeper 172.30.93.118:2181
```

你使用 Kafka 脚本来创建一个名为`category_topic`的主题，该主题只有一个分区和一个副本。你只指定单个副本和分区，因为你在集群中只有一个代理。例如，如果你在集群中有三个代理，你可以使用三个分区和 2 的副本因子。

#### 11.3.2\. Admin 微服务

现在 Kafka 已经运行并且你的主题已经创建，是时候修改 Admin 微服务以将事件发送到该主题了！

为了帮助将你的企业 Java 代码与 Kafka 集成，你将使用一个库，该库将 Kafka 的*拉取*方法转换为*推送*方法。这个库仍然处于初级阶段，但很容易使用，因为它去除了直接使用 Kafka API 时所需的大量样板代码。它被编写为一个 CDI 扩展，并以 Maven 工件的形式提供给你使用。代码可在[`github.com/aerogear/kafka-cdi`](https://github.com/aerogear/kafka-cdi)找到。

将 Kafka 的*拉取*方法转换为*推送*方法有什么优势？这对于我们这些更熟悉企业 Java 开发的人来说是有益的，因为在 CDI 编程模型中，我们能够监听事件并在接收到事件时执行操作。这正是我们使用的 Kafka 库为我们带来的，即每次向主题写入新记录时都能监听事件，就像它是 CDI 事件监听器一样。

你需要做的第一件事是更新 Admin 微服务的 pom.xml 以使用新的依赖项：

```
<dependency>
  <groupId>org.aerogear.kafka</groupId>
  <artifactId>kafka-cdi-extension</artifactId>
  <version>0.0.10</version>
</dependency>
```

接下来，你修改 `CategoryResource` 以连接到 Kafka 主题，并生成要附加到其上的记录。

##### 列表 11.2\. `CategoryResource`

```
@Path("/")
@ApplicationScoped
@KafkaConfig(bootstrapServers =
"#{KAFKA_SERVICE_HOST}:#{KAFKA_SERVICE_PORT}")                          *1*
public class CategoryResource {
...
    @Producer
    private SimpleKafkaProducer<Integer, Category> producer;              *2*
...
    public Response create(Category category) throws Exception {
...
        producer.send("category_topic", category.getId(), category);      *3*
...
    public Response remove(@PathParam("categoryId") Integer categoryId,
        @Context SecurityContext context) throws Exception {
...
        producer.send("category_topic", categoryId, null);                *4*
...
}
```

+   ***1*** 识别你连接到的 Kafka 服务器。你可以使用环境变量来指定主机和端口，因为你在与 Kafka 相同的 OpenShift 命名空间中部署微服务。

+   ***2*** 注入一个接受整数作为键和类别作为值的 CDI 生产者。

+   ***3*** `create()` 方法被修改为在创建新的类别之后调用 `send()` 方法。这表示你将记录发送到的主题，以及键和值。

+   ***4*** `remove()` 方法以类似的方式进行修改。与 `create()` 方法的主要区别在于你传递了一个 null 值，因为没有有效的值了。

在 Admin 微服务所做的更改之后，你现在可以部署它了！在部署微服务之前，你需要确保 Keycloak 正在运行，因为你的微服务使用它来保护删除端点。为此，你需要运行以下命令：

```
/chapter9/keycloak> java -Dswarm.http.port=9090 -jar
keycloak-2018.1.0-swarm.jar
```

如果数据库文件尚未从目录中删除，Keycloak 应该启动并记住你之前安装的所有设置。随着 Keycloak 再次运行，你现在可以部署 Admin：

```
/chapter11/admin> mvn clean fabric8:deploy -Popenshift
```

微服务启动并运行后，你可以使用新的管理界面，或者通过 Postman 直接使用 HTTP 请求来更新和删除类别。你怎么知道 Admin 微服务是否正确地将记录放入 Kafka 主题？因为你没有任何消费者消费这些记录！

幸运的是，Kafka 提供了一个可以在控制台中使用的消费者，用于查看主题的内容。在 OpenShift 控制台中，你回到 kafka-* 容器，就像之前一样，并选择终端标签页。在命令行中，运行以下命令：

```
./bin/kafka-console-consumer.sh --bootstrap-server 172.30.225.60:9092 –
from-beginning --topic category_topic
```

或者，你也可以连接到 kafka-* 容器，并远程运行命令：

```
oc rsh kafka-<identifier>
./bin/kafka-console-consumer.sh --bootstrap-server 172.30.225.60:9092 –
from-beginning --topic category_topic
```

你使用了之前从 OpenShift 服务列表中检索到的 Kafka 服务 IP 地址和端口来指定 Kafka 的位置。接下来，你告诉脚本你想要从开始消费所有记录，这与从偏移量 0 开始相同。最后，你给出主题的名称。如果一切正常，你应该会看到通过 Admin 微服务所做的每个更改都对应一个记录。

我们已经涵盖了 Kafka 主题的生产方面。现在让我们看看消费方面。

#### 11.3.3\. Kafka 消费者

书中代码的 /chapter11/kafka-consumer/ 目录中包含了 Kafka 消费者的所有代码。与生产者类似，你需要在 pom.xml 中添加 `kafka-cdi-extension` 依赖项。pom.xml 的其余部分包含常规的 Thorntail 插件和依赖项，以及用于部署到 OpenShift 的 fabric8 Maven 插件。你还指定了一个 MySQL JDBC 驱动依赖项，以便你可以更新 Cayambe 数据库中的记录。

要连接到 Cayambe 数据库，你需要定义一个 `DataSource`。

##### 列表 11.3\. project-defaults.yml

```
swarm:
  datasources:
    data-sources:
      CayambeDS:                                            *1*
      driver-name: mysql                                    *2*
      connection-url: jdbc:mysql://mysql:3306/cayambe       *3*
      user-name: cayambe                                    *4*
      password: cayambe
      valid-connection-checker-class-name:

   org.jboss.jca.adapters.jdbc.extensions.mysql.MySQLValidConnectionChecker
      validate-on-match: true
      background-validation: false
      exception-sorter-class-name:
   org.jboss.jca.adapters.jdbc.extensions.mysql.MySQLExceptionSorter
```

+   ***1*** JNDI 中 DataSource 的名称。

+   ***2*** 使用从 MySQL JDBC 驱动依赖项创建的模块

+   ***3*** OpenShift 上 MySQL 数据库实例的 URL

+   ***4*** Cayambe 数据库的凭据

最后，你创建一个类来处理从 Kafka 主题接收到的记录。

##### 列表 11.4\. `CategoryEventListener`

```
@ApplicationScoped
@KafkaConfig(bootstrapServers =
"#{KAFKA_SERVICE_HOST}:#{KAFKA_SERVICE_PORT}")                            *1*
public class CategoryEventListener {

  private static final String DATASOURCE =
     "java:/jboss/datasources/CayambeDS";                                   *2*

  @Consumer(topics = "category_topic", keyType = Integer.class,
        groupId = "cayambe-listener", offset = "earliest")                  *3*
  public void handleEvent(Integer key, Category category) {                 *4*
    if (null == category) {
      // Remove Category
      executeUpdateSQL("delete from category where category_id = " + key);  *5*
      // Remove from Category Hierarchy
      executeUpdateSQL("delete from category_category where category_id = " +
key);                                                                     *6*
      executeUpdateSQL("delete from category_category where parent_id = " +
key);
    } else {
      boolean update = rowExists("select * from category where category_id = "
+ key);                                                                   *7*
      if (update) {
        // Update Category
        executeUpdateSQL("update category set name = '" + category.getName()*8*
                         + "' header = '" + category.getHeader()
                         + "' image = '" + category.getImagePath()
                         + "' where category_id = " + key);
      } else {
        // Create Category                                                  *9*
        executeUpdateSQL("insert into category (id,name,header,visible,image)
values("
                         + key + ",'" + category.getName() + "', '"
                         + category.getHeader() + "', " +
(category.isVisible() ? 1 : 0)
                         + ", '" + category.getImagePath() + "')");
        executeUpdateSQL("insert into category_category (category_id,
parent_id)"
                         + " values (" + category.getId() + "," +
category.getParent().getId() + ")");
      }
    }
  }

  private void executeUpdateSQL(String sql) {                               *10*
    Statement statement = null;
    Connection conn = null;

    try {
      conn = getDatasource().getConnection();
      statement = conn.createStatement();
      statement.executeUpdate(sql);
      statement.close();
      conn.close();
    } catch (Exception e) {
    ...
    }
  }
boolean rowExists(String sql) {                                             *11*
    Statement statement = null;
    Connection conn = null;
    ResultSet results = null;

    try {
      conn = getDatasource().getConnection();
      statement = conn.createStatement();
      results = statement.executeQuery(sql);
      return results.next();
    } catch (Exception e) {
    ...
    }
    return false;
  }

  private DataSource getDatasource() {                                      *12*
    if (null == dataSource) {
      try {
        dataSource = (DataSource) new InitialContext().lookup(DATASOURCE);
      } catch (NamingException ne) {
        ne.printStackTrace();
      }
    }
    return dataSource;
  }

  private DataSource dataSource = null;
}
```

+   ***1*** 如你在生产者中做的那样，你为配置定义 Kafka 主机和端口。

+   ***2*** 使用 project-defaults.yml 创建的 CayambeDS 的 JNDI 名称。它由 getDatasource() 使用，因此你可以使用更改后的类别更新 Cayambe 数据库。

+   ***3*** @Consumer 将方法标识为接受 Kafka 主题记录，并提供将它们连接到 Kafka API 所需的配置。它定义了你想要记录的主题的名称、键的类型、一个唯一的消费者组标识符，以及你想要从主题的开始偏移量处开始。

+   ***4*** 接收 Kafka 记录的方法，带有传递给键和值类型的参数

+   ***5*** 执行删除类别的 SQL 语句。

+   ***6*** 执行从类别层次结构中删除类别的 SQL 语句，无论是作为子节点还是父节点。

+   ***7*** 执行 SQL 以确定是否存在类别 ID 的行。确定你是否正在更新或插入记录。

+   ***8*** 执行 SQL 更新数据库中类别字段的操作。

+   ***9*** 执行 SQL 将新类别插入数据库，并将其插入到类别层次结构中。

+   ***10*** 处理 SQL 更新执行的方法。

+   ***11*** 检查数据库中是否存在 Category 行的方法。

+   ***12*** 从 JNDI 获取 DataSource 的方法。

`CategoryEventListener` 通过定义键类型、值类型、你正在处理的主题、消费者组以及你想要从流中从头开始处理所有记录，注册一个方法来监听 Kafka 事件。当你收到 Kafka 记录时，然后确定你是否需要删除一个类别，值是 `null`，或者我们正在处理一个新或更新的记录。

为了区分更新和新的类别，你需要在 Cayambe 中对现有类别执行 SQL 语句，以查看此记录是否存在。如果存在，则是一个更新记录；如果不存在，则是一个新记录。

如果你不想运行 SQL 语句来确定你是否正在处理更新或新类别，你可以将 Kafka 中记录的值类型更改为封装对象。`Category` 实例、当前值可以是一个新类型上的字段，带有指示正在处理哪种更改事件的标志。

现在你已经完成了 Kafka 消费者的开发，你准备好看到它协同工作！但在你部署刚刚创建的 Kafka 消费者之前，为了看到它们发生的视觉变化，值得从 第十章 启动 Cayambe 混合模式，如下所示：

```
/wildfly-11.0.0.Final/bin/standalone.sh
```

Cayambe 启动后，打开浏览器并浏览类别树。你应该注意到你通过 Admin 微服务所做的任何更改都是不可见的，这是有道理的，因为你还没有激活更新 Cayambe 数据库的任何更改的过程。所以现在让我们启动你的 Kafka 消费者：

```
/chapter11/kafka-consumer> mvn clean fabric8:deploy -Popenshift
```

当 pod 变得可用时，它应该处理 Kafka 主题上所有现有的记录，因为你指定它从主题的最早偏移量开始。你可以打开服务的日志，查看为每个处理的记录打印的控制台语句。

使用 Kafka 消费者处理过的记录，返回 Cayambe UI 并刷新页面。在浏览类别树并找到通过 Admin 微服务更改的类别时，你会注意到它们现在根据你之前所做的操作已更新或删除。

你已经成功地将两个系统之间的数据解耦，一个拥有数据（Admin 微服务），另一个以只读方式消费其副本。作为额外的好处，只要 Kafka 生产者和消费者正在运行，数据永远不会过时。

### 11.4. 额外练习

如本章前面所述，作为额外练习，尝试将 Cayambe 混合型转换为使用 Debezium 直接处理数据库条目，而不是在 Admin 微服务中由您生成记录。

这也将为当前解决方案提供另一个优势，因为每当需要时，类别层次结构都可以完全从 Kafka 主题记录中重建。该层次结构将包含所有您最初为加载数据库而进行的初始插入记录，以及自那时以来发生的任何插入、更新和删除操作。

### 摘要

+   数据流通过使单独的组件或微服务保持解耦，同时仍然使用相同的数据，从而简化了架构。

+   您可以使用 Apache Kafka 的数据流技术在微服务和应用程序之间共享数据，而无需通过 REST 调用来检索它。


##### 注意

关于使用 Spring Boot 开发微服务的更多详细信息，请参阅附录。

