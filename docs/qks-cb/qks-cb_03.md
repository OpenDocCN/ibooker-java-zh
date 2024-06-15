# 第三章：开发 RESTful 服务

Quarkus 集成了 RESTEasy，一个 JAX-RS 实现，用于定义 REST API。在本章中，您将学习如何在 Quarkus 中开发 RESTful Web 服务。我们将涵盖以下主题：

+   如何使用 JAX-RS 创建 CRUD 服务

+   如何为请求其他域中的资源启用 CORS

+   如何实现响应式路由

+   如何实现过滤器以操纵请求和响应

# 3.1 创建一个简单的 REST API 端点

## 问题

您想要创建一个带有 CRUD 操作的 REST API 端点。

## 解决方案

使用先前生成的 JAX-RS `GreetingResource` 资源，并填充 JAX-RS 注解。

JAX-RS 是 Quarkus 中用于定义 REST 端点的默认框架。所有 JAX-RS 注解都已经正确地存在于您的类路径上。您将需要使用 HTTP 动词注解（`@GET`、`@POST`、`@PUT`、`@DELETE`）来声明端点方法将监听的 HTTP 动词。当然，您还需要 `@Path` 注解来定义相对于应用程序其余部分的 URI。

打开 `org.acme.quickstart.GreetingResource.java`：

```java
package org.acme.quickstart;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

@Path("/hello") ![1](img/1.png)
public class GreetingResource {
    @GET ![2](img/2.png)
    @Produces(MediaType.TEXT_PLAIN) ![3](img/3.png)
    public String hello() {
        return "hello"; ![4](img/4.png)
    }
}
```

