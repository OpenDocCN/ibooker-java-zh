# 第十三章 Quarkus REST 客户端

在第三章，您已经学习了如何开发 RESTful 服务，但在本章中，您将学习如何在 RESTful Web 服务之间进行通信。

使用任何基于服务的架构都意味着您需要与外部服务进行通信。这些服务可能是内部服务（您控制服务的生命周期，通常部署在同一集群中）或外部服务（第三方服务）。

如果这些服务实现为 RESTful Web 服务，则需要客户端与这些服务进行交互。Quarkus 提供了两种方法：JAX-RS Web 客户端，这是与 RESTful 服务通信的标准 Java EE 方式；以及 MicroProfile REST 客户端，这是与 RESTful 服务通信的新方式。

本章将包括以下示例：

+   使用 JAX-RS 客户端与其他 RESTful 服务进行通信

+   使用 MicroProfile Rest 客户端与其他 RESTful 服务进行通信

+   安全地在 RESTful 服务之间进行通信

# 13.1 使用 JAX-RS Web 客户端

## 问题

您想要与另一个 RESTful Web 服务进行通信。

## 解决方案

使用 JAX-RS Web 客户端与其他 RESTful Web 服务进行通信。

让我们看看如何使用 JAX-RS 规范与其他 RESTful 服务进行通信。

