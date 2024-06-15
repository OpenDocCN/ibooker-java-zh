# 第十二章\. 反应式 REST 客户端：与 HTTP 端点连接

前两章专注于消息传递，这是反应式系统的连接组件。现代消息代理提供了完美的功能集来实现反应式系统的内部通信。然而，在系统的前沿，当您需要集成远程服务时，您很有可能需要使用 HTTP。因此，让我们务实一点，看看如何在不违反反应式原则的情况下消费 HTTP 服务。

在 第八章 中，您看到如何 *公开* 反应式 HTTP 端点。本章介绍另一方面：如何 *消费* HTTP 端点。Quarkus 提供了一种非阻塞方式来消费 HTTP 端点。此外，它提供了可靠性特性，以保护集成点免受故障和缓慢的影响。重要的是要注意，被调用的服务不一定是反应式应用程序，这取决于该服务的实现。

让我们看看 Quarkus 提供了哪些功能来消费 HTTP 端点。

# 与 HTTP 端点交互

Quarkus 提供了多种消费 HTTP 端点的方式：

Vert.x Web 客户端

这种低级别的 HTTP 客户端是基于 Vert.x 和 Netty 实现的（因此本质上是异步的，基于非阻塞 I/O）。

反应式消息连接器

此连接器为每个处理的消息发送 HTTP 请求。

REST 客户端

这种类型安全的方法简化了消费基于 HTTP 的 API。

当您不想被暴露于底层 HTTP 细节（如动词、头部、主体和响应状态）时，Vert.x Web 客户端是非常方便的选择。Web 客户端灵活，并且您可以完全控制 HTTP 请求和响应处理。

要使用 Vert.x Web 客户端，您需要在项目中添加 示例 12-1 中显示的依赖项。

##### 示例 12-1\. Mutiny Vert.x Web 客户端的依赖项

```java
<dependency>
    <groupId>io.smallrye.reactive</groupId>
    <artifactId>smallrye-mutiny-vertx-web-client</artifactId>
</dependency>
```

然后您可以按照 示例 12-2 中所示使用它。

##### 示例 12-2\. Vert.x Web 客户端示例

```java
@ApplicationScoped
public class WebClientExample {

    private final WebClient client;

    @Inject
    public WebClientExample(Vertx vertx) {
        client = WebClient.create(vertx);
    }

    @PreDestroy
    public void close() {
        client.close();
    }

    public Uni<JsonObject> invokeService() {
        return client
            .getAbs("https://httpbin.org/json").send()
            .onItem().transform(response -> {
                if (response.statusCode() == 200) {
                    return response.bodyAsJsonObject();
                } else {
                    return new JsonObject()
                            .put("error", response.statusMessage());
                }
            });
    }
}
```

正如您所见，您需要自己创建并关闭 `WebClient`。它暴露了 Mutiny API，因此它完美地集成在您的 Quarkus 应用程序中。

HTTP 反应式连接器集成了反应式消息传递（见 第十章），允许为每条消息发送 HTTP 请求。当您设计一个消息处理流水线，其中出站是 HTTP 端点时，这非常方便。它处理背压并控制并发量（正在处理的请求数量），但不允许处理响应。

要使用此 HTTP 连接器，您需要 示例 12-3 中显示的依赖项。

##### 示例 12-3\. HTTP 连接器的依赖项

```java
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-reactive-messaging-http</artifactId>
</dependency>
```

然后，您可以按照 示例 12-4 中所示配置连接器。

##### 示例 12-4\. 使用 HTTP 连接器通过 HTTP POST 请求发送消息

```java
mp.messaging.outgoing.my-http-endpoint.connector=quarkus-http ![1](img/1.png)
mp.messaging.outgoing.my-http-endpoint.method=POST ![2](img/2.png)
mp.messaging.outgoing.my-http-endpoint.url=https://httpbin.org/anything ![3](img/3.png)
```

