# 8 网络堆栈

本章涵盖

+   构建边缘服务和公共 API

+   Vert.x 网络客户端

+   JSON Web 令牌（JWT）和跨源资源共享（CORS）

+   使用 Vert.x 将 Vue.js 响应式应用程序提供和集成

+   使用 REST Assured 测试 HTTP API

响应式应用程序通常使用 HTTP，因为它是一个非常通用的协议，而 Vert.x 提供了对 Web 技术的全面支持。Vert.x 网络堆栈提供了许多用于构建 Web 应用程序后端的工具。这包括高级路由、身份验证、HTTP 客户端等。本章将指导你如何公开使用*JSON Web 令牌*（JWT）进行访问控制的 HTTP API，向其他服务发出 HTTP 请求，并构建一个连接到 HTTP API 的响应式单页应用程序。

注意：本书不涵盖以下从 Vert.x 网络堆栈中值得注意的元素，这些元素对于构建本书的这一部分的应用程序不是必需的：正则表达式路由、cookies、服务器端会话、服务器端模板渲染和跨站请求伪造保护。你可以在官方文档中了解更多关于这些主题的详细信息：[`vertx.io/`](https://vertx.io/)。

## 8.1 公开公共 API

让我们首先回顾一下公共 API 服务的作用，如图 8.1 所示。这个服务是一个边缘服务（或服务网关，取决于你更喜欢如何命名），因为它公开了一个 HTTP API，但它本质上组合了其他服务中找到的功能。在这种情况下，使用了用户资料和活动服务。这两个服务是应用程序内部的，并且没有公开暴露。它们也没有任何形式的身份验证和访问控制，这是公共 API 在大多数操作中无法承担的。

![](img/CH08_F01_Ponge.png)

图 8.1 公共 API 概述

以下 Vert.x 模块需要实现公共 API：

+   `vertx-web`，提供高级 HTTP 请求处理功能

+   `vertx-web-client`，用于向用户资料和活动服务发出 HTTP 请求

+   `vertx-auth-jwt`，用于生成和处理 JSON Web 令牌（JWT）以及执行访问控制

公共 API 服务的完整源代码可以在本书源代码仓库的 part2-steps-challenge/public-api 文件夹中找到。

我们将从 Vert.x 网络路由开始。

### 8.1.1 路由 HTTP 请求

Vert.x 核心提供了一个非常底层的 HTTP 服务器 API，其中你需要为所有类型的 HTTP 请求传递一个请求处理器。如果你只使用 Vert.x 核心，你需要手动检查请求的路径和方法。这在简单情况下是可以的，这也是我们在一些早期章节中做的事情，但它很快就会变得复杂。

`vertx-web`模块提供了一个可以作为 Vert.x HTTP 服务器请求处理器的`router`，并根据请求路径（例如，`/foo`）和 HTTP 方法（例如，`POST`）将 HTTP 请求调度到合适的处理器。这如图 8.2 所示。

![图片](img/CH08_F02_Ponge.png)

图 8.2 路由 HTTP 请求

下面的列表显示了如何初始化并设置路由作为 HTTP 请求处理程序。

列表 8.1 初始化和使用路由作为 HTTP 请求处理程序

```
Router router = Router.router(vertx);
// (...)                               ❶

vertx.createHttpServer()
  .requestHandler(router)              ❷
  .listen(8080);
```

❶ 定义路由

❷ 路由器只是另一个 HTTP 请求处理程序。

`Router` 类提供了一个流畅的 API 来描述基于 HTTP 方法和路径的 *路由*，如下面的列表所示。

列表 8.2 定义路由

```
BodyHandler bodyHandler = BodyHandler.create();               ❶
router.post().handler(bodyHandler);                           ❷
router.put().handler(bodyHandler);

String prefix = "/api/v1";

router.post(prefix + "/register").handler(this::register);    ❸
router.post(prefix + "/token").handler(this::token);
// (...) defines jwtHandler, more later

router.get(prefix + "/:username/:year/:month")                ❹
  .handler(jwtHandler)                                        ❺
  .handler(this::checkUser)
  .handler(this::monthlySteps);
// (...)
```

❶ BodyHandler 是一个预定义的处理程序，用于提取 HTTP 请求体有效载荷。

❷ 在这里，bodyHandler 被用于所有 HTTP POST 和 PUT 请求。

❸ 注册方法处理 /api/v1/register POST 请求。

❹ 我们可以通过在元素前加上“:”来提取路径参数。

❺ 处理程序可以链式调用。

Vert.x 路由器的一个有趣特性是处理程序可以链式调用。根据列表 8.2 中的定义，一个发往 `/api/v1/register` 的 `POST` 请求首先通过一个 `BodyHandler` 实例。这个处理程序对于轻松解码 HTTP 请求体有效载荷很有用。下一个处理程序是 `register` 方法。

列表 8.2 还定义了发往 `monthlySteps` 的 `GET` 请求的路由，其中请求首先通过 `jwtHandler`，然后是 `checkUser`，如图 8.3 所示。这对于分解 HTTP 请求，分步骤处理关注点很有用：`jwtHandler` 检查请求中是否存在有效的 JWT 令牌，`checkUser` 检查 JWT 令牌是否授予访问资源的权限，而 `monthlySteps` 检查用户一个月内走了多少步。

![图片](img/CH08_F03_Ponge.png)

图 8.3 每月步骤端点的路由链

注意，`checkUser` 和 `jwtHandler` 都将在第 8.2 节中讨论。

提示：`io.vertx.ext.web.handler` 包包含有用的实用处理程序，包括 `BodyHandler`。它特别提供了 HTTP 认证、CORS、CSRF、favicon、HTTP 会话、静态文件服务、虚拟主机和模板渲染的处理程序。

### 8.1.2 制作 HTTP 请求

现在我们来深入了解处理程序的实现。由于公共 API 服务将请求转发到用户配置文件和活动服务，我们需要使用 Vert.x 网络客户端来执行 HTTP 请求。如前所述，Vert.x 核心 API 提供了低级 HTTP 客户端，而 `vertx-web-client` 模块中的 `WebClient` 类提供了一个更丰富的 API。

创建一个网络客户端实例就像这样：

```
WebClient webClient = WebClient.create(vertx);
```

`WebClient` 实例通常存储在 verticle 类的私有字段中，因为它可以用来执行多个并发 HTTP 请求。整个应用程序使用 RxJava 2 绑定，因此我们可以利用它们来组合异步操作。正如你将在后面的示例中看到的，RxJava 绑定有时会带来处理错误管理的附加功能。

下面的列表显示了 `register` 路由处理程序的实现。

列表 8.3 在路由处理程序中使用 Vert.x 网络客户端

```
private void register(RoutingContext ctx) {
  webClient
    .post(3000, "localhost", "/register")                       ❶
    .putHeader("Content-Type", "application/json")              ❷
    .rxSendJson(ctx.getBodyAsJson())                            ❸
    .subscribe(
      response -> sendStatusCode(ctx, response.statusCode()),   ❹
      err -> sendBadGateway(ctx, err));
}

private void sendStatusCode(RoutingContext ctx, int code) {
  ctx.response().setStatusCode(code).end();
}

private void sendBadGateway(RoutingContext ctx, Throwable err) {
  logger.error("Woops", err);
  ctx.fail(502);
}
```

❶ 方法与 HTTP 方法（GET、POST 等）匹配。

❷ 可以传递 HTTP 头。

❸ 这将请求从 Vert.x Buffer 转换为 JsonObject。

❹ RxJava 的订阅触发请求。

这个例子演示了如何使用路由器处理 HTTP 请求，以及如何使用 Web 客户端。`RoutingContext`类封装了关于 HTTP 请求的详细信息，并通过`response`方法提供 HTTP 响应对象。HTTP 头可以在请求和响应中设置，一旦调用`end`方法，响应就会被发送。可以指定状态码，尽管默认情况下它将是`200`（OK）。

你可以看到`getBodyAsJson`将 HTTP 请求体转换为`JsonObject`，而`rxSendJson`则发送一个带有`JsonObject`作为体的 HTTP 请求。默认情况下，Vert.x `Buffer`对象在请求和响应中都携带体，但有一些辅助方法可以将它们转换为`String`、`JsonObject`和`JsonArray`。

下一个列表提供了一个用于 HTTP `GET`请求到`/api/v1/:username`的路由处理器方法，其中`:username`是一个路径参数。

列表 8.4 获取并转发用户详情

```
private void fetchUser(RoutingContext ctx) {
  webClient
    .get(3000, "localhost", "/" + ctx.pathParam("username"))      ❶
    .as(BodyCodec.jsonObject())                                   ❷
    .rxSend()
    .subscribe(
      resp -> forwardJsonOrStatusCode(ctx, resp),
      err -> sendBadGateway(ctx, err));
}

private void forwardJsonOrStatusCode(RoutingContext ctx, 
➥ HttpResponse<JsonObject> resp) {
  if (resp.statusCode() != 200) {
    sendStatusCode(ctx, resp.statusCode());
  } else {
    ctx.response()
      .putHeader("Content-Type", "application/json")
      .end(resp.body().encode());                                 ❸
  }
}
```

❶ 提取路径参数

❷ 将响应转换为 JsonObject

❸ 使用一些内容结束响应

这个例子展示了`as`方法，它使用`BodyCodec`将 HTTP 响应转换为除`Buffer`之外的其他类型。你还可以看到 HTTP 响应的`end`方法可以接受一个参数，即响应内容。它可以是`String`或`Buffer`。虽然通常情况下响应是在单个`end`方法调用中发送的，但你可以在最终`end`调用关闭 HTTP 响应之前，使用`write`方法发送中间片段，如下所示：

```
response.write("hello").write(" world").end();
```

## 8.2 使用 JWT 令牌进行访问控制

JSON Web Token (JWT)是一个用于在各方之间安全传输 JSON 编码数据的开放规范([`tools.ietf.org/html/rfc7519`](https://tools.ietf.org/html/rfc7519))。JWT 令牌使用对称共享密钥或非对称公钥/私钥对进行签名，因此始终可以验证它们包含的信息没有被修改。这非常有趣，因为 JWT 令牌可以用来持有诸如身份和授权许可之类的声明。JWT 令牌可以作为 HTTP 请求的一部分通过`Authorization` HTTP 头进行交换。

让我们看看如何使用 JWT 令牌，它们包含哪些数据，以及如何使用 Vert.x 验证和颁发它们。

提示 JWT 是 Vert.x 支持的一种协议。Vert.x 提供了`vertx-auth-oauth2`模块用于 OAuth2，这是 Google、GitHub 和 Twitter 等公共服务提供商中流行的协议。如果你需要将应用程序与这些服务集成（例如，在访问用户的 Gmail 账户数据时），或者当你的应用程序想要通过 OAuth2 授予第三方访问权限时，你将对此感兴趣。

### 8.2.1 使用 JWT 令牌

为了说明使用 JWT 令牌，让我们与公共 API 交互，并使用密码`123`作为用户`foo`进行认证，以获取 JWT 令牌。以下列表显示了 HTTP 响应。

列表 8.5 获取 JWT 令牌

```
$ http :4000/api/v1/token username=foo password=123      ❶
HTTP/1.1 200 OK
Content-Type: application/jwt
content-length: 496

eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJkZXZpY2VJZCI6ImExYjIiLCJpYXQiOjE1NjUx
➥ Njc0NzUsImV4cCI6MTU2NTc3MjI3NSwiaXNzIjoiMTBrLXN0ZXBzLWFwaSIsInN1YiI6ImZvb
➥ yJ9.J_tn2BjMNYE6eFSHwSJ9e8DoCEUr_xMSlYAyBSy1-E_pouvDq4lp8QjG51cJoa5Gbrt1bg
➥ tDHinJsLncG1RIsGr_cz1rQw8_GlI_-GdhqFBw8dVjlsgykSf5tfaiiRwORmz7VH_AAk-935aV
➥ lxMg4mxkbOvN4YDxRLhLb4Y78TA47F__ivNsM4gLD8CHzOUmTEta_pjpZGzsErmYvzDOV6F7rO
➥ ZcRhZThJxLvR3zskrtx83iaNHTwph53bkHNOQzC66wxNMar_T4HMRWzqnrr-sFIcOwLFsWJKow
➥ c1rQuadjv-ew541YQLaVmkEcai6leZLwCfCTcsxMX9rt0AmOFg
```

❶ 使用密码 123 作为用户 foo 进行认证

JWT 令牌的 MIME 类型为`application/jwt`，它是纯文本。我们可以传递令牌来发送请求，如下所示。

列表 8.6 使用 JWT 令牌访问资源

```
http :4000/api/v1/foo Authorization:'Bearer 
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJkZXZpY2VJZCI6ImExYjIiLCJpYXQiOjE1NjUx➥ Njc0NzUsImV4cCI6MTU2NTc3MjI3NSwiaXNzIjoiMTBrLXN0ZXBzLWFwaSIsInN1YiI6ImZvb
➥ yJ9.J_tn2BjMNYE6eFSHwSJ9e8DoCEUr_xMSlYAyBSy1-E_pouvDq4lp8QjG51cJoa5Gbrt1bg
➥ tDHinJsLncG1RIsGr_cz1rQw8_GlI_-GdhqFBw8dVjlsgykSf5tfaiiRwORmz7VH_AAk-935a
➥ VlxMg4mxkbOvN4YDxRLhLb4Y78TA47F__ivNsM4gLD8CHzOUmTEta_pjpZGzsErmYvzDOV6F7
➥ rOZcRhZThJxLvR3zskrtx83iaNHTwph53bkHNOQzC66wxNMar_T4HMRWzqnrr-sFIcOwLFsWJK
➥ owc1rQuadjv-ew541YQLaVmkEcai6leZLwCfCTcsxMX9rt0AmOFg'
HTTP/1.1 200 OK                                            ❶
Content-Type: application/json
content-length: 90

{
    "city": "Lyon",
    "deviceId": "a1b2",
    "email": "foo@bar.com",
    "makePublic": true,
    "username": "foo"
}
```

❶ 我们可以访问资源，因为我们有一个针对用户 foo 的有效令牌。

提示：令牌值适合单行，并且`Bearer`和令牌之间只有一个空格。

令牌通过`Authorization` HTTP 头部传递，并且其值以`Bearer`为前缀。在这里，令牌允许我们访问资源`/api/v1/foo`，因为令牌是为用户`foo`生成的。如果我们尝试不使用令牌，或者尝试访问其他用户的资源，如下列所示，我们将被拒绝访问。

列表 8.7 无匹配 JWT 令牌访问资源

```
http :4000/api/v1/abc Authorization:'Bearer 
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJkZXZpY2VJZCI6ImExYjIiLCJpYXQiOjE1NjUx➥ Njc0NzUsImV4cCI6MTU2NTc3MjI3NSwiaXNzIjoiMTBrLXN0ZXBzLWFwaSIsInN1YiI6ImZvb
➥ yJ9.J_tn2BjMNYE6eFSHwSJ9e8DoCEUr_xMSlYAyBSy1-E_pouvDq4lp8QjG51cJoa5Gbrt1b
➥ gtDHinJsLncG1RIsGr_cz1rQw8_GlI_-GdhqFBw8dVjlsgykSf5tfaiiRwORmz7VH_AAk-935
➥ aVlxMg4mxkbOvN4YDxRLhLb4Y78TA47F__ivNsM4gLD8CHzOUmTEta_pjpZGzsErmYvzDOV6F
➥ 7rOZcRhZThJxLvR3zskrtx83iaNHTwph53bkHNOQzC66wxNMar_T4HMRWzqnrr-sFIcOwLFsW
➥ JKowc1rQuadjv-ew541YQLaVmkEcai6leZLwCfCTcsxMX9rt0AmOFg'
HTTP/1.1 403 Forbidden                                        ❶
content-length: 0
```

❶ 我们因为传递了一个针对用户 foo 的（有效）令牌而被拒绝访问用户 abc 的资源。

### 8.2.2 JWT 令牌中包含什么？

到目前为止一切顺利，但令牌字符串中包含什么呢？

如果你仔细看，你会看到 JWT 令牌字符串是一行大文本，分为三部分，每部分之间由点分隔。这三部分的形式为`header.payload.signature`：

+   `header`是一个 JSON 文档，指定了令牌的类型和使用的签名算法。

+   `payload`是一个包含*声明*的 JSON 文档，这些声明是 JSON 条目，其中一些是规范的一部分，而另一些可以是自由形式的。

+   `signature`是头部和`payload`的签名，使用共享密钥或私钥，具体取决于你选择的算法。

头部和`payload`使用*Base64*算法进行编码。如果你解码列表 8.5 中获得的 JWT 令牌，头部包含以下内容：

```
{
  "typ": "JWT",
  "alg": "RS256"
}
```

这就是`payload`包含的内容：

```
{
  "deviceId": "a1b2",
  "iat": 1565167475,
  "exp": 1565772275,
  "iss": "10k-steps-api",
  "sub": "foo"
}
```

这里，`deviceId`是用户`foo`的设备标识符，`sub`是*主题*（用户`foo`），`iat`是令牌签发日期，`exp`是令牌过期日期，而`iss`是令牌发行者（我们的服务）。

签名允许你检查头部和`payload`的内容是否由发行者签名且未被修改，只要你知道公钥。这使得 JWT 令牌成为 API 中授权和访问控制的绝佳选择；包含所有所需声明的令牌是自包含的，无需对每个请求与身份管理服务（如 LDAP/OAuth 服务器）进行检查。

重要的是要理解，任何拥有 JWT 令牌的人都可以解码其内容，因为*Base64*不是一个加密算法。你绝不应该在令牌中放入敏感数据，如密码，即使它们通过像 HTTPS 这样的安全通道传输。同样重要的是设置令牌过期日期，以便受损的令牌不能无限期地使用。处理 JWT 令牌过期的策略有很多，比如在后台维护受损令牌列表，以及将短期过期期限与客户端频繁的有效性扩展请求相结合，其中发行者重新发送令牌，但带有扩展的`exp`声明。

### 8.2.3 使用 Vert.x 处理 JWT 令牌

为了颁发和检查令牌，我们首先需要一对公钥和私钥，这样我们就可以签署 JWT 令牌。您可以使用以下列表中的 shell 脚本生成这些密钥。

列表 8.8 生成 RSA 2048 位公钥和私钥

```
#!/bin/bash
openssl genrsa -out private.pem 2048
openssl pkcs8 -topk8 -inform PEM -in private.pem -out 
➥ private_key.pem -nocrypt
openssl rsa -in private.pem -outform PEM -pubout -out public_key.pem
```

下一个列表展示了用于将 PEM 文件作为字符串读取的辅助类。

列表 8.9 读取 RSA 密钥的辅助类

```
class CryptoHelper {

  static String publicKey() throws IOException {
    return read("public_key.pem");
  }

  static String privateKey() throws IOException {
    return read("private_key.pem");
  }

  private static String read(String file) throws IOException {
    Path path = Paths.get("public-api", file);
    if (!path.toFile().exists()) {                                             ❶
      path = Paths.get("..", "public-api", file);
    }
    return String.join("\n", Files.readAllLines(path, StandardCharsets.UTF_8));❷
  }
}
```

❶ 这允许我们从服务文件夹或应用程序项目根目录运行服务。

❷ 将所有行连接起来，用换行符分隔

注意到`CryptoHelper`中的代码使用了阻塞 API。由于此代码仅在初始化时运行一次，且 PEM 文件较小，我们可以承受对事件循环的潜在但可忽略的阻塞。

然后，我们可以创建一个 Vert.x JWT 处理器，如下所示。

列表 8.10 创建 JWT 处理器

```
String publicKey = CryptoHelper.publicKey();
String privateKey = CryptoHelper.privateKey();

jwtAuth = JWTAuth.create(vertx, new JWTAuthOptions()          ❶
  .addPubSecKey(new PubSecKeyOptions()
    .setAlgorithm("RS256")
    .setBuffer(publicKey))
  .addPubSecKey(new PubSecKeyOptions()
    .setAlgorithm("RS256")
    .setBuffer(privateKey)));

JWTAuthHandler jwtHandler = JWTAuthHandler.create(jwtAuth);   ❷
```

❶ jwtAuth 是 JWTAuth 类型的一个私有字段。

❷ Vert.x 路由处理器用于 JWT 认证

JWT 处理器可用于需要 JWT 认证的路由，因为它解码`Authorization`头以提取 JWT 数据。

下一个列表回顾了一个具有处理器链中的处理器的路由。

列表 8.11 路由中的 JWT 处理器

```
router.get(prefix + "/:username/:year/:month")
  .handler(jwtHandler)                           ❶
  .handler(this::checkUser)
  .handler(this::monthlySteps);
```

❶ JWT 处理器

JWT 处理器支持来自`vertx-auth-common`模块的常用认证 API，它为不同类型的认证机制（如数据库、OAuth 或 Apache `.htdigest`文件）提供了一个统一的视图。处理器将认证数据放入路由上下文中。

以下列表展示了`checkUser`方法的实现，其中我们检查 JWT 令牌中的用户是否与 HTTP 请求路径中的用户相同。

列表 8.12 检查是否存在有效的 JWT 令牌

```
private void checkUser(RoutingContext ctx) {
  String subject = ctx.user().principal().getString("sub");    ❶
  if (!ctx.pathParam("username").equals(subject)) {            ❷
    sendStatusCode(ctx, 403);
  } else {
    ctx.next();                                                ❸
  }
}
```

❶ 从 JWT 令牌中获取用户名

❷ 在 HTTP 请求路径中指定的用户名

❸ 传递给下一个处理器

这提供了一个简单的关注点分离，因为`checkUser`处理器专注于访问控制，并在授予访问权限时通过调用`next`将控制权委托给链中的下一个处理器，或者在错误的用户尝试访问资源时，以 403 状态码结束请求。

确认访问控制正确后，以下列表中的`monthlySteps`方法可以专注于向活动服务发送请求。

列表 8.13 获取月度步数数据

```
private void monthlySteps(RoutingContext ctx) {
  String deviceId = ctx.user().principal().getString("deviceId");     ❶
  String year = ctx.pathParam("year");
  String month = ctx.pathParam("month");
  webClient
    .get(3001, "localhost", "/" + deviceId + "/" + year + "/" + month)
    .as(BodyCodec.jsonObject())
    .rxSend()
    .subscribe(
      resp -> forwardJsonOrStatusCode(ctx, resp),
      err -> sendBadGateway(ctx, err));
}
```

❶ 来自 JWT 令牌

设备标识符从 JWT 令牌数据中提取出来，并传递给 Web 客户端请求。

### 8.2.4 使用 Vert.x 发行 JWT 令牌

最后，但同样重要的是，我们需要生成 JWT 令牌。为此，我们需要向用户配置文件服务发出两个请求：首先我们需要检查凭证，然后收集配置文件数据以准备令牌。

以下列表展示了`/api/v1/token`路由的处理程序。

列表 8.14 JWT 令牌创建路由处理程序

```
private void token(RoutingContext ctx) {
  JsonObject payload = ctx.getBodyAsJson();                ❶
  String username = payload.getString("username");
  webClient
    .post(3000, "localhost", "/authenticate")              ❷
    .expect(ResponsePredicate.SC_SUCCESS)
    .rxSendJson(payload)
    .flatMap(resp -> fetchUserDetails(username))           ❸
    .map(resp -> resp.body().getString("deviceId"))
    .map(deviceId -> makeJwtToken(username, deviceId))     ❹
    .subscribe(
      token -> sendToken(ctx, token),
      err -> handleAuthError(ctx, err));
}

private void sendToken(RoutingContext ctx, String token) {
  ctx.response().putHeader("Content-Type", "application/jwt").end(token);
}

private void handleAuthError(RoutingContext ctx, Throwable err) {
  logger.error("Authentication error", err);
  ctx.fail(401);
}
```

❶ 我们从/api/v1/token 请求中提取凭证。

❷ 我们首先发起一个认证请求。

❸ 成功后，我们再次请求以获取配置文件数据。

❹ 我们准备令牌。

这是一个典型的 RxJava 异步操作组合，使用`flatMap`来链式请求。您还可以看到 Vert.x 路由器的声明式 API，其中我们可以指定我们期望第一个请求是成功的。

以下列表展示了`fetchUserDetails`的实现，该实现是在认证请求成功后获取用户配置文件数据。

列表 8.15 获取用户详细信息

```
private Single<HttpResponse<JsonObject>> fetchUserDetails(String username) {
  return webClient
    .get(3000, "localhost", "/" + username)
    .expect(ResponsePredicate.SC_OK)         ❶
    .as(BodyCodec.jsonObject())
    .rxSend();
}
```

❶ 我们期望成功。

最后，下一个列表展示了如何准备 JWT 令牌。

列表 8.16 准备 JWT 令牌

```
private String makeJwtToken(String username, String deviceId) {
  JsonObject claims = new JsonObject()                          ❶
    .put("deviceId", deviceId);
  JWTOptions jwtOptions = new JWTOptions()
    .setAlgorithm("RS256")
    .setExpiresInMinutes(10_080) // 7 days
    .setIssuer("10k-steps-api")                                 ❷
    .setSubject(username);
  return jwtAuth.generateToken(claims, jwtOptions);
}
```

❶ 我们的定制声明

❷ 一个在 JWT 规范中的声明

`JWTOptions`类提供了 JWT RFC 中常见声明的相关方法，例如发行者、过期日期和主题。您可以看到我们没有指定令牌的发行时间，尽管`JWTOptions`中有一个方法可以做到这一点。`jwtAuth`对象在这里做了正确的事情，并代表我们添加了它。

## 8.3 跨源资源共享（CORS）

我们有一个公开的 API，它将请求转发到内部服务，并且这个 API 使用 JWT 令牌进行身份验证和访问控制。我在命令行上也演示了我们可以与 API 交互。实际上，*任何*第三方应用程序都可以通过 HTTP 与我们的 API 通信：手机应用程序、另一个服务、桌面应用程序等等。您可能会认为 Web 应用程序也可以从运行在 Web 浏览器中的 JavaScript 代码中与 API 通信，但事实（幸运的是！）并非如此简单。

### 8.3.1 问题是什么？

网络浏览器强制执行安全策略，其中之一就是*同源策略*。假设我们从 https://my.tld:4000/js/app.js 加载 app.js：

+   app.js 允许向 https://my.tld:4000/api/foo/bar 发出请求。

+   app.js 不允许向 https://my.tld:4001/a/b/c 发出请求，因为不同的端口不是同一个源。

+   app.js 不允许向 https://other.tld/123 发出请求，因为不同的主机不是同一个源。

跨源资源共享 (CORS) 是一种机制，允许服务允许来自其他 *源* 的传入请求 ([`fetch.spec.whatwg.org/`](https://fetch.spec.whatwg.org/))。例如，暴露 https://other.tld/123 的服务可以指定允许来自 https://my.tld:4000 的代码或来自 *任何* 源的跨源请求。这允许当请求源允许时，网络浏览器可以继续执行跨源请求；否则，它将拒绝请求，这是默认行为。

当触发跨源请求时，例如加载一些 JSON 数据、图片或网络字体，网络浏览器会向服务器发送带有请求资源的请求，并传递一个 `Origin` HTTP 头部。然后服务器响应一个带有允许源的 `Access-Control-Allow-Origin` HTTP 头部，如图 8.4 所示。

![](img/CH08_F04_Ponge.png)

图 8.4 CORS 交互示例

`"*"` 的值表示任何源都可以访问资源，而像 `https://my.tld` 这样的值则表示只有来自 https://my.tld 的跨源请求被允许。在图 8.4 中，请求成功并带有 JSON 有效负载，但如果 CORS 策略禁止调用，当尝试执行跨源请求时，app.js 代码将收到错误。

根据跨源 HTTP 请求的类型，网络浏览器会执行 *简单* 或 *预检* 请求。图 8.4 中的请求是一个简单的请求。相比之下，一个 `PUT` 请求需要预检请求，因为它可能产生副作用（`PUT` 表示修改资源），因此必须向资源发送预检 `OPTIONS` HTTP 请求以检查 CORS 策略，然后允许时再执行实际的 `PUT` 请求。预检请求提供了更多细节，例如允许的 HTTP 头部和方法，因为服务器可以，例如，有一个禁止执行 `DELETE` 请求或包含 `ABC` 头部的 CORS 策略。我建议阅读 Mozilla 的“跨源资源共享 (CORS)”文档 ([`mng.bz/X0Z6`](http://mng.bz/X0Z6))，因为它提供了详细且易于理解的浏览器和服务器之间 CORS 交互的解释。

### 8.3.2 使用 Vert.x 支持 CORS

Vert.x 随带一个可用的 CORS 处理器，名为 `CorsHandler` 类。创建 `CorsHandler` 实例需要三个设置：

+   允许的源模式

+   允许的 HTTP 头部

+   允许的 HTTP 方法

下面的列表展示了如何在 Vert.x 路由器中安装 CORS 支持。

列表 8.17 在路由器中安装 CORS 支持

```
Set<String> allowedHeaders = new HashSet<>();        ❶
allowedHeaders.add("x-requested-with");
allowedHeaders.add("Access-Control-Allow-Origin");
allowedHeaders.add("origin");
allowedHeaders.add("Content-Type");
allowedHeaders.add("accept");
allowedHeaders.add("Authorization");

Set<HttpMethod> allowedMethods = new HashSet<>();    ❷
allowedMethods.add(HttpMethod.GET);
allowedMethods.add(HttpMethod.POST);
allowedMethods.add(HttpMethod.OPTIONS);
allowedMethods.add(HttpMethod.PUT);

router.route().handler(CorsHandler                   ❸
  .create("*")
  .allowedHeaders(allowedHeaders)
  .allowedMethods(allowedMethods));
```

❶ 允许的 HTTP 头部集合

❷ 允许的 HTTP 方法集合

❸ 适用于所有源的 CORS 处理器

HTTP 方法是我们 API 所支持的。你可以看到，例如，我们不支持`DELETE`。由于所有路由都是 API 的一部分，并且应该可以从任何类型的应用程序（包括网页浏览器）访问，因此已经为所有路由安装了 CORS 处理器。允许的头部应该与你的 API 需求相匹配，同时也应该与客户端可能传递的内容类型或代理可能注入的头部相匹配，以及用于分布式跟踪目的的头部。

我们可以通过向 API 支持的某个路由发出 HTTP `OPTIONS`预检请求来检查 CORS 是否得到适当的支持。

列表 8.18 检查 CORS 支持

```
$ http OPTIONS :4000/api/v1/token Origin:'http://foo.tld'
HTTP/1.1 405 Method Not Allowed
access-control-allow-origin: *
content-length: 0
```

通过指定一个`origin` HTTP 头部，CORS 处理器会在响应中插入一个`access-control-allow-origin` HTTP 头部。HTTP 状态码是 405，因为特定的路由不支持`OPTION` HTTP 方法，但这不是问题，因为当网页浏览器进行预检请求时，它们只对与 CORS 相关的头部感兴趣。

## 8.4 现代网络前端

我们已经讨论了公共 API 中的有趣点：如何使用 Vert.x 网络客户端进行 HTTP 请求，如何使用 JWT 令牌，以及如何启用 CORS 支持。现在是时候看看我们如何公开用户网络应用程序（在第七章中定义），以及该应用程序如何连接到公共 API。

应用程序是用 Vue.js JavaScript 框架编写的。Vert.x 用于提供应用程序的编译资源：HTML、CSS 和 JavaScript。

相应的源代码位于书籍源代码仓库的 part2-steps-challenge/user-webapp 文件夹中。

### 8.4.1 Vue.js

Vue.js 值得有一本书来专门介绍，如果你对这个框架感兴趣，我们建议你阅读 Erik Hanchett 和 Benjamin Listwon 的《Vue.js in Action》（Manning，2018）。由于我们正在使用 Vue.js 作为两个作为更大 10k steps 应用程序一部分开发的网络应用程序的 JavaScript 框架，所以我将在这里提供一个快速概述。

Vue.js 是一个现代 JavaScript 前端框架，类似于 React 或 Angular，用于构建现代网络应用程序，包括单页应用程序。它是*响应式*的，因为组件模型的变化会触发用户界面的变化。假设我们在网页上显示温度。当相应的数据发生变化时，温度会更新，Vue.js 会负责（大多数）完成这项工作的管道。

Vue.js 支持组件，其中可以将 HTML 模板、CSS 样式和 JavaScript 代码组合在一起，如下面的列表所示。

列表 8.19 Vue.js 组件的画布

```
<template>
  <div id="app">
    {{ hello }}                     ❶
  </div>
</template>

<style scoped>                      ❷
  div {
    border: solid 1px black;
  }
</style>

<script>
  export default {
    data() {
      return {
        hello: "Hello, world!"      ❸
      }
    }
  }
</script>
```

❶ 被 hello 属性的值替换

❷ 组件本地的 CSS 规则

❸ hello 属性的初始定义

可以使用 Vue.js 命令行界面创建 Vue.js 项目([`cli.vuejs.org/`](https://cli.vuejs.org/))：

```
$ vue create user-webapp
```

然后，可以使用 `yarn` 构建工具安装依赖项（`yarn install`），以自动实时重新加载的方式开发项目（`yarn run serve`），以及构建项目的生产版本 HTML、CSS 和 JavaScript 资产（`yarn run build`）。

### 8.4.2 Vue.js 应用程序结构和构建集成

用户网络应用程序是一个单页应用程序，有三个不同的屏幕：登录表单、用户详情页面和注册表单。

关键 Vue.js 文件如下：

+   src/main.js--入口点

+   src/router.js--将分发到三个不同屏幕组件的 Vue.js 路由

+   src/DataStore.js--一个使用网络浏览器本地存储 API 保存应用程序存储的对象，所有屏幕共享

+   src/App.vue--挂载 Vue.js 路由的主组件

+   src/views--包含三个屏幕组件：`Home.vue`、`Login.vue` 和 `Register.vue`

Vue.js 路由配置如下所示。

列表 8.20 Vue.js 路由配置

```
import Vue from 'vue'
import Router from 'vue-router'
import Home from './views/Home.vue'
import Login from './views/Login.vue'
import Register from './views/Register.vue'

Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/',        ❶
      name: 'home',     ❷
      component: Home   ❸
    },
    {
      path: '/login',
      name: 'login',
      component: Login
    },
    {
      path: '/register',
      name: 'register',
      component: Register
    },
  ]
})
```

❶ 组件路径

❷ 组件名称

❸ 组件引用

应用程序代码与为用户网络应用程序提供服务的 Vert.x 应用程序位于同一模块中，因此你将在 src/main/java 下找到常规的 Java 源文件，以及一个 Gradle build.gradle.kts 文件。Vue.js 编译资源（`yarn build`）必须复制到 src/main/resources/webroot/assets，以便基于 Vert.x 的服务提供它们。

这使得在单个项目中存在两个构建工具，幸运的是，它们可以和平共存。实际上，从 Gradle 中调用 `yarn` 非常容易，因为 com.moowork.node Gradle 插件提供了一个自包含的 Node 环境。以下列表显示了用户网络应用程序 Gradle 构建文件的 Node 相关配置。

列表 8.21 使用 com.moowork.node Gradle 插件

```
import com.moowork.gradle.node.yarn.YarnTask
apply(plugin = "com.moowork.node")                             ❶
tasks.register<YarnTask>("buildVueApp") {                      ❷
  dependsOn("yarn_install")                                    ❸
  // (...)                                                     ❹
  args = listOf("build")                                       ❺
}
tasks.register<Copy>("copyVueDist") {                          ❻
  dependsOn("buildVueApp")
  from("$projectDir/dist")
  into("$projectDir/src/main/resources/webroot/assets")
}
val processResources by tasks.named("processResources") {      ❼
  dependsOn("copyVueDist")
}
val clean by tasks.named<Delete>("clean") {                    ❽
  delete("$projectDir/dist")
  delete("$projectDir/src/main/resources/webroot/assets")
}
// (...)
```

❶ 使用 Node 插件

❷ 创建一个调用 yarn 的任务

❸ 添加了对先运行 yarn install 的依赖

❹ 你可以在完整源代码中找到的 Gradle 缓存指令

❺ 调用 yarn build

❻ 复制编译资源的任务

❼ 确保构建项目时也构建 Vue.js 应用程序。

❽ 为 Vue.js 编译资源执行额外清理任务

`buildVueApp` 和 `copyVueDist` 任务作为常规项目构建任务的一部分被插入，因此项目构建了 Java Vert.x 代码和 Vue.js 代码。我们还自定义了 `clean` 任务以删除生成的资源。

### 8.4.3 后端集成示例

让我们看看一个 Vue.js 组件：如图 8.5 所示的登录屏幕。

![](img/CH08_F05_Ponge.png)

图 8.5 登录屏幕截图

该组件的文件位于 src/views/Login.vue。该组件显示登录表单，提交时必须调用公共 API 获取 JWT 令牌。成功后，必须在本地存储 JWT 令牌，然后切换视图到 `home` 组件。出错时，必须停留在登录表单并显示错误信息。

组件的 HTML 模板部分如下所示。

列表 8.22 登录组件 HTML 模板

```
<template>
  <div>
    <div class="alert alert-danger" role="alert" 
    ➥ v-if="alertMessage.length > 0">                                   ❶
      {{ alertMessage }}                                                 ❷
    </div>
    <form v-on:submit="login">                                           ❸
      <div class="form-group">
        <label for="username">User name</label>
        <input type="username" class="form-control" id="username" 
 ➥ placeholder="somebody123" v-model="username">                ❹
      </div>
      <div class="form-group">
        <label for="password">Password</label>
        <input type="password" class="form-control" id="password" placeholder="abc123" v-model="password">
      </div>
      <button type="submit" class="btn btn-primary">Submit</button>
    </form>
    <div>
      <p>...or <router-link to="/register">register</router-link></p>    ❺
    </div>
  </div>
</template>
```

❶ 根据 alertMessage 组件数据值有条件地显示 div 块。

❷ 模板语法用于渲染 alertMessage 的值

❸ 在表单提交时调用登录方法。

❹ `v-model`将字段值绑定到用户名组件数据。

❺ `<router-link>`允许链接到另一个组件。

组件的 JavaScript 部分提供了组件数据声明以及`login`方法实现。我们使用 Axios JavaScript 库对公共 API 进行 HTTP 客户端调用。以下列表提供了组件 JavaScript 代码。

列表 8.23 登录组件 JavaScript 代码

```
import DataStore from '../DataStore'
import axios from 'axios'

export default {
  data() {                                                             ❶
    return {
      username: '',
      password: '',
      alertMessage: ''
    }
  },
  methods: {                                                           ❷
    login: function () {
      if (this.username.length === 0 || this.password.length === 0) {  ❸
        return
      }
      axios
        .post("http://localhost:4000/api/v1/token", {                  ❹
          username: this.username,
          password: this.password
        })
        .then(response => {
          DataStore.setToken(response.data)                            ❺
          DataStore.setUsername(this.username)
          this.$router.push({name: 'home'})                            ❻
        })
        .catch(err => this.alertMessage = err.message)                 ❼
    }
  }
}
```

❶ 组件数据声明

❷ 组件方法声明

❸ 如果任一字段为空，尝试对公共 API 进行身份验证是没有意义的。

❹ 以 JSON 有效负载的形式使用凭证发出身份验证请求。

❺ 如果成功，从响应中存储令牌和用户名。

❻ 告诉路由器更改组件。

❼ 当 alertMessage 的值改变时，触发错误消息的响应式显示

当用户在用户名和密码字段中输入文本时，组件数据属性会更新，并在表单提交时调用`login`方法。如果调用成功，应用程序将移动到`home`组件。

以下列表来自 Home.vue 组件的代码，展示了如何使用 JWT 令牌获取用户的总步数。

列表 8.24 使用 Axios 与 JWT 令牌

```
axios
  .get(`http://localhost:4000/api/v1/${DataStore.username()}/total`, {
    headers: {
      'Authorization': `Bearer ${DataStore.token()}`          ❶
    }
  })
  .then(response => this.totalSteps = response.data.count)    ❷
  .catch(err => {
    if (err.response.status === 404) {
      this.totalSteps = 0
    } else {
      this.alertMessage = err.message
    }
  })
```

❶ 从登录组件获取的值中传递令牌。

❷ 更新组件数据，触发视图刷新。

现在我们来看看如何使用 Vert.x 提供 Web 应用程序资源。

### 8.4.4 使用 Vert.x 提供静态内容

Vert.x 代码除了启动 HTTP 服务器和提供静态内容外，没有太多要做。以下列表显示了`UserWebAppVerticle`类的`rxStart`方法的内容。

列表 8.25 使用 Vert.x 提供静态内容

```
@Override
public Completable rxStart() {
  Router router = Router.router(vertx);

  router.route().handler(StaticHandler.create("webroot/assets"));    ❶
  router.get("/*").handler(ctx -> ctx.reroute("/index.html"));       ❷

  return vertx.createHttpServer()
    .requestHandler(router)
    .rxListen(HTTP_PORT)
    .ignoreElement();
}
```

❶ 在类路径中解析 webroot/assets 的静态内容。

❷ 别名 /* 到 /index.html.

`StaticHandler`在内存中缓存文件，除非在调用`create`方法时进行其他配置。在开发模式下禁用缓存很有用，因为你可以修改静态资源的内容，并通过在网页浏览器中重新加载来查看更改，而无需重新启动 Vert.x 服务器。默认情况下，静态文件是从类路径中的 webroot 文件夹解析的，但你可以通过指定 webroot/assets 来覆盖它，就像我们做的那样。

既然我们已经讨论了如何使用 Vert.x Web 栈，现在是时候关注测试组成响应式应用程序的服务了。

## 8.5 编写集成测试

测试是一个非常重要的关注点，尤其是在 10k 步挑战反应式应用程序的制作中涉及多个服务时。测试用户 Web 应用程序服务是否正确提供静态内容是没有意义的，但拥有涵盖与公共 API 服务交互的测试至关重要。让我们讨论如何为该服务编写集成测试。

公共 API 源代码揭示了一个 `IntegrationTest` 类。它包含几个按顺序排列的测试方法，用于检查 API 行为：

1.  注册一些用户。

1.  为每个用户获取 JWT 令牌。

1.  获取用户数据。

1.  尝试获取另一个用户的数据。

1.  更新用户数据。

1.  检查用户的一些活动统计信息。

1.  尝试检查另一个用户的活动。

由于公共 API 服务依赖于活动和用户配置文件服务，我们需要在测试执行期间使用 *fake* 服务来模拟它们，或者像数据库一样与所有依赖项一起部署。任何一种方法都可以。在本部分的章节中，我们有时会创建一个模拟服务来运行我们的集成测试，有时我们只是部署实际的服务。

在这种情况下，我们将部署实际的服务，并且需要以自包含和可重复的方式从 JUnit 5 中进行。我们首先需要添加项目依赖项，如下所示。

列表 8.26 运行集成测试的测试依赖项

```
testImplementation(project(":user-profile-service"))                         ❶
testImplementation(project(":activity-service"))
testImplementation("io.vertx:vertx-pg-client:$vertxVersion")                 ❷
testImplementation("org.testcontainers:junit-jupiter:$testContainersVersion")❸

testImplementation("org.junit.jupiter:junit-jupiter-api:$junit5Version")
testImplementation("io.vertx:vertx-junit5:$vertxVersion")
testImplementation("io.vertx:vertx-junit5-rx-java2:$vertxVersion")
testImplementation("io.rest-assured:rest-assured:$restAssuredVersion")       ❹
testImplementation("org.assertj:assertj-core:$assertjVersion")
```

❶ 依赖另一个项目模块

❷ 这用于在 PostgreSQL 中插入数据。稍后会有更多介绍。

❸ 这是运行 Docker 容器的。

❹ 一个用于测试 HTTP 服务的优秀 DSL 库

这些依赖项为我们提供了两个用于编写测试的有用工具：

+   Testcontainers 是一个在 JUnit 测试中运行 Docker 容器的项目，因此我们将能够使用像 PostgreSQL 或 Kafka 这样的基础设施服务（[www.testcontainers.org](http://www.testcontainers.org)）。

+   REST Assured 是一个专注于测试 HTTP 服务的库，提供了一个方便的流畅 API 来描述请求和响应断言（[`rest-assured.io`](http://rest-assured.io)）。

测试类的序言如下所示。

列表 8.27 集成测试类的序言

```
@ExtendWith(VertxExtension.class)                                    ❶
@TestMethodOrder(OrderAnnotation.class)                              ❷
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
@DisplayName("Integration tests for the public API")
@Testcontainers                                                      ❸
class IntegrationTest {

  @Container
  private static final DockerComposeContainer CONTAINERS =
    new DockerComposeContainer(new File("../docker-compose.yml"));   ❹
  // (...)
}
```

❶ 使用 Vert.x JUnit 5 支持。

❷ 测试方法必须按顺序运行。

❸ 使用 Testcontainers 支持。

❹ 从 Docker Compose 文件启动容器。

Testcontainers 提供了启动一个或多个容器的大量选择。它支持通用的 Docker 镜像、针对常见基础设施（PostgreSQL、Apache Kafka 等）的专用类，以及 Docker Compose。在这里，我们重用 Docker Compose 描述符来运行整个应用程序（`docker-compose.yml`），在运行第一个测试之前启动文件中描述的容器。所有测试执行完毕后，容器将被销毁。这非常有趣——我们可以编写针对实际基础设施服务的集成测试，这些服务将在生产中使用。

`prepareSpec` 方法被 `@BeforeAll` 注解，用于准备测试。它为活动服务在 PostgreSQL 数据库中插入一些数据，然后部署用户配置文件和活动垂直服务。它还从 REST Assured 准备一个 `RequestSpecification` 对象，如下所示。

列表 8.28 准备 REST Assured 请求规范

```
requestSpecification = new RequestSpecBuilder()
  .addFilters(asList(new ResponseLoggingFilter(), new RequestLoggingFilter()))❶
  .setBaseUri("http://localhost:4000/")
  .setBasePath("/api/v1")                                                     ❷
  .build();
```

❶ 所有请求和响应都将被记录，这对于跟踪错误很有用。

❷ 这避免了在请求中重复所有 URL 的基本路径。

此对象在所有测试方法之间共享，因为它们都必须向 API 发送请求。我们启用对所有请求和响应的记录，以便更容易调试，并将 `/api/v1` 设置为所有请求的基本路径。

测试类维护一个用户注册的哈希表，稍后用于调用，以及一个 JWT 令牌的哈希表。

列表 8.29 集成测试的实用哈希表

```
private final HashMap<String, JsonObject> registrations = new 
➥ HashMap<String, JsonObject>() {                               ❶
  {
    put("Foo", new JsonObject()
      .put("username", "Foo")
      .put("password", "foo-123")
      .put("email", "foo@email.me")
      .put("city", "Lyon")
      .put("deviceId", "a1b2c3")
      .put("makePublic", true));
    // (...)
};

private final HashMap<String, String> tokens = new HashMap<>();  ❷
```

❶ 用户

❷ 一旦检索到 JWT 令牌

以下列表是第一个测试，其中注册了来自 `registrations` 哈希表的用户。

列表 8.30 注册用户的测试

```
@Test
@Order(1)
@DisplayName("Register some users")
void registerUsers() {
  registrations.forEach((key, registration) -> {
    given(requestSpecification)
      .contentType(ContentType.JSON)
      .body(registration.encode())     ❶
      .post("/register")               ❷
      .then()
      .assertThat()
      .statusCode(200);                ❸
  });
}
```

❶ 我们将 JSON 数据编码为字符串。

❷ 向 `/api/v1/register` 发送 HTTP POST 请求

❸ 断言状态码是 200。

REST Assured 流畅 API 允许我们表达我们的请求，然后对响应进行断言。我们可以将响应作为文本或 JSON 提取以进行进一步的断言，如下所示，这是从检索 JWT 令牌的测试方法中提取的。

列表 8.31 获取 JWT 令牌的测试代码

```
JsonObject login = new JsonObject()
  .put("username", key)
  .put("password", registration.getString("password"));

String token = given(requestSpecification)
  .contentType(ContentType.JSON)
  .body(login.encode())
  .post("/token")
  .then()
  .assertThat()
  .statusCode(200)
  .contentType("application/jwt")    ❶
  .extract()                         ❷
  .asString();
assertThat(token)                    ❸
  .isNotNull()
  .isNotBlank();

tokens.put(key, token);
```

❶ 断言响应中的内容类型头与 JWT 令牌匹配。

❷ 提取响应。

❸ 在字符串上使用 AssertJ 断言

测试获取一个令牌，然后断言该令牌既不是 `null` 值也不是空白字符串（空或包含空格）。提取 JSON 数据的方式类似，如下所示。

列表 8.32 使用 REST Assured 提取 JSON

```
JsonPath jsonPath = given(requestSpecification)
  .headers("Authorization", "Bearer " + tokens.get("Foo"))         ❶
  .get("/Foo/total")
  .then()
  .assertThat()
  .statusCode(200)
  .contentType(ContentType.JSON)
  .extract()
  .jsonPath();

assertThat(jsonPath.getInt("count")).isNotNull().isEqualTo(6255);  ❷
```

❶ 传递 JWT 令牌。

❷ 与 JSON 表示形式一起工作。

测试获取用户 `Foo` 的总步骤数，提取 JSON 响应，然后检查步骤数（JSON 响应中的 `count` 键）等于 6255。

集成测试可以使用 Gradle (`./gradlew :public-api:test`) 或从开发环境运行，如图 8.6 所示。

![图片](img/CH08_F06_Ponge.png)

图 8.6 从 IntelliJ IDEA 运行集成测试

您现在对使用 Vert.x 网络堆栈来公开端点和消费其他服务有了很好的理解。下一章将重点介绍 Vert.x 的消息和事件流堆栈。

## 摘要

+   Vert.x 网络模块使得构建支持 CORS 和对其他服务进行 HTTP 调用的边缘服务变得简单。

+   JSON 网络令牌在公共 API 的授权和访问控制中非常有用。

+   Vert.x 对前端应用程序框架没有偏好，但它很容易集成 Vue.js 前端应用程序。

+   通过结合由 Testcontainers 管理的 Docker 容器和 Rest Assured 库，您可以编写针对 HTTP API 的集成测试。