我们将要连接的外部服务是[世界时钟 API](https://oreil.ly/wl2IE)，它通过时区返回当前日期/时间。您需要获取由[API](https://oreil.ly/7M0tf)公开的当前日期/时间。

您需要添加扩展来使用 REST 客户端以及 JAX-B/Jackson 用于 JSON 和 Java 对象的编组/解组。

```java
./mvnw quarkus:add-extension -Dextensions="resteasy-jsonb, rest-client"
```

或者，如果您是从空目录创建，请运行以下命令：

```java
mvn io.quarkus:quarkus-maven-plugin:1.4.1.Final:create \
 -DprojectGroupId=org.acme.quickstart \
 -DprojectArtifactId=clock-app \
 -DclassName="org.acme.quickstart.WorldClockResource" \
 -Dextensions="resteasy-jsonb, rest-client"
 -Dpath="/now"
```

您可以开始使用 JAX-RS REST 客户端与外部 REST API 进行通信。让我们看看与世界时钟服务的交互是什么样子的。

打开 `org.acme.quickstart.WorldClockResource.java` 并添加以下代码：

```java
package org.acme.quickstart;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.client.Client;
import javax.ws.rs.client.ClientBuilder;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;

import org.eclipse.microprofile.config.inject.ConfigProperty;
import org.eclipse.microprofile.rest.client.inject.RestClient;

@Path("/now")
public class WorldClockResource {

  @ConfigProperty(name = "clock.host",
  defaultValue = "http://worldclockapi.com")
    String clockHost; ![1](img/1.png)

  private Client client = ClientBuilder.newClient(); ![2](img/2.png)

  @GET
  @Path("{timezone}")
  @Produces(MediaType.APPLICATION_JSON)
  public WorldClock getCurrentTime(@PathParam("timezone") String timezone) {
    WorldClock worldClock = client.target(clockHost) ![3](img/3.png)
      .path("api/json/{timezone}/now") ![4](img/4.png)
      .resolveTemplate("timezone", timezone) ![5](img/5.png)
      .request(MediaType.APPLICATION_JSON)
      .get(WorldClock.class); ![6](img/6.png) ![7](img/7.png)

    return worldClock;
  }
}
```

![1](img/#co_quarkus_rest_clients_CO1-1)

使服务主机可配置

![2](img/#co_quarkus_rest_clients_CO1-2)

创建新的 REST 客户端

![3](img/#co_quarkus_rest_clients_CO1-3)

设置主机

![4](img/#co_quarkus_rest_clients_CO1-4)

设置到服务的路径

![5](img/#co_quarkus_rest_clients_CO1-5)

将 `timezone` 占位符解析为所提供的时区

![6](img/#co_quarkus_rest_clients_CO1-6)

执行 `GET` HTTP 方法

![7](img/#co_quarkus_rest_clients_CO1-7)

将 JSON 输出转换为提供的 POJO

通过打开新的终端窗口，启动 Quarkus 应用程序，并发送请求到 `GET` 方法来尝试它：

```java
./mvnw clean compile quarkus:dev

curl localhost:8080/now/cet
{"currentDateTime":"2019-11-13T13:29+01:00","dayOfTheWeek":"Wednesday"}%
```

## 讨论

以类似的方式，您可以对其他 HTTP 方法发出请求。例如，要执行 POST 请求，您调用 `post` 方法：

```java
target(host)
    .request(MediaType.APPLICATION_JSON)
    .post(entity);
```

您还可以使用 `javax.ws.rs.core.Response` 来获取所有响应细节，而不仅仅是响应体：

```java
  @GET
  @Path("{timezone}/raw")
  @Produces(MediaType.APPLICATION_JSON)
  public Response getCurrentTimeResponse(@PathParam("timezone")
      String timezone) {
    javax.ws.rs.core.Response responseWorldClock = client.target(clockHost)
      .path("api/json/{timezone}/now")
      .resolveTemplate("timezone", timezone)
      .request(MediaType.APPLICATION_JSON)
      .get(Response.class);

    System.out.println(responseWorldClock.getStatus());
    System.out.println(responseWorldClock.getStringHeaders());
    // ... more methods

    return responseWorldClock;
  }
```

## 另请参阅

您可以进一步在 Oracle 网站上的以下页面探索 JAX-RS REST 客户端：

+   [使用 JAX-RS 客户端 API 访问 REST 资源](https://oreil.ly/7neQm)

+   [在 JAX-RS 示例应用程序中使用客户端 API](https://oreil.ly/QA8lA)

# 13.2 使用 MicroProfile REST 客户端

## 问题

您希望与另一个 RESTful Web 服务通信，而不需了解低级细节。

## 解决方案

使用 MicroProfile REST 客户端与其他 RESTful Web 服务通信。

到目前为止，您已经了解到如何使用 JAX-RS `Web Client` 与其他 REST API 进行通信，但这并不是类型安全的，并且需要处理低级参数而不是专注于消息通信。

MicroProfile REST 客户端尽可能地使用 JAX-RS 2.0 规范，提供了一种类型安全的方法来通过 HTTP 调用 RESTful 服务。 REST 客户端定义为 Java 接口，使其类型安全，并使用 JAX-RS 注解提供网络配置。

我们将在此处继续使用在前一节中使用的相同的世界时钟 API。 记得获取[当前日期/时间](https://oreil.ly/7M0tf)。

创建负责与外部服务交互的 `org.acme.quickstart.WorldClockService` 接口：

```java
package org.acme.quickstart;

import javax.enterprise.context.ApplicationScoped;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

import org.eclipse.microprofile.rest.client.inject.RegisterRestClient;

@Path("/api") ![1](img/1.png)
@ApplicationScoped
@RegisterRestClient ![2](img/2.png)
public interface WorldClockService {

    @GET ![3](img/3.png)
    @Path("/json/{timezone}/now") ![4](img/4.png)
    @Produces(MediaType.APPLICATION_JSON) ![5](img/5.png)
    WorldClock getNow(@PathParam("timezone") String timezone); ![6](img/6.png)

}
```

![1](img/#co_quarkus_rest_clients_CO2-1)

全局路径

![2](img/#co_quarkus_rest_clients_CO2-2)

将接口设置为 REST 客户端

![3](img/#co_quarkus_rest_clients_CO2-3)

请求使用 `GET` HTTP 方法

![4](img/#co_quarkus_rest_clients_CO2-4)

具有路径参数的子路径

![5](img/#co_quarkus_rest_clients_CO2-5)

请求的媒体类型

![6](img/#co_quarkus_rest_clients_CO2-6)

使用传递的参数解析路径参数

打开 `org.acme.quickstart.WorldClockResource.java` 并添加以下代码：

```java
@RestClient ![1](img/1.png)
WorldClockService worldClockService;

@GET
@Path("{timezone}/mp")
@Produces(MediaType.APPLICATION_JSON)
public WorldClock getCurrentTimeMp(@PathParam("timezone") String timezone) {
  return worldClockService.getNow(timezone); ![2](img/2.png)
}
```

![1](img/#co_quarkus_rest_clients_CO3-1)

注入 REST 客户端

![2](img/#co_quarkus_rest_clients_CO3-2)

调用外部服务

您仍然需要设置外部服务的主机。 MicroProfile REST 客户端有一个配置属性用于设置它。

打开 *application.properties*：

```java
org.acme.quickstart.WorldClockService/mp-rest/url=http://worldclockapi.com
```

属性名称使用以下格式：`*fully_qualified_name_rest_client*/mp-rest/url`，其值为主机名（或 URL 的根）：

```java
./mvnw clean compile quarkus:dev

curl localhost:8080/now/cet/mp
{"currentDateTime":"2019-11-13T16:46+01:00","dayOfTheWeek":"Wednesday"}%
```

## 讨论

通过实现 `org.eclipse.microprofile.rest.client.ex.ResponseExceptionMapper` 接口，您还可以将状态码大于或等于 400 的响应转换为异常。 如果注册了多个映射器，则需要使用 `javax.annotation.Priority` 注解设置优先级。

创建以下 `ResponseExecptionMapper` 类以注册它，并使应用程序对 400 状态码抛出 `IOExceptions`：

```java
package org.acme.quickstart;

import java.io.IOException;
import javax.ws.rs.core.MultivaluedMap;
import javax.ws.rs.core.Response;
import org.eclipse.microprofile.rest.client.ext.ResponseExceptionMapper;

public class CustomResponseExceptionMapper
                implements ResponseExceptionMapper<IOException> { ![1](img/1.png)

    @Override
    public IOException toThrowable(Response response) { ![2](img/2.png)
        return new IOException();
    }

    @Override
    public boolean handles(int status,
                            MultivaluedMap<String, Object> headers) { ![3](img/3.png)
        return status >= 400 && status < 500;
    }

}
```

![1](img/#co_quarkus_rest_clients_CO4-1)

实现映射器接口

![2](img/#co_quarkus_rest_clients_CO4-2)

将响应转换为异常

![3](img/#co_quarkus_rest_clients_CO4-3)

默认情况下，会将状态码 ≥ 400 的任何响应转换，但您可以覆盖该方法以提供更小的范围。

`ResponseExceptionMapper` 是专门从 MicroProfile REST Client 规范中的扩展点，但您也可以使用 JAX-RS 规范提供的扩展模型：

`ClientRequestFilter`

调用请求发送到外部服务时触发的过滤器。

`ClientResponseFilter`

从外部服务接收到响应时调用的过滤器。

`MessageBodyReader`

调用后读取实体。

`MessageBodyWriter`

在支持主体的操作中编写请求主体。

`ParamConverter`

将资源中的参数转换为请求或响应中使用的格式。

`ReadInterceptor`

当从外部服务接收到响应时触发的监听器。

`WriteInterceptor`

请求发送到外部服务时触发的监听器。

您还可以使用`@InjectMock`与`@RestClient`一起模拟`WorldClockService`接口：

```java
@InjectMock
@RestClient
WorldClockService worldClockService;
```

## 参见

可以在以下网站找到 MicroProfile Rest Client 规范：

+   [Eclipse MicroProfile 的 REST 客户端](https://oreil.ly/7D0Zv)

# 13.3 实现 CRUD 客户端

## 问题

您希望与另一个提供 CRUD 操作的 RESTful web 服务通信。

## 解决方案

使用 MicroProfile REST Client 和 JAX-RS 注解实现 CRUD 客户端。

到目前为止，您已经看到如何使用 MicroProfile REST Client 从外部服务获取信息。当服务是内部服务时，通常需要实现更多操作，如插入、删除或更新。

要实现这些操作，可以在 MicroProfile REST Client 上使用 JAX-RS 注解。让我们看一个例子：

```java
package org.acme.quickstart;

import java.util.List;

import javax.ws.rs.BeanParam;
import javax.ws.rs.Consumes;
import javax.ws.rs.CookieParam;
import javax.ws.rs.DELETE;
import javax.ws.rs.GET;
import javax.ws.rs.HEAD;
import javax.ws.rs.HeaderParam;
import javax.ws.rs.POST;
import javax.ws.rs.PUT;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.core.Response;

import org.eclipse.microprofile.rest.client.inject.RegisterRestClient;

@Path("/developer")
@RegisterRestClient
@Consumes("application/json") ![1](img/1.png)
@Produces("application/json")
public interface DeveloperService {

  @HEAD ![2](img/2.png)
  Response head();

  @GET
  List<Developer> getDevelopers();

  @POST ![3](img/3.png)
  Response createDeveloper(
      @HeaderParam("Authorization") String authorization, ![4](img/4.png)
      Developer developer); ![5](img/5.png)

  @DELETE ![6](img/6.png)
  @Path("/{userId}")
  Response deleteUser(@CookieParam("AuthToken") String authorization, ![7](img/7.png)
      @PathParam("developerId") Long developerId);
}
```

![1](img/#co_quarkus_rest_clients_CO5-1)

请求和响应以 JSON 格式。

![2](img/#co_quarkus_rest_clients_CO5-2)

使用`HEAD` HTTP 方法。

![3](img/#co_quarkus_rest_clients_CO5-3)

使用`POST` HTTP 方法。

![4](img/#co_quarkus_rest_clients_CO5-4)

设置`Authorization` 头

![5](img/#co_quarkus_rest_clients_CO5-5)

开发人员内容作为主体发送

![6](img/#co_quarkus_rest_clients_CO5-6)

使用`DELETE` HTTP 方法。

![7](img/#co_quarkus_rest_clients_CO5-7)

设置`AuthToken` cookie

## 讨论

注意 JAX-RS 注解如何用于配置请求如何发送到其他服务。您无需编写任何程序代码。

这种方法对开发人员友好，有助于减少使用 JAX-RS Web 客户端时可能使用的样板代码。

当然，它也有一些缺点。例如，方法可能包含大量参数，因为有许多路径参数、要设置的标头和 cookie。为了解决此问题，请传递一个带有所有必需字段的 POJO（而不是在方法中设置它们）。

让我们为`PUT`需求创建一个 Java 类（即，授权头和路径参数）：

```java
package org.acme.quickstart;

import javax.ws.rs.HeaderParam;
import javax.ws.rs.PathParam;

public class PutDeveloper {

    @HeaderParam("Authorization") ![1](img/1.png)
    private String authorization;

    @PathParam("developerId") ![2](img/2.png)
    private String developerId;

    public String getAuthorization() {
        return authorization;
    }

    public void setAuthorization(String authorization) {
        this.authorization = authorization;
    }

    public String getDeveloperId() {
        return developerId;
    }

    public void setDeveloperId(String developerId) {
        this.developerId = developerId;
    }

}
```

![1](img/#co_quarkus_rest_clients_CO6-1)

设置`Authorization`头部

![2](img/#co_quarkus_rest_clients_CO6-2)

设置解析的`path`参数

使用前述类的`interface`方法如下：

```java
@PUT
@Path("/{developerId}")
Response updateUser(@BeanParam PutDeveloper putDeveloper, ![1](img/1.png)
    Developer developer);
```

![1](img/#co_quarkus_rest_clients_CO7-1)

`BeanParam`用于指示此类是*参数聚合器*

# 13.4 操纵头部

## 问题

您希望从传入请求中操纵和传播头部到传出服务（服务到服务认证）。

## 解决方案

使用 MicroProfile REST Client 功能操作头部。

当您需要与其他 RESTful Web 服务进行通信时，可能需要将一些请求头传递给传出/下游服务。其中一个典型情况是通过`Authorization`头部进行服务间认证。服务架构中的认证和授权通常通过传播令牌来解决，通常是 JWT 令牌，通过应用程序中所有服务进行传递。您可以在图 13-1 中看到这个想法。

![qucb 1301](img/qucb_1301.png)

###### 图 13-1\. 服务到服务认证

MicroProfile REST Client 通过允许您使用注解在静态级别或通过实现`ClientHeadersFactory`接口在编程级别来传播和操作头部，简化了所有这些操作。

要在接口定义的一个方法或所有方法上设置头部，您可以使用`org.eclipse.microprofile.rest.client.annotation.ClientHeaderParam`注解在方法级别或类级别设置具有静态值的头部：

```java
@Path("/somePath")
@ClientHeaderParam(name="user-agent", value="curl/7.54.0") ![1](img/1.png)
Response get();
```

![1](img/#co_quarkus_rest_clients_CO8-1)

将`user-agent`设置为请求

`value`可以是方法调用，其中返回值将是头部的值：

```java
@ClientHeaderParam(name="user-agent", value="{determineHeaderValue}") ![1](img/1.png)
Response otherGet();

default String determineHeaderValue(String headerName) { ![2](img/2.png)
    return "Hi-" + headerName;
}
```

![1](img/#co_quarkus_rest_clients_CO9-1)

设置要调用的方法

![2](img/#co_quarkus_rest_clients_CO9-2)

头部名称是方法的第一个参数

这些方法提供了基本的头部操作，但不会帮助传播从传入请求到传出服务的头部。还可以通过实现`ClientHeadersFactory`接口并将其注册到`RegisterClientHeaders`注解来添加或传播头部。

假设您的服务从上游服务的头部`x-auth`接收认证令牌，并且您的下游服务要求将此值设置为`Authorization`头部。让我们在 MicroProfile REST Client 中实现这些头部的重命名：

```java
package org.acme.quickstart;

import java.util.List;

import javax.ws.rs.core.MultivaluedHashMap;
import javax.ws.rs.core.MultivaluedMap;

import org.eclipse.microprofile.rest.client.ext.ClientHeadersFactory;

public class CustomClientHeadersFactory implements ClientHeadersFactory {

  @Override
  public MultivaluedMap<String, String> update(
      MultivaluedMap<String, String> incomingHeaders, ![1](img/1.png)
      MultivaluedMap<String, String> clientOutgoingHeaders) { ![2](img/2.png)

    final MultivaluedMap<String, String> headers =
      new MultivaluedHashMap<String, String>(incomingHeaders);
    headers.putAll(clientOutgoingHeaders); ![3](img/3.png)

    final List<String> auth = headers.get("x-auth"); ![4](img/4.png)
    headers.put("Authorization", auth);
    headers.remove("x-auth");

    return headers;
  }
}
```

![1](img/#co_quarkus_rest_clients_CO10-1)

来自入站 JAX-RS 请求的头部

![2](img/#co_quarkus_rest_clients_CO10-2)

客户端接口上指定的头部参数

![3](img/#co_quarkus_rest_clients_CO10-3)

添加所有头部

![4](img/#co_quarkus_rest_clients_CO10-4)

重命名头部值

最后，您需要使用`R⁠e⁠g⁠i⁠s⁠t⁠e⁠r⁠C⁠l⁠i⁠e⁠n⁠t​H⁠e⁠a⁠d⁠e⁠r⁠s`注解将此工厂注册到客户端：

```java
@RegisterClientHeaders(CustomClientHeadersFactory.class) ![1](img/1.png)
public interface ConfigureHeaderServices {
```

![1](img/#co_quarkus_rest_clients_CO11-1)

为此客户端注册头部工厂

## 讨论

如果您只想按原样传播头部而不做任何修改，可以通过只用`@RegisterClientHeaders`注解 REST 客户端来实现。然后将使用默认头部工厂。

此默认工厂将从入站 JAX-RS 请求中传播指定的头部到出站请求。要配置要传播的头部，您需要将它们设置为逗号分隔的值，放置在`org.eclipse.microprofile.rest.client.propagateHeaders`属性下：

```java
org.eclipse.microprofile.rest.client.propagateHeaders=Authorization,\
 MyCustomHeader
```

# 13.5 使用 REST 客户端处理多部分消息

## 问题

您希望发送多部分内容与需要其的 REST API 进行交互。

## 解决方案

使用 RESTEasy 多部分支持处理多部分消息。

有时，您需要连接的服务要求您发送多个内容主体嵌入到一个消息中，通常使用`multipart/form-data` MIME 类型。处理多部分 MIME 类型最简单的方法是使用 RESTEasy 多部分提供程序，该提供程序与 MicroProfile REST 客户端集成。

###### 重要

此功能特定于 RESTEasy/Quarkus，不属于 MicroProfile REST 客户端规范。

在开始开发之前，您需要在构建工具中添加`resteasy-multipart-provider`依赖项：

```java
<dependency>
    <groupId>org.jboss.resteasy</groupId>
    <artifactId>resteasy-multipart-provider</artifactId>
</dependency>
```

然后，您需要创建定义消息有效负载的模型对象。让我们定义一个包含两个部分的多部分消息，一个作为二进制内容，另一个作为字符串：

```java
package org.acme.quickstart;

import java.io.InputStream;

import javax.ws.rs.FormParam;
import javax.ws.rs.core.MediaType;

import org.jboss.resteasy.annotations.providers.multipart.PartType;

public class MultipartDeveloperModel {

    @FormParam("avatar") ![1](img/1.png)
    @PartType(MediaType.APPLICATION_OCTET_STREAM) ![2](img/2.png)
    public InputStream file;

    @FormParam("name")
    @PartType(MediaType.TEXT_PLAIN)
    public String developerName;

}
```

![1](img/#co_quarkus_rest_clients_CO12-1)

JAX-RS 注解来定义包含在请求内部的表单参数

![2](img/#co_quarkus_rest_clients_CO12-2)

RESTEasy 注解来定义部分内容类型

最后，您需要声明一个新方法，该方法使用`MultipartDeveloperModel`对象作为参数，并注解为`org.jboss.resteasy.annotations.providers.multipart.MultipartForm`：

```java
@POST
@Consumes(MediaType.MULTIPART_FORM_DATA) ![1](img/1.png)
@Produces(MediaType.TEXT_PLAIN)
String sendMultipartData(@MultipartForm ![2](img/2.png)
                            MultipartDeveloperModel data); ![3](img/3.png)
```

![1](img/#co_quarkus_rest_clients_CO13-1)

设置输出 MIME 类型为多部分

![2](img/#co_quarkus_rest_clients_CO13-2)

将参数定义为`multipart/form-type` MIME 类型

![3](img/#co_quarkus_rest_clients_CO13-3)

多部分数据

# 13.6 使用 REST 客户端配置 SSL

## 问题

您想要配置 REST 客户端以使用 SSL。

## 解决方案

MicroProfile REST 客户端提供了一种配置 SSL 以与其他服务通信的方式。

默认情况下，MicroProfile REST 客户端在使用 HTTPS 连接时使用 JVM 信任库来验证证书。但有时，特别是在内部服务的情况下，无法使用 JVM 信任库验证证书，因此需要提供自定义信任库。

MicroProfile REST Client 可以通过使用 `trustStore` 配置属性来设置自定义信任存储：

```java
org.acme.quickstart.FruityViceService/mp-rest/trustStore= \
    classpath:/custom-truststore.jks ![1](img/1.png)
org.acme.quickstart.FruityViceService/mp-rest/trustStorePassword=acme ![2](img/2.png)
org.acme.quickstart.FruityViceService/mp-rest/trustStoreType=JKS ![3](img/3.png)
```

![1](img/#co_quarkus_rest_clients_CO14-1)

`trustStore` 设置信任存储位置；这可以是一个类路径资源（`classpath:`）或一个文件（`file:`）

![2](img/#co_quarkus_rest_clients_CO14-2)

`trustStorePassword` 设置信任存储的密码

![3](img/#co_quarkus_rest_clients_CO14-3)

`trustStoreType` 设置信任存储的类型

Keystores 也提供了，在双向 SSL 连接中非常有用。

MicroProfile REST Client 可以通过使用 `keyStore` 配置属性来设置自定义密钥库。

```java
org.acme.quickstart.FruityViceService/mp-rest/keyStore= \
    classpath:/custom-keystore.jks ![1](img/1.png)
org.acme.quickstart.FruityViceService/mp-rest/keyStorePassword=acme ![2](img/2.png)
org.acme.quickstart.FruityViceService/mp-rest/keyStoreType=JKS ![3](img/3.png)
```

![1](img/#co_quarkus_rest_clients_CO15-1)

`keyStore` 设置密钥存储的位置；这可以是一个类路径资源（`classpath:`）或一个文件（`file:`）

![2](img/#co_quarkus_rest_clients_CO15-2)

`keyStorePassword` 设置信任存储的密码

![3](img/#co_quarkus_rest_clients_CO15-3)

`keyStoreType` 设置密钥存储的类型

最后，您可以实现 `javax.net.ssl.HostnameVerifier` 接口来覆盖当 URL 的主机名与服务器的识别主机名不匹配时的行为。然后，该接口的实现可以确定是否应允许此连接。

以下是主机名验证器的示例：

```java
package org.acme.quickstart;

import javax.net.ssl.HostnameVerifier;
import javax.net.ssl.SSLSession;

public class FruityHostnameVerifier implements HostnameVerifier {

    @Override
    public boolean verify(String hostname, SSLSession session) {
        if ("fruityvice.com".equals(hostname)) {
            return true;
        }

        return false;
    }

}
```

需要在配置文件中启用它：

```java
org.acme.quickstart.FruityViceService/mp-rest/hostnameVerifier=\
org.acme.quickstart.FruityHostnameVerifier
```

## 讨论

大多数情况下，当您在本地运行测试时，可能没有安装所有连接到外部服务所需的信任存储或密钥存储。在这些情况下，您可能会针对服务的 HTTP 版本运行测试。然而，并非总是可能，在某些第三方服务中，只启用了 HTTPS 协议。

解决此问题的一个可能方案是配置 MicroProfile REST Client 以信任任何证书。为此，您需要配置客户端并提供自定义信任管理器：

```java
package org.acme.quickstart;

import java.net.Socket;
import java.net.URI;
import java.security.KeyManagementException;
import java.security.NoSuchAlgorithmException;
import java.security.cert.CertificateException;
import java.security.cert.X509Certificate;

import javax.enterprise.context.ApplicationScoped;
import javax.net.ssl.HttpsURLConnection;
import javax.net.ssl.SSLContext;
import javax.net.ssl.SSLEngine;
import javax.net.ssl.TrustManager;
import javax.net.ssl.X509ExtendedTrustManager;

import org.apache.http.conn.ssl.NoopHostnameVerifier;
import org.eclipse.microprofile.rest.client.RestClientBuilder;

@ApplicationScoped ![1](img/1.png)
public class TrustAllFruityViceService {

  public FruityVice getFruitByName(String name) {
    FruityViceService fruityViceService = RestClientBuilder.newBuilder()
      .baseUri(URI.create("https://www.fruityvice.com/"))
      .hostnameVerifier(NoopHostnameVerifier.INSTANCE) ![2](img/2.png)
      .sslContext(trustEverything()) ![3](img/3.png)
      .build(FruityViceService.class);

    return fruityViceService.getFruitByName(name);
  }

  private static SSLContext trustEverything() { ![4](img/4.png)

    try {
      SSLContext sc = SSLContext.getInstance("SSL");
      sc.init(null, trustAllCerts(), new java.security.SecureRandom());
      HttpsURLConnection.setDefaultSSLSocketFactory(sc.getSocketFactory());
      return sc;
    } catch (KeyManagementException | NoSuchAlgorithmException e) {
      throw new IllegalStateException(e);
    }
  }

  private static TrustManager[] trustAllCerts() {
    return  new TrustManager[]{
      new X509ExtendedTrustManager(){

        @Override
        public X509Certificate[] getAcceptedIssuers() {
          return null;
        }

        @Override
        public void checkServerTrusted(X509Certificate[] chain,
                                       String authType)
          throws CertificateException {
        }

        @Override
        public void checkClientTrusted(X509Certificate[] chain,
                                       String authType)
          throws CertificateException {
        }

        @Override
        public void checkServerTrusted(X509Certificate[] chain,
                                      String authType,
                                      SSLEngine sslEngine)
          throws CertificateException {
        }

        @Override
        public void checkServerTrusted(X509Certificate[] chain,
                                       String authType,
                                       Socket socket)
          throws CertificateException {
        }

        @Override
        public void checkClientTrusted(X509Certificate[] chain,
                                       String authType,
                                       SSLEngine sslEngine)
          throws CertificateException {
        }

        @Override
        public void checkClientTrusted(X509Certificate[] chain,
                                       String authType,
                                       Socket socket)
          throws CertificateException {
        }
      }
    };
  }
}
```

![1](img/#co_quarkus_rest_clients_CO16-1)

创建一个 CDI bean；您需要使用 `@Inject` 而不是 `@RestClient` 来使用它

![2](img/#co_quarkus_rest_clients_CO16-2)

禁用主机验证

![3](img/#co_quarkus_rest_clients_CO16-3)

在不进行任何验证的情况下信任所有证书

![4](img/#co_quarkus_rest_clients_CO16-4)

使用空信任管理器自定义 `SSLContext`，有效地取消所有 SSL 检查

然后，如果您注入此实例而不是生产实例，则每个 HTTPS 请求都是有效的，不论外部服务使用的证书是什么。