![1](img/#co_reactive_rest_client__connecting___span_class__keep_together__with_http_endpoints__span__CO1-1)

指示 Quarkus 使用 `quarkus-http` 连接器管理 `my-http-endpoint` 通道。

![2](img/#co_reactive_rest_client__connecting___span_class__keep_together__with_http_endpoints__span__CO1-2)

配置要使用的 HTTP 方法。

![3](img/#co_reactive_rest_client__connecting___span_class__keep_together__with_http_endpoints__span__CO1-3)

配置要调用的服务的 URL。

默认情况下，连接器将接收到的消息编码为 JSON。你还可以配置自定义序列化器。连接器还支持重试和超时，但正如前面所述，不允许处理响应。任何非 2*XX* HTTP 响应都被视为失败，并将 NACK 消息。

REST 客户端提供了一种声明性的方式来调用 HTTP 服务。它实现了[MicroProfile REST Client 规范](https://oreil.ly/dX0Lv)。你无需处理 HTTP 请求和响应，只需将 HTTP API 映射到一个 Java 接口中。通过几个注解，你可以表达如何发送请求和处理响应。客户端使用与服务器端相同的 JAX-RS 注解；详见示例 12-5。

##### 示例 12-5\. REST 客户端示例

```java
@Path("/v2")
@RegisterRestClient
public interface CountriesService {

    @GET
    @Path("/name/{name}")
    Set<Country> getByName(@PathParam String name);
}
```

应用程序直接使用接口中的方法。本章的其余部分重点介绍 REST 客户端以及如何在响应式应用程序中集成它，包括如何优雅地处理故障。

# 响应式 REST 客户端

我们需要小心。许多*异步* HTTP 和 REST 客户端并未使用非阻塞 I/O，而是将 HTTP 请求委托给内部线程池。但这不适用于 Quarkus 的响应式 REST 客户端。它依赖于 Quarkus 的响应式架构。请注意，尽管它是响应式的，但你仍然可以以阻塞的方式使用它。与大多数 Quarkus 特性一样，你可以选择使用方式。即使你决定使用阻塞方式，Quarkus 仍会继续使用 I/O 线程，并将调用委托给应用程序的工作线程。

要在 Quarkus 中使用响应式 REST 客户端，请将依赖项添加到示例 12-6 所述的项目中。

##### 示例 12-6\. 用于响应式 REST 客户端的依赖（*chapter-12/rest-client-example/pom.xml*）

```java
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-rest-client-reactive</artifactId>
</dependency>
```

如果计划使用 JSON（在使用 HTTP 服务时通常如此），还需在示例 12-7 中添加依赖项。

##### 示例 12-7\. 用于支持响应式 REST 客户端的 Jackson 依赖（*chapter-12/rest-client-example/pom.xml*）

```java
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-rest-client-reactive-jackson</artifactId>
</dependency>
```

此依赖项增加了将请求体序列化为 JSON 并将 JSON 载荷反序列化为对象的能力。

## 将 HTTP API 映射到 Java 接口

REST 客户端的基石是一个 Java 接口，代表被消耗的 HTTP 端点。这个接口表示您的应用程序消耗的 HTTP API。它充当一个门面，使您的应用程序避免直接处理 HTTP。在接口的每个方法上，您使用 JAX-RS 注解来描述如何处理 HTTP 请求和响应。让我们考虑一个例子。想象一下，您需要集成[`httpbin`](https://httpbin.org)服务。要调用`/uuid`端点（返回 UUID），您需要创建一个 Java 接口，该方法代表调用；参见示例 12-8。

##### 示例 12-8。`httpbin`服务的 REST 客户端（*chapter-12/rest-client-example/src/main/java/org/acme/restclient/HttpApi.java*）

```java
@RegisterRestClient(configKey = "httpbin")
public interface HttpBinService {

    @GET
    @Path("/uuid")
    String getUUID();
}
```

这个接口的第一个重要事实是`@RegisterRestClient`注解。它表示这个接口代表一个 HTTP 端点。`configKey`属性定义了我们将用来配置 HTTP 端点的键，比如位置。

目前，这个接口只有一个方法：`getUUID`。当应用程序调用此方法时，它发送一个`GET`请求到`/uuid`，等待响应，并读取响应主体作为`String`。我们使用`@GET`和`@Path`注解定义了这种行为。

让我们通过使用另一种 HTTP 方法添加一个方法，如示例 12-9 所示。

##### 示例 12-9。发送 JSON 负载

```java
class Person {
    public String name;
}

@POST
@Path("/anything")
String anything(Person someone);
```

调用此方法将发送一个带有 JSON 载荷的`POST`请求到`/anything`。响应式 REST 客户端将`Person`的实例映射到 JSON。您还可以使用`@Consume`注解来显式设置`content-type`。

REST 客户端还允许您配置请求标头。这些标头可以是常量，并且可以通过使用`@HeaderParam`注解作为方法参数传递；参见示例 12-10。

##### 示例 12-10。传递头信息

```java
@POST
@Path("/anything")
@ClientHeaderParam(name = "X-header", value = "constant value")
String anythingWithConstantHeader(Person someone);

@POST
@Path("/anything")
String anythingWithHeader(Person someone,
                          @HeaderParam("X-header") String value);
```

要传递查询参数，请使用`@QueryParameter`注解（示例 12-11）。

##### 示例 12-11。传递查询参数

```java
@POST
@Path("/anything")
String anythingWithQuery(Person someone,
                         @QueryParam("param1") String p1,
                         @QueryParam("param2") String p2);
```

现在让我们来看看响应。到目前为止，我们只使用了`String`，但是 REST 客户端可以将响应映射到对象。如果我们回到最初的例子（`/uuid`），它返回一个具有一个字段`uuid`的 JSON 对象。我们可以将此响应映射到一个对象，如示例 12-12 所示。

##### 示例 12-12。接收 JSON 对象

```java
class UUID {
    public String uuid;
}

@GET
@Path("/uuid")
UUID uuid();
```

默认情况下，REST 客户端使用 JSON 将响应映射到对象。您可以使用`@Produces`注解来配置`Accept`头。

如果您想自己处理 HTTP 响应以检索状态代码或头信息，可以返回一个`Response`，如示例 12-13 所示。

##### 示例 12-13。使用响应

```java
@GET
@Path("/uuid")
Response getResponse();
```

它与“处理故障和自定义响应”中的`Response`相同。

将 HTTP API 映射到 Java 接口引入了一个清晰的契约，并避免了在应用程序代码中到处使用 HTTP 代码。它还提高了可测试性，因为你可以快速模拟远程服务。最后，请注意，你不必映射远程服务的所有端点，只需映射你在应用程序中使用的端点即可。

## 调用服务

要在应用程序中使用我们在前一节中定义的接口，我们需要执行以下步骤：

1.  在应用程序代码中注入 REST 客户端。

1.  配置服务的 URL。

第一步很简单；我们只需要像 示例 12-14 中所示注入客户端即可。

##### 示例 12-14\. 使用 HTTPBin REST 客户端 (*chapter-12/rest-client-example/src/main/java/org/acme/restclient/HttpEndpoint.java*)

```java
@Inject
@RestClient HttpBinService service;

public HttpBinService.UUID invoke() {
    return service.uuid();
}
```

`@RestClient` 限定符表示注入的对象是一个 REST 客户端。一旦它被注入，你就可以使用你在接口上定义的任何方法。

要配置客户端，打开 *application.properties* 文件并添加以下属性：

```java
httpbin/mp-rest/url=https://httpbin.org
```

`httpbin` 部分是在 `@RegisterRestClient` 接口中使用的配置键。在这里我们只配置了位置，但你可以[配置更多](https://oreil.ly/rD1tL)。

当然，URL 也可以在运行时通过使用 `httpbin/mp-rest/url` 系统属性或 `HTTPBIN_MP_REST_URL` 环境属性传递。

# 阻塞和非阻塞

如前所述，Quarkus 的响应式 REST 客户端支持即时（阻塞）方法和反应式（非阻塞）方法。通过返回类型来区分。在 “将 HTTP API 映射到 Java 接口” 中，我们所有的返回类型都是同步的。因此，Quarkus 会阻塞调用线程，直到接收到 HTTP 响应。这在响应式方面不是很好。幸运的是，你可以通过将返回类型更改为 `Uni` 来避免这种情况，就像 示例 12-15 中所示的那样。

##### 示例 12-15\. 在 REST 客户端接口中使用 Mutiny

```java
@RegisterRestClient(configKey = "reactive-httpbin")
public interface ReactiveHttpBinService {

    @GET
    @Path("/uuid")
    Uni<String> getUUID();

    class Person {
        public String name;
    }

    @POST
    @Path("/anything")
    Uni<String> anything(Person someone);

    class UUID {
        public String uuid;
    }

    @GET
    @Path("/uuid")
    Uni<UUID> uuid();

    @GET
    @Path("/uuid")
    Uni<Response> getResponse();

}
```

通过返回 `Uni` 而不是直接的结果类型，你指示 REST 客户端不要阻塞调用线程。更好的是，它将使用当前的 I/O 线程，采用异步执行模型，并避免额外的线程切换。

在消费者端，你只需使用 `Uni` 并将响应的处理附加到你的管道中（示例 12-16）。

##### 示例 12-16\. 使用 REST 客户端的响应式 API

```java
@Inject
@RestClient ReactiveHttpBinService service;

public void invoke() {
    service.uuid()
            .onItem().transform(u -> u.uuid)
            .subscribe().with(
                    s -> System.out.println("Received " + s),
                    f -> System.out.println("Failed with " + f)
    );
}
```

在这个例子中，我们自己处理订阅。不要忘记，你通常依赖于 Quarkus 来处理这个。例如，从 RESTEasy 反应式端点返回 `Uni` 将订阅返回的 `Uni`。

# 处理失败

如果你查看 示例 12-16，你会发现调用 REST 客户端可能会导致失败，你需要处理它。Quarkus 提供多种方法来优雅地处理失败。你可以使用 Mutiny API 处理失败，执行重试或优雅地恢复，如 第七章 所示。此外，Quarkus 提供了一种声明性的方法来表达如何处理失败。当集成远程系统（如 REST 客户端）时，这种方法特别方便，因为它将集成点和失败管理结合在一个位置。

`quarkus-smallrye-fault-tolerance` 扩展提供了一组注解来配置：

+   回退方法

+   重试

+   断路器

+   隔离舱

`quarkus-smallrye-fault-tolerance` 适用于命令式和响应式 API。在本节中，我们仅关注后者。

首先，要使用容错扩展，在你的项目中添加依赖项 示例 12-17。

##### 示例 12-17\. 容错支持的依赖项（*chapter-12/api-gateway-example/api-gateway/pom.xml*）

```java
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-smallrye-fault-tolerance</artifactId>
</dependency>
```

## 回退

*回退* 是在原始调用失败时提供替代结果的方法。让我们重用之前看过的示例，再次在 示例 12-18 中展示。

##### 示例 12-18\. `uuid` 方法

```java
@GET
@Path("/uuid")
Uni<UUID> uuid();
```

如果与远程服务的交互失败，我们可以生成一个回退（本地）UUID，如示例 12-19 所示。

##### 示例 12-19\. 声明 `uuid` 方法的回退

```java
import io.smallrye.mutiny.Uni;
import org.eclipse.microprofile.faulttolerance.Fallback;
import org.eclipse.microprofile.rest.client.inject.RegisterRestClient;

import javax.ws.rs.GET;
import javax.ws.rs.Path;

@RegisterRestClient(configKey = "reactive-httpbin")
public interface ReactiveHttpBinServiceWithFallbackMethod {

    class UUID {
        public String uuid;
    }

    @GET
    @Path("/uuid")
    @Fallback(fallbackMethod = "fallback")
    Uni<UUID> uuid();

    default Uni<UUID> fallback() {
        UUID u = new UUID();
        u.uuid = java.util.UUID.randomUUID().toString();
        return Uni.createFrom().item(u);
    }

}
```

`@Fallback` 注解指示调用的方法名称。此方法必须与原始方法具有相同的签名。因此，在我们的情况下，它必须返回 `Uni<UUID>`。我们的回退实现很简单，但你可以想象更复杂的场景，如调用备用服务。

如果你需要更多对回退的控制，还可以提供 `FallbackHandler`；参见 示例 12-20。

##### 示例 12-20\. 使用回退处理程序

```java
import io.smallrye.common.annotation.NonBlocking;
import io.smallrye.mutiny.Uni;
import org.eclipse.microprofile.faulttolerance.ExecutionContext;
import org.eclipse.microprofile.faulttolerance.Fallback;
import org.eclipse.microprofile.faulttolerance.FallbackHandler;
import org.eclipse.microprofile.rest.client.inject.RegisterRestClient;

import javax.ws.rs.GET;
import javax.ws.rs.Path;

@RegisterRestClient(configKey = "reactive-httpbin")
public interface ReactiveHttpBinServiceWithFallbackHandler {

    class UUID {
        public String uuid;
    }

    @GET
    @Path("/uuid")
    @Fallback(value = MyFallbackHandler.class)
    Uni<UUID> uuid();

    class MyFallbackHandler implements FallbackHandler<Uni<UUID>> {

        @Override
        public Uni<UUID> handle(ExecutionContext context) {
            UUID u = new UUID();
            u.uuid = java.util.UUID.randomUUID().toString();
            return Uni.createFrom().item(u);
        }
    }

}
```

配置的回退处理程序通过封装原始方法和异常的 `ExecutionContext` 被调用。

## 重试

回退允许以本地值优雅地恢复。容错扩展还允许重试。请记住，只有在调用服务是幂等的情况下才应使用重试功能。因此，在使用此功能之前，请确保它不会损害你的系统。

`@Retry` 注解指示 Quarkus 多次重试调用，希望下一次调用会成功（示例 12-21）。

##### 示例 12-21\. 声明重试策略

```java
@GET
@Path("/uuid")
@Retry(maxRetries = 10, delay = 1000, jitter = 100)
Uni<UUID> uuid();
```

正如你所见，你可以配置重试次数，以及延迟和抖动，以避免立即重试。你可以结合 `@Retry` 和 `@Fallback`，如果所有尝试都失败，调用回退方法。

## 超时

快速失败总比让用户长时间等待好。`@Timeout`注解强制调用在及时完成；参见示例 12-22。

##### 示例 12-22\. 配置超时

```java
@GET
@Path("/uuid")
@Timeout(value = 3, unit = ChronoUnit.SECONDS)
Uni<UUID> uuid();
```

如果调用在配置的超时时间内未产生结果，则调用失败。将`@Timeout`与`@Fallback`结合使用允许在原始调用无法按预期时间完成时使用备用结果。

## 舱壁和断路器

Quarkus 的容错特性还提供了断路器和舱壁功能。虽然其他模式保护您的应用免受远程故障和缓慢的影响，但这两个模式避免了对不健康或脆弱服务的过度调用。

第一个模式几年前由[Hystrix](https://oreil.ly/eLHn0)或[Resilience4j](https://oreil.ly/yr4dO)等库广泛推广，它检测到失败的服务并给予其恢复的时间（而不是不断地调用它们）。断路器允许您的应用立即失败，以防止可能会失败的重复调用。断路器的操作类似于电气断路器。闭合电路表示完全功能的系统，开放电路意味着不完整的系统。如果发生故障，断路器将触发打开电路，从系统中删除故障点。

软件断路器有一个额外的状态：半开放状态。在断路器打开后，它周期性地转换到半开放状态。它检查失败的组件是否已恢复，并在被认为安全和功能正常后关闭电路。在 Quarkus 中使用断路器，只需使用`@CircuitBreaker`注解，如示例 12-23 所示。

##### 示例 12-23\. 使用断路器

```java
@GET
@Path("/uuid")
@CircuitBreaker
Uni<UUID> uuid();
```

您还可以配置断路器；参见示例 12-24。

##### 示例 12-24\. 配置断路器

```java
@GET
@Path("/uuid")
@CircuitBreaker(
    // Delay before switching to the half-open state
    delay = 10, delayUnit = ChronoUnit.SECONDS,
    // The number of successful executions,
    // before a half-open circuit is closed again
    successThreshold = 2,
    // The ratio of failures within the rolling
    // window that will trip the circuit to open
    failureRatio = 0.75,
    // The number of consecutive requests in a
    // rolling window
    requestVolumeThreshold = 10
)
Uni<UUID> uuidWithConfiguredCircuitBreaker();
```

通过断路器保护您的集成点，不仅可以防止应用程序中的响应缓慢和故障，还可以让故障服务有时间恢复。当您与的服务处于维护或高负载状态时，使用断路器非常方便。

舱壁模式的理念是限制故障的传播。您保护应用程序免受远程服务中的故障，并避免将其级联到整个系统中，从而导致广泛的问题。`@Bulkhead`注解限制并发请求的数量，并节省无响应的远程服务浪费系统资源的情况。

使用该注解可帮助您避免处理过多的失败，并避免向远程服务发送大量并发请求。 后者很重要。 由于反应式原则使高并发成为可能，您永远不应忘记您调用的服务可能无法承受这种负载。 因此，使用 `@Bulkhead` 注解允许控制出站并发性。 是的，这会增加您的响应时间并减少您的并发性，但这是为了全局系统状态的利益。

您可以使用最大并发性 (`value`) 和等待轮次的最大请求数来配置批处理隔离，如 示例 12-25 所示。 如果队列已满，它将立即拒绝任何新的调用，避免响应时间过慢。

##### 示例 12-25\. 声明一个批处理隔离

```java
@GET
@Path("/uuid")
@Bulkhead(value = 5, waitingTaskQueue = 1000)
Uni<UUID> uuid();
```

# 使用 RESTEasy Reactive 客户端构建 API 网关

在 第八章 中，我们展示了如何实现依赖非阻塞 I/O 的 HTTP 端点，以及此实现如何使用 Mutiny（在 第七章 中介绍）来编排异步任务并避免阻塞。 此方法实现了高并发和有效的资源使用。 这是构建 API 网关的完美组合。

*API 网关* 是位于一组后端服务前面的服务。 网关处理外部请求并编排其他服务。 例如，它可以将请求委托给单个服务，并实现涉及多个后端服务的更复杂的流程。

通过结合 Mutiny、RESTEasy Reactive、反应式 REST 客户端和容错注解，我们可以构建高并发、响应迅速、弹性的 API 网关。 在本节中，我们通过探索一个示例来解释实现这样一个网关的基础知识。 我们的系统由三个应用程序组成：

问候服务

暴露一个 HTTP API 返回 `Hello *{name}*`，其中名称是查询参数。

报价服务

暴露另一个 HTTP API 返回有关咖啡的随机有趣引用。

API 网关

将 HTTP API 暴露，将 `/quote` 上的请求委托给报价服务。 发送到 `/` 的请求将调用问候和报价服务，并构建一个封装两者的 JSON 对象。

您可以在 *chapter-12/api-gateway-example* 目录中找到系统的完整代码。 本节重点介绍 API 网关组件。 我们在这里不涵盖问候和报价服务的代码，因为它们很简单。 这些组件的源代码位于 *chapter-12/api-gateway-example/greeting-service* 和 *chapter-12/api-gateway-example/quote-service* 目录中。 您可以在 *chapter-12/api-gateway-example/api-gateway* 中找到的 API 网关应用程序更有趣，我们一起来探索它。

首先，让我们构建并运行此系统。 要构建此示例，请在终端中导航到 *chapter-12/api-gateway-example*，然后运行 `mvn package`。 然后您将需要三个终端：

1.  在第一个中，从*chapter-12/api-gateway-example/greeting-service*目录运行`java -jar target/quarkus-app/quarkus-run.jar`。

1.  在第二个中，从*chapter-12/api-gateway-example/quote-service*目录运行`java -jar target/quarkus-app/quarkus-run.jar`。

1.  最后，在第三个中，从*chapter-12/api-gateway-example/api-gateway*目录运行`java -jar target/quarkus-app/quarkus-run.jar`。

###### 注意

确保你的系统上没有使用端口 9010（问候服务）和 9020（报价服务）。API 网关使用端口 8080。

一旦所有服务都运行起来，你可以使用`curl`来调用 API 网关，它会编排其他后端服务（示例 12-26）。

##### 示例 12-26\. 调用问候端点

```java
> curl http://localhost:8080/
{"greeting":"Hello anonymous","quote":"I never drink coffee
at lunch. I find it keeps me awake for the afternoon."}
```

API 网关的核心是`Gateway`类，如示例 12-27 所示。

##### 示例 12-27\. `Gateway`类（*chapter-12/api-gateway-example/api-gateway/src/main/java/org/acme/gateway/Gateway.java*）

```java
package org.acme.gateway;

import io.smallrye.mutiny.Uni;
import org.eclipse.microprofile.rest.client.inject.RestClient;

import javax.ws.rs.DefaultValue;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.QueryParam;

@Path("/")
public class Gateway {

    @RestClient
    GreetingService greetingService;

    @RestClient
    QuoteService quoteService;

    @GET
    @Path("/quote")
    public Uni<String> getQuote() {
        return quoteService.getQuote();
    }

    @GET
    @Path("/")
    public Uni<Greeting> getBoth(
            @QueryParam("name")
            @DefaultValue("anonymous") String name) {
        Uni<String> greeting = greetingService.greeting(name);
        Uni<String> quote = quoteService.getQuote()
                .onFailure()
                    .recoverWithItem("No coffee - no quote");

        return Uni.combine().all().unis(greeting, quote).asTuple()
                .onItem().transform(tuple ->
                        new Greeting(tuple.getItem1(),
                                tuple.getItem2())
                );
    }

    public static class Greeting {
        public final String greeting;
        public final String quote;

        public Greeting(String greeting, String quote) {
            this.greeting = greeting;
            this.quote = quote;
        }
    }
}
```

`Gateway`类检索两个 REST 客户端并使用它们处理 HTTP 请求。由于 API 网关可能会承受大量负载和高并发，我们使用 RESTEasy Reactive 及其 Mutiny 集成，因此不需要工作线程。

`getQuote`方法很简单。它将调用委托给`QuoteService`。

`getBoth`方法更加有趣。它需要调用两个服务并聚合响应。由于这两个服务是无关的，我们可以同时调用它们。正如你在第七章中看到的，通过 Mutiny 的`Uni.combine`结构可以轻松实现这一点。一旦我们将两个响应封装在元组中，我们就构建`Greeting`结构并发出它。

让我们看看 REST 客户端。`GreetingService`使用容错注解以确保我们适当处理失败或慢响应；参见示例 12-28。

##### 示例 12-28\. `GreetingService` REST 客户端（*chapter-12/api-gateway-example/api-gateway/src/main/java/org/acme/gateway/GreetingService.java*）

```java
package org.acme.gateway;

import io.smallrye.mutiny.Uni;
import org.eclipse.microprofile.faulttolerance.*;
import org.eclipse.microprofile.rest.client.inject.RegisterRestClient;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.QueryParam;

@RegisterRestClient(configKey = "greeting-service")
public interface GreetingService {

    @GET
    @Path("/")
    @CircuitBreaker
    @Timeout(2000)
    @Fallback(GreetingFallback.class)
    Uni<String> greeting(@QueryParam("name") String name);

    class GreetingFallback implements FallbackHandler<Uni<String>> {
        @Override
        public Uni<String> handle(ExecutionContext context) {
            return Uni.createFrom().item("Hello fallback");
        }
    }
}
```

注意问候方法结合了断路器、超时和回退注解。`QuoteService`类似，但不使用回退注解，正如你在示例 12-29 中所见。

##### 示例 12-29\. `QuoteService` REST 客户端（*chapter-12/api-gateway-example/api-gateway/src/main/java/org/acme/gateway/QuoteService.java*）

```java
package org.acme.gateway;

import io.smallrye.mutiny.Uni;
import org.eclipse.microprofile.faulttolerance.CircuitBreaker;
import org.eclipse.microprofile.faulttolerance.Timeout;
import org.eclipse.microprofile.rest.client.inject.RegisterRestClient;

import javax.ws.rs.GET;
import javax.ws.rs.Path;

@RegisterRestClient(configKey = "quote-service")
public interface QuoteService {

    @GET
    @Path("/")
    @CircuitBreaker
    @Timeout(2000)
    Uni<String> getQuote();

}
```

相反，正如你可能已经注意到的那样，我们在`Gateway`类中使用 Mutiny API 来处理失败（示例 12-30）。

##### 示例 12-30\. 使用 Mutiny API 从失败中恢复（*chapter-12/api-gateway-example/api-gateway/src/main/java/org/acme/gateway/Gateway.java*）

```java
Uni<String> quote = quoteService.getQuote()
   .onFailure().recoverWithItem("No coffee - no quote");
```

通常情况下，我们在使用容错注解或使用 Mutiny API 之间进行选择，我们想要强调您可以选择并且可以轻松地将两者结合起来。然而，`getQuote` 方法不处理失败并传播错误。因此，在使用 Mutiny 处理失败时，请确保覆盖所有入口点。现在，如果停止报价服务（通过在第二个终端中按 Ctrl-C），您将在 Example 12-31 中获得输出。

##### 示例 12-31\. 调用端点

```java
> curl http://localhost:8080\?name\=luke
{"greeting":"Hello luke","quote":"No coffee - no quote"}
```

如果您也停止了问候服务，通过在第一个终端中按 Ctrl-C，您将获得 Example 12-32。

##### 示例 12-32\. 在没有问候服务时调用问候端点

```java
> curl http://localhost:8080\?name\=luke
{"greeting":"Hello fallback","quote":"No coffee - no quote"}
```

本节探讨了构建高度并发 API 网关的基本构造。结合 RESTEasy Reactive、反应式 REST 客户端、容错和 Mutiny 提供了所有您需要向用户公开健壮 API 并优雅处理失败的功能。下一节说明了如何在消息应用程序中也可以使用 REST 客户端。

# 在消息应用程序中使用 REST 客户端

REST 客户端在消息应用程序中也很有用。例如，消息处理管道可以为每个消息调用远程 HTTP API，或将消息转发到远程 HTTP 端点。我们可以在使用 Reactive Messaging 建模的处理管道中使用 REST 客户端来实现这一点。在本节中，我们探讨了一个从 HTTP 端点接收简单订单、处理它们并将其持久化到数据库中的应用程序：

1.  `OrderEndpoint` 接收 `Order` 并将其发送到 `new-orders` 通道中。

1.  `OrderProcessing` 组件从 `new-order` 通道消费订单，并调用远程验证服务。如果订单验证成功，则将其发送到 `validated-orders` 通道。否则，将否定确认订单。

1.  `OrderStorage` 接收来自 `validated-orders` 通道的订单并将其存储在数据库中。

为了简单起见，验证端点在同一进程中运行，但调用仍然使用 REST 客户端。您可以在 *chapter-12/http-messaging-example* 找到应用程序的完整代码，但让我们快速浏览一下。

如您在 Example 12-33 中所见，`Order` 结构很简单。它仅包含产品名称和数量。请注意，`Order` 类是 Panache 实体。我们将使用它来在数据库中存储验证通过的订单。

##### 示例 12-33\. `Order` 结构 (*chapter-12/http-messaging-example/src/main/java/org/acme/http/model/Order.java*)

```java
package org.acme.http.model;

import io.quarkus.hibernate.reactive.panache.PanacheEntity;

import javax.persistence.Entity;
import javax.persistence.Table;

@Entity
@Table(name = "orders")
public class Order extends PanacheEntity {

    public String name;
    public int quantity;

}
```

`OrderEndpoint` 同样简单，如您在 Example 12-34 中所见。

##### 示例 12-34\. `OrderEndpoint` 类 (*chapter-12/http-messaging-example/src/main/java/org/acme/http/OrderEndpoint.java*)

```java
package org.acme.http;

import io.smallrye.mutiny.Multi;
import io.smallrye.mutiny.Uni;
import io.smallrye.reactive.messaging.MutinyEmitter;
import org.acme.http.model.Order;
import org.eclipse.microprofile.reactive.messaging.Channel;

import javax.ws.rs.GET;
import javax.ws.rs.POST;
import javax.ws.rs.Path;
import javax.ws.rs.core.Response;

@Path("/order")
public class OrderEndpoint {

    @Channel("new-orders")
    MutinyEmitter<Order> emitter;

    @POST
    public Uni<Response> order(Order order) {
        return emitter.send(order)
                .log()
                .onItem().transform(x -> Response.accepted().build())
                .onFailure().recoverWithItem(
                        Response.status(Response.Status.BAD_REQUEST)
                                .build()
                );
    }

    @GET
    public Multi<Order> getAllValidatedOrders() {
        return Order.streamAll();
    }

}
```

`order` 方法将收到的订单发送到 `new-orders` 通道。当消息被确认时，它会产生 `202 - ACCEPTED` 响应。如果消息被拒绝，它会创建一个 `400 - BAD REQUEST` 响应。

`getAllValidatedOrders` 方法允许我们检查数据库中的写入情况。`OrderProcessing` 组件从 `new-orders` 通道中消费订单，并调用验证服务，如示例 12-35 所示。

##### 示例 12-35\. `OrderProcessing` 类 (*chapter-12/http-messaging-example/src/main/java/org/acme/http/OrderProcessing.java*)

```java
package org.acme.http;

import io.smallrye.mutiny.Uni;
import org.acme.http.model.Order;
import org.eclipse.microprofile.reactive.messaging.Incoming;
import org.eclipse.microprofile.reactive.messaging.Outgoing;
import org.eclipse.microprofile.rest.client.inject.RestClient;

import javax.enterprise.context.ApplicationScoped;

@ApplicationScoped
public class OrderProcessing {

    @RestClient
    ValidationService validation;

    @Incoming("new-orders")
    @Outgoing("validated-orders")
    Uni<Order> validate(Order order) {
        return validation.validate(order)
                .onItem().transform(x -> order);
    }

}
```

如果订单无效，则验证服务失败。结果，消息被拒绝。如果订单有效，则它会将订单发送到 `validated-orders` 通道。`OrderStorage` 组件消费此通道，并将每个订单写入数据库（示例 12-36）。

##### 示例 12-36\. 在数据库中持久化订单（*chapter-12/http-messaging-example/src/main/java/org/acme/http/OrderStorage.java*)

```java
package org.acme.http;

import io.quarkus.hibernate.reactive.panache.Panache;
import io.smallrye.mutiny.Uni;
import org.acme.http.model.Order;
import org.eclipse.microprofile.reactive.messaging.Incoming;

import javax.enterprise.context.ApplicationScoped;

@ApplicationScoped
public class OrderStorage {

    @Incoming("validated-orders")
    public Uni<Void> store(Order order) {
        return Panache.withTransaction(order::persist)
                .replaceWithVoid();
    }

}
```

从 *chapter-12/http-messaging-example* 目录运行应用程序使用 `mvn quarkus:dev` 命令。由于它使用 Quarkus Dev 服务，在开发模式下不需要为数据库提供服务；Quarkus 会为您完成这一切。

应用程序运行后，按照示例 12-37 的方法添加一个订单。

##### 示例 12-37\. 调用端点以添加新订单

```java
> curl -v  --header "Content-Type: application/json" \
  --request POST \
  --data '{"name":"coffee", "quantity":2}' \
  http://localhost:8080/order
```

你可以通过使用示例 12-38 来验证订单是否已被处理。

##### 示例 12-38\. 调用端点以检索持久化的订单

```java
> curl http://localhost:8080/order
[{"id":1,"name":"coffee","quantity":2}]
```

现在尝试插入一个无效订单（使用负数量），如示例 12-39 所示。

##### 示例 12-39\. 调用端点以引入无效订单

```java
> curl -v  --header "Content-Type: application/json" \
  --request POST \
  --data '{"name":"book", "quantity":-1}' \
  http://localhost:8080/order
```

响应是 `400`。您可以使用示例 12-40 中的代码验证它是否未被插入到数据库中。

##### 示例 12-40\. 调用端点以检索持久化的订单：无效订单未列出

```java
> curl http://localhost:8080/order
[{"id":1,"name":"coffee","quantity":2}]
```

处理的验证步骤可以使用容错注解来提高应用程序的可靠性（示例 12-41）。

##### 示例 12-41\. 使用容错注解

```java
@Incoming("new-orders")
@Outgoing("validated-orders")
@Timeout(2000)
Uni<Order> validate(Order order) {
    return validation.validate(order)
            .onItem().transform(x -> order);
}
```

在消息应用程序中使用 REST 客户端可以让您在处理工作流程中顺利集成远程服务。

# 总结

本章重点介绍了如何将远程 HTTP API 集成到您的响应式应用程序中，而不会违反响应式原则。您学会了如何执行以下操作：

+   通过使用响应式 REST 客户端，将远程 HTTP 终端集成到您的响应式应用程序中

+   使用容错注解保护你的集成点

+   通过结合 Mutiny、RESTEasy Reactive 和 REST 客户端构建 API 网关

+   在消息处理管道中集成远程 HTTP API

下一章重点介绍可观察性，以及如何在负载波动和面对故障时保持响应式系统的运行。