![1](img/#co_developing_restful_services_CO1-1)

标识当前资源的 URI 路径

![2](img/#co_developing_restful_services_CO1-2)

响应 HTTP `GET` 请求

![3](img/#co_developing_restful_services_CO1-3)

定义返回的媒体类型（们）

![4](img/#co_developing_restful_services_CO1-4)

返回纯文本

让我们创建用于创建、更新和删除消息的其余方法：

```java
@POST ![1](img/1.png)
@Consumes(MediaType.TEXT_PLAIN) ![2](img/2.png)
public void create(String message) { ![3](img/3.png)
    System.out.println("Create");
}

@PUT ![4](img/4.png)
@Consumes(MediaType.TEXT_PLAIN)
@Produces(MediaType.TEXT_PLAIN)
public String update(String message) {
    System.out.println("Update");
    return message;
}

@DELETE ![5](img/5.png)
public void delete() {
    System.out.println("Delete");
}
```

![1](img/#co_developing_restful_services_CO2-1)

响应 HTTP `POST` 请求

![2](img/#co_developing_restful_services_CO2-2)

定义接受的媒体类型（们）

![3](img/#co_developing_restful_services_CO2-3)

请求的主体内容

![4](img/#co_developing_restful_services_CO2-4)

响应 HTTP `PUT` 请求

![5](img/#co_developing_restful_services_CO2-5)

响应 HTTP `DELETE` 请求

以下是有效的 HTTP 方法：`@GET`、`@POST`、`@PUT`、`@DELETE`、`@PATCH`、`@HEAD` 和 `@OPTIONS`。

# 3.2 提取请求参数

## 问题

您想要使用 JAX-RS 提取请求参数。

## 解决方案

使用 JAX-RS 规范提供的一些内置注解。

打开 `org.acme.quickstart.GreetingResource.java` 类，并将 `hello` 方法与请求参数更改为以下提取内容：

```java
public static enum Order {
    desc, asc;
}

@GET
@Produces(MediaType.TEXT_PLAIN)
public String hello(
                @Context UriInfo uriInfo, ![1](img/1.png)
                @QueryParam("order") Order order, ![2](img/2.png)
                @NotBlank @HeaderParam("authorization") String authorization ![3](img/3.png)
                ) {

    return String.format("URI: %s - Order %s - Authorization: %s",
                         uriInfo.getAbsolutePath(), order, authorization);
}
```

![1](img/#co_developing_restful_services_CO3-1)

获取请求的 `UriInfo`；`UriInfo` 是 JAX-RS 的一部分，允许您获取应用程序和请求 URI 信息

![2](img/#co_developing_restful_services_CO3-2)

获取名为 `order` 的查询参数作为 `Enum`

![3](img/#co_developing_restful_services_CO3-3)

获取名为 `authorization` 的头部参数，集成了 Bean 验证

通过打开新的终端窗口，启动 Quarkus 应用程序，并发送请求到`GET`方法来尝试：

```java
./mvnw clean compile quarkus:dev

curl -X GET "http://localhost:8080/hello?order=asc" \
 -H "accept: text/plain" -H "authorization: XYZ"
URI: http://localhost:8080/hello - Order asc - Authorization: XYZ

curl -X GET "http://localhost:8080/hello?order=asc" \
 -H "accept: text/plain" -v
HTTP/1.1 400 Bad Request
```

其他请求参数可以使用注解来提取，例如表单参数(`@FormParam`)，矩阵参数(`@MatrixParam`)或者 cookie 值(`@CookieParam`)。同时，使用`@Context`注解，可以注入与 JAX-RS 相关的其他元素，如`javax.ws.rs.core.SecurityContext`，`javax.ws.rs.sse.SseEventSink`或`javax.ws.rs.sse.Sse`。

## 讨论

在 Recipe 3.1 中，您看到了如何使用 JAX-RS 创建 REST API 端点，但通常需要从请求中提取更多信息而不仅仅是主体内容。

使用 Quarkus 和 JAX-RS 时需要考虑的一件重要事情是，在内部，Quarkus 默认使用 RESTEasy 直接与 Vert.x 一起工作，而不使用与`Servlet`规范相关的任何内容。

总体而言，对于开发 REST API 端点可能需要的一切都得到了很好的支持，而且在需要实现自定义`Servlet`过滤器或直接将 HTTP 请求直接编码到代码中时，Quarkus 提供了替代方案。

但是，如果有要求的话，可以通过添加`quarkus-undertow`扩展来配置 Quarkus，以在使用`Servlet`规范而不是 Vert.x 的情况下使用 RESTEasy：

```java
./mvnw quarkus:add-extension -Dextensions="quarkus-undertow"

./gradlew addExtension --extensions="quarkus-undertow"
```

## 参见

要了解更多关于 JAX-RS 的信息，请访问以下网站：

+   [Eclipse Foundation: Jakarta RESTful Web Services](https://oreil.ly/Tgn5d)

+   [RESTEasy](https://oreil.ly/WpJ3x)

# 3.3 使用语义化 HTTP 响应状态码

## 问题

您希望使用 HTTP 响应状态码正确反映请求的结果。

## 解决方案

JAX-RS 规范使用`javax.ws.rs.core.Response`接口来返回正确的 HTTP 响应状态码，以及设置任何其他必需的信息，如响应内容、cookies 或头部：

```java
package org.acme.quickstart;

import javax.ws.rs.Consumes;
import javax.ws.rs.POST;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;
import javax.ws.rs.core.UriBuilder;

@Path("/developer")
public class DeveloperResource {

    @POST
    @Produces(MediaType.APPLICATION_JSON)
    @Consumes(MediaType.APPLICATION_JSON)
    public Response createDeveloper(Developer developer) {
        developer.persist();
        return Response.created( ![1](img/1.png)
            UriBuilder
                .fromResource(DeveloperResource.class) ![2](img/2.png)
                .path(Long.toString(developer.getId())) ![3](img/3.png)
                .build()
            )
            .entity(developer) ![4](img/4.png)
            .build(); ![5](img/5.png)
    }

    public static class Developer {

        static long counter = 1;

        private long id;
        private String name;

        public long getId() {
            return id;
        }

        public void setName(String name) {
            this.name = name;
        }

        public String getName() {
            return name;
        }

        public void persist() {
            this.id = counter++;
        }
    }
}
```

![1](img/#co_developing_restful_services_CO4-1)

将响应状态码设置为 201 并将`Location`头设置为 URI，以创建的资源。

![2](img/#co_developing_restful_services_CO4-2)

从资源类设置路径

![3](img/#co_developing_restful_services_CO4-3)

在`Location`头中设置开发者 ID。

![4](img/#co_developing_restful_services_CO4-4)

将创建的开发者设置为响应内容

![5](img/#co_developing_restful_services_CO4-5)

构建`Response`对象

###### 注意

如果从你的端点返回 JSON，则在项目中需要`quarkus-resteasy-jsonb`或`quarkus-resteasy-jackson`扩展。

通过打开新的终端窗口，启动 Quarkus 应用程序，并发送请求到`GET`方法来尝试：

```java
./mvnw clean compile quarkus:dev

curl -d '{"name":"Ada"}' -H "Content-Type: application/json" \
 -X POST http://localhost:8080/developer -v

< HTTP/1.1 201 Created
< Content-Length: 21
< Content-Type: application/json
< Location: http://localhost:8080/developer/1
<
{"id":1,"name":"Ada"}
```

请注意，`Location`头包含一个有效的 URI，用于访问已创建的资源。

## 讨论

在定义 RESTful Web API 时，遵循底层技术提供的一些约定非常重要；对于 RESTful Web 服务来说，这是 HTTP 层。

定义 API 的另一个关键部分是使用正确的响应状态代码，这些代码将返回给客户端，以指示请求是否已完成。有五类状态代码：

+   信息响应（100–199）

+   成功的响应（200–299）

+   重定向（300–399）

+   客户端错误（400–499）

+   服务器错误（500–599）

默认情况下，Quarkus 尝试提供正确的 HTTP 状态代码的响应。例如，在约束违规的情况下，它提供 400 Bad Request，在服务器异常的情况下，它提供 500 Internal Server Error。但有一种情况不会默认处理：在创建资源时，应该向客户端发送一个 HTTP 201 Created 状态响应代码，该代码应在消息体中带有新资源，并将新资源的 URL 设置在`Location`头中。

## 另请参阅

完整的 HTTP 响应状态码总结在以下网站：

+   [MDN Web 文档：HTTP 响应状态代码](https://oreil.ly/Gq02d)

# 3.4 绑定 HTTP 方法

## 问题

您希望将方法绑定到 HTTP 动词，但 JAX-RS 规范没有提供专用的注解。

## 解决方案

使用`javax.ws.rs.HttpMethod`注解创建您的 HTTP 方法注解。

JAX-RS 规范提供了七个注解来指定方法应该响应的 HTTP 方法。这些注解是`@GET`、`@POST`、`@PUT`、`@DELETE`、`@PATCH`、`@HEAD`和`@OPTIONS`。但还有许多其他 HTTP 方法，JAX-RS 提供了`javax.ws.rs.HttpMethod`注解来支持这些其他方法。

首先要做的是创建一个元注解。我们将使用在[RFC-4918](https://tools.ietf.org/html/rfc4918#section-9.10)中定义的`LOCK`动词，该动词锁定访问或刷新资源的现有锁。我们的注解将命名为`LOCK`，并用`@javax.ws.rs.HttpMethod`注解：

```java
package org.acme.quickstart;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import javax.ws.rs.HttpMethod;

@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@HttpMethod("LOCK") ![1](img/1.png)
@Documented
public @interface LOCK {
}
```

![1](img/#co_developing_restful_services_CO5-1)

将`LOCK` HTTP 方法绑定到注解

最后，将此注解用于资源方法，以将其绑定到`LOCK` HTTP 动词。

打开`org.acme.quickstart.GreetingResource.java`类并创建一个`LOCK`方法：

```java
@LOCK ![1](img/1.png)
@Produces(MediaType.TEXT_PLAIN)
@Path("{id}")
public String lockResource(@PathParam("id") long id) {
    return id + " locked";
}
```

![1](img/#co_developing_restful_services_CO6-1)

绑定到`LOCK` HTTP 方法

通过打开新的终端窗口，启动 Quarkus 应用程序，并向`LOCK`方法发送请求来尝试它：

```java
./mvnw clean compile quarkus:dev

curl -X LOCK http://localhost:8080/hello/1
1 locked
```

## 另请参阅

完整的 HTTP 方法列表可以在以下 GitHub 页面找到：

+   [深入了解你的 HTTP 方法](https://oreil.ly/DC9Wi)

# 3.5 启用跨域资源共享（CORS）

## 问题

您希望从另一个域请求受限资源。

## 解决方案

使用`quarkus.http.cors`配置属性启用 CORS。

## 讨论

*跨域资源共享*（CORS）是一种机制，允许从提供第一个资源的域之外的另一个域请求受限资源。Quarkus 提供了一组配置属性来配置 CORS。

要在 Quarkus 中启用 CORS，您需要在 *application.properties* 文件中将 `quarkus.http.cors` 配置属性设置为 `true`。

CORS 配置示例可能如下所示：

```java
quarkus.http.cors=true
quarkus.http.cors.origins=http://example.com
quarkus.http.cors.methods=GET,PUT,POST,DELETE
quarkus.http.cors.headers=accept,authorization,content-type,x-requested-with
```

您可以使用 `curl` 查看输出和头部：

```java
curl -d '{"name":"Ada"}' -H "Content-Type: application/json" \
 -X POST http://localhost:8080/developer \
 -H "Origin: http://example.com" --verbose
```

输出应显示 `access-control-allow-origin` 头部：

```java
upload completely sent off: 14 out of 14 bytes
* Mark bundle as not supporting multiuse
< HTTP/1.1 201 Created
< access-control-allow-origin: http://example.com
< access-control-allow-credentials: true
< Content-Length: 21
< Content-Type: application/json
< Location: http://localhost:8080/developer/5
```

## 参见

您可以在以下 Wikipedia 页面上找到有关 CORS 的更多信息：

+   [跨源资源共享](https://oreil.ly/iSiqh)

# 3.6 使用响应式路由

## 问题

您希望使用响应式路由实现 HTTP 端点。

## 解决方案

使用 Vert.x `io.vertx.ext.web.Router` 路由器实例或 `i⁠o⁠.⁠q⁠u⁠a⁠r⁠k⁠u⁠s​.⁠v⁠e⁠r⁠t⁠x⁠.⁠w⁠e⁠b⁠.⁠R⁠o⁠u⁠t⁠e` 注解。

在 Quarkus 中有两种使用响应式路由的方式。第一种方式是直接注册路由，使用 `io.vertx.ext.web.Router` 类。

要在启动时检索 `Router` 实例，您需要使用上下文和依赖注入（CDI）观察对象的创建。

创建一个名为 `org.acme.quickstart.ApplicationRoutes.java` 的新类：

```java
package org.acme.quickstart;

import javax.enterprise.context.ApplicationScoped;
import javax.enterprise.event.Observes;

import io.quarkus.vertx.http.runtime.filters.Filters;
import io.quarkus.vertx.web.Route;
import io.vertx.core.http.HttpMethod;   ![5](img/5.png)
import io.vertx.ext.web.Router;         ![5](img/5.png)
import io.vertx.ext.web.RoutingContext; ![5](img/5.png)

@ApplicationScoped ![1](img/1.png)
public class ApplicationRoutes {

    public void routes(@Observes Router router) { ![2](img/2.png)

        router
            .get("/ok") ![3](img/3.png)
            .handler(rc -> rc.response().end("OK from Route")); ![4](img/4.png)

    }
}
```

![1](img/#co_developing_restful_services_CO7-4)

将对象实例化到 CDI 容器中，作用域为 *application*

![2](img/#co_developing_restful_services_CO7-5)

提供 `Router` 对象来注册路由

![3](img/#co_developing_restful_services_CO7-6)

将 `GET` HTTP 方法绑定到 `/ok`

![4](img/#co_developing_restful_services_CO7-7)

处理逻辑

![5](img/#co_developing_restful_services_CO7-1)

稍后在示例中使用的导入

通过打开新的终端窗口，启动 Quarkus 应用程序，并向新方法发送请求来尝试它：

```java
./mvnw clean compile quarkus:dev

curl http://localhost:8080/ok
OK from Route
```

使用第二种方式使用响应式路由是一种声明性方法，使用 `i⁠o⁠.⁠q⁠u⁠a⁠r⁠k⁠u⁠s​.⁠v⁠e⁠r⁠t⁠x⁠.⁠w⁠e⁠b⁠.⁠R⁠o⁠u⁠t⁠e` 注解。要访问此注解，您需要添加 `quarkus-vertx-web` 扩展：

```java
./mvnw quarkus:add-extension -Dextensions="quarkus-vertx-web"
```

然后您可以使用 `@Route` 注释方法。这些方法必须在 CDI bean 中定义。

打开 `org.acme.quickstart.ApplicationRoutes.java` 类并定义一个路由：

```java
@Route(path = "/declarativeok", methods = HttpMethod.GET) ![1](img/1.png)
public void greetings(RoutingContext routingContext) { ![2](img/2.png)
    String name = routingContext.request().getParam("name"); ![3](img/3.png)

    if (name == null) {
        name = "world";
    }

    routingContext.response().end("OK " + name + " you are right"); ![4](img/4.png)
}
```

![1](img/#co_developing_restful_services_CO8-1)

设置 HTTP 路径和方法

![2](img/#co_developing_restful_services_CO8-2)

使用 `RoutingContext` 获取请求信息

![3](img/#co_developing_restful_services_CO8-3)

获取查询参数

![4](img/#co_developing_restful_services_CO8-4)

处理逻辑

通过打开新的终端窗口，启动 Quarkus 应用程序，并向新方法发送请求来尝试它：

```java
./mvnw clean compile quarkus:dev

curl localhost:8080/declarativeok?name=Alex
OK Alex you are right
```

## 讨论

Quarkus HTTP 基于非阻塞和响应式引擎。在幕后它使用 Vert.x 和 Netty。当接收到请求时，它由 *事件循环* 管理，可以依靠工作线程（对于 servlet 或 JAX-RS），或使用 I/O 线程（对于响应式路由），来处理调用的逻辑。

需要注意的是，响应式路由必须是非阻塞的，或者显式声明为阻塞；否则，由于响应式事件循环的特性，会阻塞循环，从而导致无法处理进一步的循环，直到线程解除阻塞。

在同一个项目中，您可以毫无问题地混合使用 JAX-RS 端点和响应式路由。

## 参见

您可以在以下网页了解更多有关 Vert.x 中响应式路由的信息：

+   [基础的 Vert.x-Web 概念](https://oreil.ly/kznp9)

# 3.7 拦截 HTTP 请求

## 问题

您希望拦截 HTTP 请求以操纵请求或响应。

## 解决方案

有时候您需要在到达终端逻辑之前操纵请求（例如安全检查），或者在返回给调用者的响应发送之前操纵响应（例如压缩响应）。使用 Quarkus，您可以通过 Vert.x 的`Filters`或者 JAX-RS 的过滤器接口来拦截 HTTP 请求。

让我们看看如何使用`io.quarkus.vertx.http.runtime.filters.Filters`来实现过滤器。

要在启动时获取`Filters`实例，您需要观察使用 CDI 创建对象的过程。

打开`org.acme.quickstart.ApplicationRoutes.java`类，并添加一个名为`filters`的方法：

```java
public void filters(@Observes Filters filters) { ![1](img/1.png)
    filters
        .register(
            rc -> {
                rc.response() ![2](img/2.png)
                    .putHeader("V-Header", "Header added by VertX Filter"); ![3](img/3.png)
                rc.next(); ![4](img/4.png)
            },
            10); ![5](img/5.png)
}
```

![1](img/#co_developing_restful_services_CO9-1)

提供`Filters`对象来注册过滤器

![2](img/#co_developing_restful_services_CO9-2)

修改响应

![3](img/#co_developing_restful_services_CO9-3)

向响应添加新的头部

![4](img/#co_developing_restful_services_CO9-4)

继续过滤器链

![5](img/#co_developing_restful_services_CO9-5)

设置执行顺序

需要注意的是，这些过滤器适用于 Servlet、JAX-RS 资源和响应式路由。

通过打开新的终端窗口，启动 Quarkus 应用，并向新方法发送请求来尝试它：

```java
./mvnw clean compile quarkus:dev

echo Reactive-Route
curl localhost:8080/ok -v
< V-Header: Header added by VertX Filter
< content-length: 13
OK from Route

echo JAX-RS
curl -X GET "http://localhost:8080/hello?order=asc" \
 -H "accept: text/plain" -H "authorization: XYZ" -v
< V-Header: Header added by VertX Filter
< content-length: 65
URI: http://localhost:8080/hello - Order asc - Authorization: XYZ
```

注意，注册的过滤器修改了两个请求（响应式路由和 JAX-RS 端点），并添加了一个新的头部。

也可以使用`javax.ws.rs.container.ContainerRequestFilter`/`javax.ws.rs.container.ContainerResponseFilter`接口来实现过滤器。

创建一个名为`org.acme.quickstart.HeaderAdditionContainerResponseFilter.java`的新类：

```java
package org.acme.quickstart;

import java.io.IOException;

import javax.ws.rs.container.ContainerRequestContext;
import javax.ws.rs.container.ContainerResponseContext;
import javax.ws.rs.container.ContainerResponseFilter;
import javax.ws.rs.ext.Provider;

@Provider ![1](img/1.png)
public class HeaderAdditionContainerResponseFilter
                implements ContainerResponseFilter { ![2](img/2.png)

    @Override
    public void filter(ContainerRequestContext requestContext,
                       ContainerResponseContext responseContext)
            throws IOException {
                responseContext.getHeaders()
                  .add("X-Header", "Header added by JAXRS Filter"); ![3](img/3.png)
    }
}
```

![1](img/#co_developing_restful_services_CO10-1)

将此类设置为扩展接口

![2](img/#co_developing_restful_services_CO10-2)

应用响应中的更改

![3](img/#co_developing_restful_services_CO10-3)

向响应添加新的头部

此过滤器仅适用于 JAX-RS 资源，不适用于响应式路由。

通过打开新的终端窗口，启动 Quarkus 应用，并向新方法发送请求来尝试它：

```java
./mvnw clean compile quarkus:dev

echo Reactive-Route
curl localhost:8080/ok -v
< V-Header: Header added by VertX Filter
< content-length: 13
OK from Route

echo JAX-RS
curl -X GET "http://localhost:8080/hello?order=asc" \
 -H "accept: text/plain" -H "authorization: XYZ" -v
< V-Header: Header added by VertX Filter
< Content-Length: 65
< Content-Type: text/plain;charset=UTF-8
< X-Header: Header added by JAXRS Filter
URI: http://localhost:8080/hello - Order asc - Authorization: XYZ
```

## 讨论

注意，在反应式路由端点的情况下，只添加了`V-Header`头部，而没有添加`X-Header`头部。与此同时，在 JAX-RS 端点中，请求由两个过滤器修改，同时添加了这两个 HTTP 头部。

## 参见

要了解更多关于 JAX-RS 和 Vert.x 的信息，您可以访问以下网站：

+   [Eclipse Foundation: Jakarta RESTful web services](https://oreil.ly/xioAv)

+   [Vert.x 文档](https://vertx.io/docs)

# 3.8 使用 SSL 进行安全连接

## 问题

您希望通过安全连接来防止攻击者窃取敏感信息。

## 解决方案

启用 Quarkus 以使用 SSL 来保护连接。

在传输敏感信息（密码、账号、健康信息等）时，保护客户端和应用程序之间的通信非常重要。因此，使用 SSL 保护服务之间的通信至关重要。

要保护通信，必须提供两个元素：证书和关联的密钥文件。这两者可以分别提供，也可以作为一个*密钥库*的形式提供。

让我们配置 Quarkus 使用包含证书条目的密钥库：

```java
quarkus.http.ssl-port=8443 ![1](img/1.png)
quarkus.http.ssl.certificate.key-store-file=keystore.jks ![2](img/2.png)
quarkus.http.ssl.certificate.key-store-file-type=jks
quarkus.http.ssl.certificate.key-store-password=changeit ![3](img/3.png)
```

![1](img/#co_developing_restful_services_CO11-1)

设置 HTTPS 端口

![2](img/#co_developing_restful_services_CO11-2)

密钥库的类型和相对于*src/main/resources*的位置

![3](img/#co_developing_restful_services_CO11-3)

打开密钥库的密码

启动应用程序并发送请求到 HTTPS 端点：

```java
./mvnw clean compile quarkus:dev

curl --insecure https://localhost:8443/hello
hello
```

由于证书是自签名的，提供了`--insecure`标志以跳过证书验证。在证书不是自签名的情况下，不应提供`insecure`标志。在本例中为简单起见使用了该标志。

###### 重要提示

在配置文件中以明文提供密码是一种不好的做法。可以通过环境变量`QUARKUS_HTTP_SSL_CERTIFICATE_KEY_STORE_PASSWORD`来提供，这是在引入 MicroProfile Config 规范时在本书开头介绍的。

## 讨论

对于忙碌的开发者，这是如何为 Quarkus 生成您自己的密钥证书的方法：

1.  转到*src/main/resources*。

1.  执行以下命令：

    ```java
    keytool -genkey -keyalg RSA -alias selfsigned \
     -keystore keystore.jks -storepass changeit \
     -validity 360 -keysize 2048
    ```

## 参见

要了解如何生成证书、密钥库和信任库，请参阅以下网页：

+   [Oracle: Java 平台标准版工具参考:`keytool`](https://oreil.ly/mwOSH)
