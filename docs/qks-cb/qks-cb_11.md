# 第十一章：身份验证和授权

在本章中，您将了解身份验证和授权如何在 Quarkus 应用程序中发挥作用，这是应用程序安全性的支柱。我们将讨论以下主题：

+   基于文件的认证和授权方案

+   基于数据库的身份验证和授权方案

+   外部服务支持的认证和授权方案

# Quarkus 安全性基础知识

在我们开始第一个配方之前，本节将向您展示 Quarkus 和安全性的基础知识，您将使用安全性扩展来加载身份验证源，以及如何使用*基于角色的访问控制*（RBAC）方法保护资源。

本节中显示的示例并不意味着可运行，但它们将是即将出现的配方的基础，其中我们将看到安全性扩展的实际操作。

下面是关于安全性的两个主要概念：

认证

验证您的凭据（即用户名/密码）以验证您的身份，以便系统知道您是谁。

授权

验证您对受保护资源的访问权。这发生在身份验证过程之后。

## 认证

Quarkus 为 HTTP 提供了两种认证机制，即众所周知的`BASIC`和`FORM`方法。这些机制可以由任何 Quarkus 扩展进行扩展，以提供自定义的身份验证方法。这些机制的示例以 Quarkus 扩展的形式提供，用于对抗 OpenID Connect 服务器（如 Keycloak）进行身份验证。我们将在本节中探讨如何做到这一点。

要使用身份验证，需要身份提供者来验证用户提供的凭据（即用户名/密码）。Quarkus 提供了以下开箱即用的身份提供者，但您也可以实现自己的身份提供者：

Elytron 属性文件

以属性文件的形式提供用户/密码/角色之间的映射。这些信息可以嵌入在*application.properties*文件中，也可以放在一个专门用于此目的的文件中。

Elytron JDBC

基于 JDBC 查询提供用户/密码/角色之间的映射。

JPA

提供通过 JPA 进行身份验证的支持。

SmallRye JWT

提供使用 JSON Web Tokens（JWT）规范进行身份验证。

OIDC

提供使用 OpenID Connect（OIDC）提供程序（如 Keycloak）进行身份验证。

Keycloak 授权

提供使用 Keycloak 授权服务的策略执行器支持。

### 基本身份验证

要使用*基本*访问认证进行身份验证，必须将`quarkus.http.auth.basic`配置属性设置为`true`。

### 基于表单的身份验证

为了使用*表单*访问认证进行身份验证，必须将`quarkus.http.auth.form.enabled`配置属性设置为`true`。

###### 重要

Quarkus 不会将经过身份验证的用户存储在 HTTP 会话中，因为没有集群化的 HTTP 会话支持。相反，身份验证信息存储在加密的 cookie 中。

可以使用`quarkus.http.auth.session.encryption-key`属性来设置加密密钥，必须至少为 16 个字符长。该密钥使用 SHA-256 进行哈希，结果用作 AES-256 加密 Cookie 值的密钥。此 Cookie 包含作为加密值的一部分的到期时间，在使用会话时每分钟生成一个新的 Cookie，并更新到期时间。

## 授权

Quarkus 与[Java EE 安全注解](https://oreil.ly/ATPpq)集成，以定义对 RESTful Web 端点和 CDI bean 的 RBAC。

此外，您可以使用配置文件（*application.properties*）而不是注解来定义 RESTful Web 端点的授权。

这两种方法可以在同一个应用程序中共存，但是配置文件检查会在任何注解检查之前执行，并且它们不是互斥的，这意味着在重叠的情况下，必须通过两个检查。

下面的代码段展示了如何使用 Java EE 安全注解来保护 JAX-RS 端点：

```java
package org.acme.quickstart;

import javax.annotation.security.DenyAll;
import javax.annotation.security.PermitAll;
import javax.annotation.security.RolesAllowed;

import io.quarkus.security.Authenticated;

import javax.ws.rs.GET;
import javax.ws.rs.Path;

@Path("/hello")
public class GreetingResource {

    @GET
    @Path("/secured")
    @RolesAllowed("Tester")  ![1](img/1.png)
    public String greetingSecured() {}

    @GET
    @Path("/unsecured")
    @PermitAll ![2](img/2.png)
    public String greetingUnsecured() {}

    @GET
    @Path("/denied")
    @DenyAll ![3](img/3.png)
    public String greetingDenied() {}

    @GET
    @Path("/authenticated")
    @Authenticated ![4](img/4.png)
    public String greetingAuthenticated() {}
}
```

![1](img/#co_authentication_and_authorization_CO1-1)

要求具有`Tester`角色的经过身份验证的用户

![2](img/#co_authentication_and_authorization_CO1-2)

未经身份验证的用户可以访问该方法

![3](img/#co_authentication_and_authorization_CO1-3)

无论是否经过身份验证，都不能访问用户。

![4](img/#co_authentication_and_authorization_CO1-4)

允许任何经过身份验证的用户访问；它是`@RolesAllowed("*")`的别名，由 Quarkus 提供，而不是规范。

可以使用`javax.ws.rs.core.Context`注解来注入`javax.ws.rs.core.SecurityContext`实例，以获取有关已验证用户的信息：

```java
@GET
@Path("/secured")
@RolesAllowed("Tester")
public String greetingSecured(@Context SecurityContext sec) { ![1](img/1.png)
    Principal user = sec.getUserPrincipal(); ![2](img/2.png)
    String name = user != null ? user.getName() : "anonymous";
    return name;
}
```

![1](img/#co_authentication_and_authorization_CO2-1)

为当前请求注入`SecurityContext`

![2](img/#co_authentication_and_authorization_CO2-2)

获取当前登录的用户

###### 重要提示

安全注解不仅限于 JAX-RS 资源。它们也可以用于 CDI bean 中来保护方法调用。

Quarkus 支持使用配置文件而不是注解来配置 RESTful Web 端点。可以通过配置文件表示等效的安全注解示例：

```java
quarkus.http.auth.policy.role-policy1.roles-allowed=Tester ![1](img/1.png)

quarkus.http.auth.permission.roles1.paths=/hello/secured ![2](img/2.png)
quarkus.http.auth.permission.roles1.policy=role-policy1 ![3](img/3.png)
quarkus.http.auth.permission.roles1.methods=GET ![4](img/4.png)

quarkus.http.auth.permission.deny1.paths=/hello/denied
quarkus.http.auth.permission.deny1.policy=deny ![5](img/5.png)

quarkus.http.auth.permission.permit1.paths=/hello/unsecured
quarkus.http.auth.permission.permit1.policy=permit ![6](img/6.png)
quarkus.http.auth.permission.permit1.methods=GET

quarkus.http.auth.permission.roles2.paths=/hello/authenticated
quarkus.http.auth.permission.roles2.policy=authenticated
quarkus.http.auth.permission.roles2.methods=GET
```

![1](img/#co_authentication_and_authorization_CO3-1)

定义应用程序的角色；`role-policy1`用作参考值

![2](img/#co_authentication_and_authorization_CO3-2)

设置资源的权限；`roles1`是一个任意的名称，以避免重复的键。

![3](img/#co_authentication_and_authorization_CO3-3)

设置角色策略

![4](img/#co_authentication_and_authorization_CO3-4)

限制对`GET`方法的权限

![5](img/#co_authentication_and_authorization_CO3-5)

拒绝访问

![6](img/#co_authentication_and_authorization_CO3-6)

允许访问

需要注意的是`paths`属性支持逗号分隔的多个值，还支持`*`通配符来匹配任何子路径。比如，`quarkus.http.auth.permission.permit1.paths=/public/_,/robots.txt`设置了对位于*/public*及其子路径下的任意资源和文件*/robots.txt*的权限。

同样，`methods`属性允许用逗号分隔的多个值。

有两个配置属性影响 RBAC 的行为：

`quarkus.security.jaxrs.deny-unannotated-endpoints`

如果设置为`true`，则所有未标记安全注解的 JAX-RS 端点默认情况下将被拒绝访问。此属性默认值为`false`。

`quarkus.security.deny-unannotated-members`

如果设置为`true`，那么所有未标记安全注解的 JAX-RS 端点和 CDI 方法默认情况下将被拒绝。此属性默认值为`false`。

到目前为止，你已经看到在 Quarkus 中可以设置授权过程（基本的、表单的，或者其他扩展提供的）并使用安全注解或在配置文件中指定身份验证角色。

本章中的示例将探讨不同的 Quarkus 扩展以提供身份验证和授权身份提供者。

# 11.1 使用 Elytron 属性文件配置进行身份验证和授权

## 问题

你想通过存储身份来保护应用程序。

## 解决方案

Quarkus 安全性支持使用 Elytron 属性文件配置将身份存储在文件中作为身份提供者。

你已经看到了如何定义身份验证机制以及如何使用 RBAC 保护资源，无论是通过安全注解还是在*application.properties*中，但你还没有看到如何注册身份提供者以及如何存储用户信息，比如用户名、密码或角色。

现在让我们看看如何使用 Elytron 属性文件配置扩展定义身份信息。该扩展基于属性文件来定义所有身份信息，其主要目的是用于开发和测试。不推荐在生产环境中使用，因为密码只能以明文或 MD5 哈希的形式表示。

要启用 Elytron 属性文件配置，需要注册`quarkus-elytron-security-properties-file`扩展：

```java
./mvnw quarkus:add-extension \
 -Dextensions="quarkus-elytron-security-properties-file"
```

该扩展支持使用属性文件的组合将用户映射到密码和用户映射到角色。

保护端点，只允许`Tester`角色访问资源：

```java
@GET
@Produces(MediaType.TEXT_PLAIN)
@RolesAllowed("Tester")
public String hello() {
    return "hello";
}
```

要注册身份，需要两个属性文件，一个用于将用户和密码映射，另一个用于将用户和他们所属的角色列表映射。

用户配置属性文件定义了每一行用户和密码的配对，在系统中注册：

```java
alex=soto
```

在用户属性文件中，关键部分是用户名，值部分是密码。

###### 警告

注意密码为明文。您可以按照以下模式使用 MD5 对密码进行哈希：`HEX(MD5(username:realm:password)`。

角色配置文件为每行定义了用户名和用户所属角色（用逗号分隔）：

```java
alex=Tester
```

在角色属性文件中，关键部分是用户名，值是分配给用户的角色。

最后，Elytron 安全属性文件扩展需要配置用户和角色属性文件的类路径位置：

```java
quarkus.http.auth.basic=true ![1](img/1.png)

quarkus.security.users.file.enabled=true ![2](img/2.png)
quarkus.security.users.file.plain-text=true ![3](img/3.png)
quarkus.security.users.file.users=users.properties ![4](img/4.png)
quarkus.security.users.file.roles=roles.properties
```

![1](img/#co_authentication_and_authorization_CO4-1)

启用基本认证方法。

![2](img/#co_authentication_and_authorization_CO4-2)

启用具有属性文件扩展的安全性

![3](img/#co_authentication_and_authorization_CO4-3)

设置未使用 MD5 哈希的密码

![4](img/#co_authentication_and_authorization_CO4-4)

设置用户和角色属性文件的类路径位置。

运行生成的测试以验证端点的保护：

```java
./mvnw clean test

...
INFO  [io.quarkus] (main) Installed features:
 [cdi, resteasy, security, security-properties-file]
[ERROR] Tests run: 2, Failures: 1, Errors: 0, Skipped: 0, Time elapsed: 8.485 s
 <<< FAILURE! - in org.acme.quickstart.GreetingResourceTest
[ERROR] testHelloEndpoint  Time elapsed: 0.076 s  <<< FAILURE!
java.lang.AssertionError:
1 expectation failed.
Expected status code <200> but was <401>.
 at org.acme.quickstart.GreetingResourceTest.testHelloEndpoint
 (GreetingResourceTest.java:17)
```

测试失败，显示 HTTP 401 未经授权的错误，因为测试未使用基本认证方法提供任何身份验证。修改测试以使用配置的用户名和密码进行身份验证：

```java
@Test
public void testSecuredHelloEndpoint() {
    given()
            .auth() ![1](img/1.png)
            .basic("alex", "soto") ![2](img/2.png)
            .when()
            .get("/hello")
            .then()
            .statusCode(200)
            .body(is("hello"));
}
```

![1](img/#co_authentication_and_authorization_CO5-1)

设置认证部分。

![2](img/#co_authentication_and_authorization_CO5-2)

使用给定的用户名和密码进行基本认证

现在，使用有效的认证参数，测试通过。

## 讨论

Elytron 属性文件配置扩展还支持在 Quarkus 配置文件（*application.properties*）中嵌入用户/密码/角色的映射，而不是使用不同的文件。

```java
quarkus.security.users.embedded.enabled=true
quarkus.security.users.embedded.plain-text=true
quarkus.security.users.embedded.users.alex=soto
quarkus.security.users.embedded.roles.alex=Admin,Tester
```

存储在文件中的密码可以使用公式`HEX(MD5(username ":" realm ":" password))`进行哈希。

可以通过使用表 11-1 中列出的属性来配置嵌入式 Elytron 属性文件。

表 11-1\. 嵌入式 Elytron 属性

| 属性 | 描述 |
| --- | --- |

|

`quarkus.security.users.embedded.realm-name`

|

生成哈希密码时使用的域名（默认为`Quarkus`）。

|

|

`quarkus.security.users.embedded.enabled`

|

使用具有属性文件扩展的安全性（默认为`false`）。

|

|

`quarkus.security.users.embedded.plain-text`

|

设置密码是否为哈希形式。如果设为`true`，则哈希密码必须采用`HEX(MD5(username:realm:password)`的格式（默认为`false`）。

|

|

`quarkus.security.users.embedded.users.<user>`

|

用户信息。关键部分是用户名，值部分是密码。

|

|

`quarkus.security.users.embedded.roles.<user>`

|

角色信息。关键部分是用户名，值部分是密码。

|

# 11.2 使用 Elytron 安全性 JDBC 配置进行认证和授权

## 问题

您希望保护应用程序并将用户身份存储在数据库中。

## 解决方案

Quarkus 安全性提供支持，可以使用 Elytron 安全性 JDBC 配置将用户身份存储在数据源中作为身份提供者。

您已经看到如何使用 Elytron 属性文件配置扩展在属性文件中定义身份，详见 Recipe 11.1。然而，正如所述，此方法更适用于测试/开发目的，不应在生产环境中使用。

Elytron 安全性 JDBC 扩展可用于将用户身份存储在数据库中，支持使用 bcrypt 密码映射器进行密码加密，并且足够灵活，不会将您锁定在任何预定义的数据库模式中。

要启用 Elytron 安全性 JDBC 扩展，您需要注册`quarkus-elytron-security-jdbc`扩展，用于连接数据库的 JDBC 驱动程序，以及可选的 Flyway 来填充模式和一些默认用户：

```java
./mvnw quarkus:add-extension \
 -Dextensions="quarkus-elytron-security-jdbc,quarkus-jdbc-h2,quarkus-flyway"
```

保护终端点，仅允许具有`Tester`角色的用户访问资源：

```java
@GET
@RolesAllowed("Tester")
@Produces(MediaType.TEXT_PLAIN)
public String hello() {
    return "hello";
}
```

下一步是定义数据库模式以存储所有 RBAC 信息。为简单起见，此示例中使用了一个包含用户、密码和角色的简单表格：

```java
CREATE TABLE test_user (
  id INT,
  username VARCHAR(255),
  password VARCHAR(255),
  role VARCHAR(255)
);

INSERT INTO test_user (id, username, password, role)
  VALUES (1, 'alex', 'soto', 'Tester');
```

最后，必须配置扩展以指定要执行的查询来验证用户并检索他们所属的角色：

```java
quarkus.datasource.url=jdbc:h2:mem:mydb
quarkus.datasource.driver=org.h2.Driver
quarkus.datasource.username=sa
quarkus.datasource.password=

quarkus.flyway.migrate-at-start=true

quarkus.security.jdbc.enabled=true ![1](img/1.png)
quarkus.security.jdbc.principal-query.sql=\
  SELECT u.password, u.role FROM test_user u WHERE u.username=? ![2](img/2.png)
quarkus.security.jdbc.principal-query.clear-password-mapper.enabled=true ![3](img/3.png)
quarkus.security.jdbc.principal-query.clear-password-mapper\
  .password-index=1![4](img/4.png)
quarkus.security.jdbc.principal-query.attribute-mappings.0.index=2 ![5](img/5.png)
quarkus.security.jdbc.principal-query.attribute-mappings.0.to=groups
```

![1](img/#co_authentication_and_authorization_CO6-1)

启用 Elytron 安全性 JDBC。

![2](img/#co_authentication_and_authorization_CO6-2)

定义验证用户并获取角色的查询；查询必须包含一个参数（用户名），并且至少返回密码，值应与键在同一行。

![3](img/#co_authentication_and_authorization_CO6-3)

密码以明文存储。

![4](img/#co_authentication_and_authorization_CO6-4)

设置密码的索引；所有内容应在同一行上

![5](img/#co_authentication_and_authorization_CO6-5)

设置角色的索引，并指定该字段为角色

###### 重要提示

索引从 1 开始。

###### 贴士

检索密码的查询（以及可选的角色）可以根据模型的要求复杂化（即，SQL 连接）。

现在，身份验证和授权数据是从数据库中检索，而不是从文件中。当提供用户名和密码（例如，使用基本的`auth`方法）时，将执行查询以检索身份验证过程所需的所有信息（将提供的密码与从数据库检索到的密码进行匹配），并获取授权过程中的角色。

记得更新测试（如果之前未完成）以使其通过：

```java
@Test
public void testSecuredHelloEndpoint() {
    given()
      .auth().basic("alex", "soto")
    .when()
      .get("/hello")
    .then()
      .statusCode(200)
      .body(is("hello"));
}
```

## 讨论

在此示例中，您使用了明文密码，显然不应在生产环境中使用。该扩展提供了与`bcrypt`密码映射器集成，因此身份验证过程也适用于哈希密码。

您需要通过向配置文件添加一些额外参数来指示 Elytron Security JDBC 使用`bcrypt`加密密码，并且不应将其作为明文进行比较。

不再配置 `clear-password-mapper`，而是使用 `bcrypt-password-mapper`。以下是使用 `bcrypt` 的配置文件示例：

```java
quarkus.security.jdbc.enabled=true
quarkus.security.jdbc.principal-query.sql=\
    SELECT u.password, u.role, u.salt, u.iteration \
    FROM test_user u WHERE u.username=?

quarkus.security.jdbc.principal-query.clear-password-mapper.enabled=false
quarkus.security.jdbc.principal-query.bcrypt-password-mapper.enabled=true ![1](img/1.png)
quarkus.security.jdbc.principal-query.bcrypt-password-mapper.password-index=\
    1 ![2](img/2.png)
quarkus.security.jdbc.principal-query.bcrypt-password-mapper.hash-encoding=\
    BASE64 ![3](img/3.png)
quarkus.security.jdbc.principal-query.bcrypt-password-mapper.salt-index=\
    3 ![4](img/4.png)
quarkus.security.jdbc.principal-query.bcrypt-password-mapper.salt-encoding=\
    BASE64 ![5](img/5.png)
quarkus.security.jdbc.principal-query.bcrypt-password-mapper.\
    iteration-count-index=4 ![6](img/6.png)

quarkus.security.jdbc.principal-query.attribute-mappings.0.index=2
quarkus.security.jdbc.principal-query.attribute-mappings.0.to=groups
```

![1](img/#co_authentication_and_authorization_CO7-1)

启用 bcrypt

![2](img/#co_authentication_and_authorization_CO7-2)

设置密码索引；这应该在同一行。

![3](img/#co_authentication_and_authorization_CO7-3)

设置密码哈希编码；这应该在同一行。

![4](img/#co_authentication_and_authorization_CO7-4)

设置盐索引；这应该在同一行。

![5](img/#co_authentication_and_authorization_CO7-5)

设置盐编码；这应该在同一行。

![6](img/#co_authentication_and_authorization_CO7-6)

设置迭代计数索引；这应该在同一行。

在此更改后，提供的密码与通过查询检索的密码不再以明文形式匹配。相反，提供的密码使用 bcrypt 进行哈希处理，然后与存储的密码进行比较：

```java
quarkus.security.jdbc.enabled=true

quarkus.security.jdbc.principal-query.sql=\
    SELECT u.password FROM test_user u WHERE u.username=? ![1](img/1.png)
quarkus.security.jdbc.principal-query.clear-password-mapper.enabled=true
quarkus.security.jdbc.principal-query.clear-password-mapper.password-index=1

quarkus.security.jdbc.principal-query.roles.sql=\
    SELECT r.role_name FROM test_role r, test_user_role ur \
    WHERE ur.username=? AND ur.role_id = r.id ![2](img/2.png) ![3](img/3.png)
quarkus.security.jdbc.principal-query.roles.datasource=permissions ![4](img/4.png)
quarkus.security.jdbc.principal-query.roles.attribute-mappings.0.index=1
quarkus.security.jdbc.principal-query.roles.attribute-mappings.0.to=groups
```

![1](img/#co_authentication_and_authorization_CO8-1)

默认数据源用于检索密码。

![2](img/#co_authentication_and_authorization_CO8-2)

`roles` 用作标识第二个查询的名称；查询应该全部位于同一行。

![3](img/#co_authentication_and_authorization_CO8-3)

从另一个查询获取角色。

![4](img/#co_authentication_and_authorization_CO8-4)

角色查询针对名为`permissions`的数据源执行。

# 11.3 使用 MicroProfile JWT 进行授权

## 问题

您希望在 RESTful web 服务和通常的无状态服务中保存安全上下文。

## 解决方案

使用 JSON Web Tokens。

JWT（JSON Web Token）是根据 RFC-7519 规范制定的标准，用于在服务之间交换信息。JWT 的特殊之处在于，令牌内容以 JSON 格式而不是纯文本或任何其他二进制格式进行格式化。

Quarkus 与[MicroProfile JWT 规范](https://oreil.ly/IU0-d)集成，以消费和验证 JWT 令牌并获取声明。

JWT 令牌由*声明*组成，这些声明是要传输的信息，例如用户名、令牌的过期时间或用户的角色。令牌经过数字签名，因此可以信任和验证其中包含的信息。

JWT 令牌由三个部分组成，所有部分均使用 Base64 编码：

头部

它包含一些元数据，例如用于签署令牌的算法；令牌的自定义信息，例如令牌类型；或者如果使用 JSON Web Encryption (JWE)，则是未加密的声明。

声明

要存储在令牌中的信息。某些声明是强制性的，其他是可选的，还有一些是我们应用程序自定义的。

签名

令牌的签名。

然后将三个部分编码为 Base64 并用句点符号（`.`）连接，因此最终令牌看起来像是`base64(Header).base64(Claims).base64(Signature)`。

对于本示例，使用以下 JWT 令牌：

```java
{ ![1](img/1.png)
  "kid": "/privateKey.pem",
  "typ": "JWT",
  "alg": "RS256"
},
{ ![2](img/2.png)
  "sub": "jdoe-using-jwt-rbac",
  "aud": "using-jwt-rbac",
  "upn": "jdoe@quarkus.io",
  "birthdate": "2001-07-13",
  "auth_time": 1570094171,
  "iss": "https://quarkus.io/using-jwt-rbac", ![3](img/3.png)
  "roleMappings": {
    "group2": "Group2MappedRole",
    "group1": "Group1MappedRole"
  },
  "groups":  ![4
    "Echoer",
    "Tester",
    "Subscriber",
    "group2"
  ],
  "preferred_username": "jdoe",
  "exp": 2200814171,
  "iat": 1570094171,
  "jti": "a-123"
}
```

![1](img/#co_authentication_and_authorization_CO9-1)

头部部分

![2](img/#co_authentication_and_authorization_CO9-2)

Claim 部分

![3](img/#co_authentication_and_authorization_CO9-3)

令牌的颁发者

![4](img/#co_authentication_and_authorization_CO9-4)

属于令牌所有者的组（或角色）。

以下是相同令牌的序列化版本：

```java
eyJraWQiOiJcL3ByaXZhdGVLZXkucGVtIiwidHlwIjoiSldUIiwiYWxnIjoiUlMyNTYifQ.
eyJzdWIiOiJqZG9lLXVzaW5nLWp3dC1yYmFjIiwiYXVkIjoidXNpbmctand0LXJiYWMiLCJ
1cG4iOiJqZG9lQHF1YXJrdXMuaW8iLCJiaXJ0aGRhdGUiOiIyMDAxLTA3LTEzIiwiYXV0aF
90aW1lIjoxNTcwMDk0MTcxLCJpc3MiOiJodHRwczpcL1wvcXVhcmt1cy5pb1wvdXNpbmcta
nd0LXJiYWMiLCJyb2xlTWFwcGluZ3MiOnsiZ3JvdXAyIjoiR3JvdXAyTWFwcGVkUm9sZSIs
Imdyb3VwMSI6Ikdyb3VwMU1hcHBlZFJvbGUifSwiZ3JvdXBzIjpbIkVjaG9lciIsIlRlc3R
lciIsIlN1YnNjcmliZXIiLCJncm91cDIiXSwicHJlZmVycmVkX3VzZXJuYW1lIjoiamRvZS
IsImV4cCI6MjIwMDgxNDE3MSwiaWF0IjoxNTcwMDk0MTcxLCJqdGkiOiJhLTEyMyJ9.
Hzr41h3_uewy-g2B-sonOiBObtcpkgzqmF4bT3cO58v45AIOiegl7HIx7QgEZHRO4PdUtR3
4x9W23VJY7NJ545ucpCuKnEV1uRlspJyQevfI-mSRg1bHlMmdDt661-V3KmQES8WX2B2uqi
rykO5fCeCp3womboilzCq4VtxbmM2qgf6ag8rUNnTCLuCgEoulGwTn0F5lCrom-7dJOTryW
1KI0qUWHMMwl4TX5cLmqJLgBzJapzc5_yEfgQZ9qXzvsT8zeOWSKKPLm7LFVt2YihkXa80l
Wcjewwt61rfQkpmqSzAHL0QIs7CsM9GfnoYc0j9po83-P3GJiBMMFmn-vg
```

注意部分如何用句点分隔。

当接收到请求时，MicroProfile JWT 规范执行以下操作：

1.  从请求中提取安全令牌，通常从`Authorization`头部提取。

1.  验证令牌以确保令牌有效。这些检查可能涉及验证签名以信任令牌或验证令牌未过期。

1.  提取令牌信息。

1.  创建一个带有身份信息的安全上下文，以便在授权（RBAC）时使用。

此外，MicroProfile JWT 规范设置了每个令牌必须提供的强制声明列表：

| 声明 | 描述 |
| --- | --- |

|

`typ`

|

令牌格式。必须是`JWT`。

|

|

`alg`

|

标识用于保护令牌的加密算法。必须是`RS256`。

|

|

`kid`

|

指示用于保护令牌的密钥。

|

|

`iss`

|

令牌颁发者。

|

|

`sub`

|

标识受令牌约束的主体。

|

|

`aud`

|

标识令牌所针对的接收者。

|

|

`exp`

|

设置过期时间。

|

|

`iat`

|

提供令牌发放的时间。

|

|

`jti`

|

令牌的唯一标识符。

|

|

`upn`

|

用于`java.security.Principal`接口中的用户主体名称。

|

|

`groups`

|

已分配给令牌主体的组名列表。它们是用户所属的角色。

|

这些是 MicroProfile JWT 规范所需的最小声明，但可以添加额外的声明，例如`preferred_username`或任何您的应用程序可能需要在服务之间传输的其他信息。

注册`quarkus-smallrye-jwt`扩展以开始使用 MicroProfile JWT 规范：

```java
./mvnw quarkus:add-extension -Dextensions="quarkus-smallrye-jwt"
```

配置扩展以设置用于验证令牌是否未被修改以及服务器接受的令牌颁发者（`iss`）声明的公钥。

规范支持的公钥格式如下：

+   Public Key Cryptography Standards #8（PKCS#8）Privacy-Enhanced Mail（PEM）

+   JSON Web Key（JWK）

+   JSON Web Key Set（JWKS）

+   JSON Web Key（JWK）Base64 URL 编码

+   JSON Web Key Set（JWKS）Base64 URL 编码

对于本示例，我们选择 JSON Web Key Set（JWKS）格式来指定用于验证令牌的公钥。

包含公钥的 JWKS 文件放置在项目目录内：

```java
{
    "keys": [
        {
            "kty": "RSA",
            "kid": "/privateKey.pem",
            "e": "AQAB",
            "n": "livFI8qB4D0y2jy0CfEqFyy46R0o7S8TKpsx5xbHKoU1VWg6QkQm-ntyIv1
 p4kE1sPEQO73-HY8-Bzs75XwRTYL1BmR1w8J5hmjVWjc6R2BTBGAYRPFRho
 r3kpM6ni2SPmNNhurEAHw7TaqszP5eUF_F9-KEBWkwVta-PZ37bwqSE4sCb
 1soZFrVz_UT_LF4tYpuVYt3YbqToZ3pZOZ9AX2o1GCG3xwOjkc4x0W7ezbQ
 ZdC9iftPxVHR8irOijJRRjcPDtA6vPKpzLl6CyYnsIYPd99ltwxTHjr3npf
 v_3Lw50bAkbT4HeLFxTx4flEoZLKO_g0bAoV2uqBhkA9xnQ"
        }
    ]
}
```

指向此数据的配置文件如下：

```java
mp.jwt.verify.publickey.location=quarkus.pub.jwk.json ![1](img/1.png)
mp.jwt.verify.issuer=https://quarkus.io/using-jwt-rbac ![2](img/2.png)
```

![1](img/#co_authentication_and_authorization_CO10-1)

公钥的位置

![2](img/#co_authentication_and_authorization_CO10-2)

服务所接受的发行者

除了处理令牌的验证过程外，MicroProfile JWT 还集成了现有的 Java EE 安全 API，提供来自令牌的数据。集成发生在以下注解中：

```java
javax.ws.rs.core.SecurityContext.getUserPrincipal()
javax.ws.rs.core.SecurityContext.isUserInRole(String)
javax.servlet.http.HttpServletRequest.getUserPrincipal()
javax.servlet.http.HttpServletRequest.isUserInRole(String)
javax.ejb.SessionContext.getCallerPrincipal()
javax.ejb.SessionContext.isCallerInRole(String)
javax.security.jacc.PolicyContext
  .getContext("javax.security.auth.Subject.container")
javax.security.enterprise.identitystore.IdentityStore
  .getCallerGroups(CredentialValidationResult)
@javax.annotation.security.RolesAllowed
```

此外，MicroProfile JWT 规范提供了两个类，用于在 CDI 或 JAX-RS 类中存储 JWT 数据。

`org.eclipse.microprofile.jwt.JsonWebToken`

公开原始令牌并提供获取声明的方法的接口

`@org.eclipse.microprofile.jwt.Claim`

注释以在类中注入声明

例如：

```java
package org.acme.quickstart;

import javax.annotation.security.RolesAllowed;
import javax.enterprise.context.RequestScoped;
import javax.inject.Inject;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

import org.eclipse.microprofile.jwt.Claim;
import org.eclipse.microprofile.jwt.Claims;
import org.eclipse.microprofile.jwt.JsonWebToken;

@Path("/hello")
@RequestScoped ![1](img/1.png)
public class GreetingResource {

    @Inject
    JsonWebToken callerPrincipal; ![2](img/2.png)

    @Claim(standard = Claims.preferred_username) ![3](img/3.png)
    String username;

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String hello() {
        return "hello " + username;
    }
}
```

![1](img/#co_authentication_and_authorization_CO11-1)

JWT 令牌基于请求范围；如果希望使用令牌，则类必须为`RequestScoped`，以避免在类中混合令牌。

![2](img/#co_authentication_and_authorization_CO11-2)

注入代表完整 JWT 令牌的`JsonWebToken`接口

![3](img/#co_authentication_and_authorization_CO11-3)

注入`preferred_username`声明

声明注解还支持注入私有声明名称。这些声明不是 RFC 提供的官方声明名称，而是特定于服务的声明（自定义声明）。要注入私有声明，请使用注解值作为声明的名称：`@Claim("*my_claim*")`。此外，在非强制性声明的情况下，可以使用`java.util.Optional`类来指示声明可为空：

```java
@Claim(standard = Claims.birthdate)
Optional<String> birthdate;
```

更新测试以向定义的端点发送 bearer JWT 令牌：

```java
@Test
public void testHelloEndpoint() {
    given().header("Authorization", "Bearer " + validToken) ![1](img/1.png)
            .when().get("/hello").then().statusCode(200).body(is("hello jdoe"));
}
```

![1](img/#co_authentication_and_authorization_CO13-1)

JWT 令牌作为`Authorization`头中的 bearer 令牌发送

使用当前解决方案，这些假设是正确的：

+   如果提供了有效的令牌，则提取`preferred_username`。

+   如果提供了无效的令牌（过期、签名无效、被第三方修改等），则向调用者返回 401 未经授权错误代码。

+   如果未提供令牌，则会处理请求，但`preferred_username`字段为空。

MicroProfile JWT 规范还通过集成`@RolesAllowed`注解支持授权流程。每当调用`isCallerInRole()`方法时，都会使用`groups`声明值，这实际上意味着`groups`中的任何值都可以作为应用程序中的角色使用。

在此示例中使用的 JWT 令牌中的`groups`声明包含以下值：`"groups": ["Echoer", "Tester", "Subscriber", "group2"]`。通过使用`@RolesAllowed`保护对`/hello`的调用，其中令牌中存在一个组值：

```java
@GET
@Produces(MediaType.TEXT_PLAIN)
@RolesAllowed("Tester")
public String hello() {}
```

现在，您可以假设以下内容：

+   如果提供了有效的令牌，并且`groups`声明包含`Tester`组，则提取`preferred_username`。

+   如果提供了有效的令牌并且`groups`声明不包含`Tester`组，则返回给调用者 403 禁止错误代码。

+   如果提供了无效的令牌（过期、签名无效、被第三方修改等），则返回给调用者 401 未授权错误代码。

+   如果未提供令牌，则返回给调用者 401 未授权错误代码。

## 讨论

在过去，安全上下文保存在 HTTP 会话中，这在服务规模扩展并变得越来越复杂时效果良好。为避免此问题，一种可能的解决方案是在所有调用中使用令牌传递此信息，特别是 JSON 令牌。

需要注意的是令牌是签名的而不是加密的，这意味着信息可以被任何人看到但不能被修改。可以使用 JSON Web Encryption 添加加密层，使声明不是明文而是加密的。

本节的目的不是让您精通 JWT，而是让您了解如何在 Quarkus 中使用它，因此我们假设您已经具有一些关于 JWT 的知识。我们还在下面的“参见”部分提供了一些链接，以帮助您更熟悉 JWT。

## 参见

要了解 JWT，请访问以下网页：

+   [JSON Web Tokens](https://jwt.io)

+   [GitHub：MicroProfile 的 JWT RBAC](https://oreil.ly/tXP9d)

+   [IETF：JSON Web Token](https://oreil.ly/p9jUC)

# 11.4 使用 OpenId Connect 进行授权和认证

## 问题

您希望使用 OpenId Connect 保护您的 RESTful Web API。

## 解决方案

使用由 OpenId Connect 签发的承载令牌进行授权。

在前一节中，您学习了如何使用 JWT 令牌保护资源，但未涵盖令牌的生成，因为令牌是预先生成并在文本文件中提供的。

在实际应用程序中，您需要一个发行令牌的身份提供者。用于分布式服务的事实上协议是 OpenId Connect 和 OAuth 2.0 以及符合该协议的授权服务器，如[Keycloak](https://www.keycloak.org)。

注册`quarkus-oidc`扩展以保护使用 OpenId Connect 的资源：

```java
./mvnw quarkus:add-extension -Dextensions="quarkus-oidc"
```

配置 OpenId Connect 服务器的位置以验证令牌：

```java
quarkus.oidc.auth-server-url=http://localhost:8180/auth/realms/quarkus ![1](img/1.png)
quarkus.oidc.client-id=backend-service ![2](img/2.png)
```

![1](img/#co_authentication_and_authorization_CO14-1)

OpenID Connect 服务器的基本 URL

![2](img/#co_authentication_and_authorization_CO14-2)

每个应用程序都有一个客户端 ID 用于标识应用程序

使用`@RolesAllowed`注解保护端点：

```java
@Inject
io.quarkus.security.identity.SecurityIdentity securityIdentity; ![1](img/1.png)

@GET
@RolesAllowed("user")
@Produces(MediaType.TEXT_PLAIN)
public String hello() {
    return "hello " + securityIdentity.getPrincipal().getName();
}
```

![1](img/#co_authentication_and_authorization_CO15-1)

Quarkus 接口，表示当前已登录的用户

测试必须更新以从 OpenId Connect 获取访问令牌并将其提供为承载令牌：

```java
@Test
public void testHelloEndpoint() {
    System.out.println(accessToken);
    given()
      .auth().oauth2(accessToken)
      .when().get("/hello")
      .then()
         .statusCode(200)
         .body(is("hello alice"));
}
```

访问令牌由 OpenId Connect 服务器生成。要生成它，必须提供一些参数，例如用户名和密码，以访问服务器并生成代表用户的令牌：

```java
package org.acme.quickstart;

import java.net.URI;
import java.net.URISyntaxException;

import io.restassured.RestAssured;
import io.restassured.builder.RequestSpecBuilder;
import io.restassured.response.Response;
import io.restassured.response.ResponseOptions;
import io.restassured.specification.RequestSpecification;

public class RestAssuredExtension {

  public static ResponseOptions<Response> getAccessToken(String url,
                                                         String clientId,
                                                         String clientIdPwd,
                                                         String username,
                                                         String password) {
    final RequestSpecification request = prepareRequest(url);
    try {
      return request
        .auth()
        .preemptive()
        .basic(clientId, clientIdPwd)
        .contentType("application/x-www-form-urlencoded; charset=UTF-8")
        .urlEncodingEnabled(true)
        .formParam("username", username)
        .and()
        .formParam("password", password)
        .and()
        .formParam("grant_type", "password")
        .post(new URI(url));
    } catch (URISyntaxException e) {
      throw new IllegalArgumentException(e);
    }
  }

  private static RequestSpecification prepareRequest(String url) {
    final RequestSpecBuilder builder = new RequestSpecBuilder();
    final RequestSpecification requestSpec = builder.build();
    return RestAssured.given().spec(requestSpec);
  }
}
```

代码本质上是使用 REST-Assured 实现下一个`curl`命令的实现：

```java
curl -X POST \
    http://localhost:8180/auth/realms/quarkus/protocol/openid-connect/token \
    --user backend-service:secret \
    -H 'content-type: application/x-www-form-urlencoded' \
    -d 'username=alice&password=alice&grant_type=password'
```

现在，在运行测试时，显示的内容与 Recipe 11.3 完全不同。

首先，令牌（JWT 令牌）不是静态的；它是由 OpenID Connect（Keycloak）为`alice`用户名颁发的。

以下是针对`alice`的示例已颁发令牌：

```java
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "cfIADN_xxCJmVkWyN-PNXEEvMUWs2r68CxtmhEDNzXU"
},
{
  "jti": "cc54b9db-5f2f-4609-8a6b-4f76026e63ae",
  "exp": 1578935775,
  "nbf": 0,
  "iat": 1578935475,
  "iss": "http://localhost:8180/auth/realms/quarkus",
  "sub": "eb4123a3-b722-4798-9af5-8957f823657a",
  "typ": "Bearer",
  "azp": "backend-service",
  "auth_time": 0,
  "session_state": "5b674175-a2a9-4a45-a3da-394923125e55",
  "acr": "1",
  "realm_access": {
    "roles": [
      "user"
    ]
  },
  "scope": "email profile",
  "email_verified": false,
  "preferred_username": "alice"
}
```

其次，OpenID Connect 负责提供所有内容来验证令牌；不手动配置公钥。

当提供令牌时，Keycloak 执行以下验证：

+   如果提供了有效令牌并且`roles`声明包含`user`组，则提取`preferred_username`。

+   如果提供了有效令牌并且`roles`声明不包含`user`组，则会向调用方返回 403 Forbidden 错误代码。

+   如果提供了无效令牌（过期、签名无效、被第三方篡改等），则会向调用方返回 403 Forbidden 错误代码。

+   如果未提供令牌，则会向调用方返回 401 Unauthorized 错误代码。

## 参见

要了解有关 OpenId Connect 协议的更多信息，请参阅以下网站：

+   [OpenID Connect](https://openid.net/connect)

+   [Keycloak](https://www.keycloak.org)

# 11.5 使用 OpenId Connect 保护 Web 资源

## 问题

您希望保护您的 Web 资源。

## 解决方案

使用 OpenId Connect 和基于文件的角色定义来保护 Web 资源。

可以使用 OpenId Connect 协议和 Quarkus 保护 Web 资源。 OpenId Connect 扩展通过实现众所周知的授权码流程，使 Web 资源能够进行身份验证，如果尝试访问受保护资源的未经身份验证用户，则会将其重定向到 OpenId Connect 提供者网站进行身份验证。完成身份验证过程后，用户将被发送回应用程序。

注册`quarkus-oidc`扩展以保护 OpenId Connect 资源：

```java
./mvnw quarkus:add-extension -Dextensions="quarkus-oidc"
```

配置 OpenId Connect 服务器的位置以验证令牌：

```java
quarkus.oidc.auth-server-url=http://localhost:8180/auth/realms/quarkus ![1](img/1.png)
quarkus.oidc.client-id=frontend ![2](img/2.png)
quarkus.oidc.application-type=web-app ![3](img/3.png)
quarkus.http.auth.permission.authenticated.paths=/* ![4](img/4.png)
quarkus.http.auth.permission.authenticated.policy=authenticated
```

![1](img/#co_authentication_and_authorization_CO16-1)

OpenID Connect 服务器的基本 URL

![2](img/#co_authentication_and_authorization_CO16-2)

每个应用程序都有一个客户端 ID 用于标识应用程序

![3](img/#co_authentication_and_authorization_CO16-3)

启用 OpenID Connect 授权码流程

![4](img/#co_authentication_and_authorization_CO16-4)

设置 Web 资源的权限

启动应用程序，打开浏览器，并输入以下网址：[*http://localhost:8080*](http://localhost:8080):

```java
./mvnw clean compile quarkus:dev
```

默认的 *index.html* 页面不会显示，而是会重定向到 Keycloak 的认证页面。输入以下有效凭据（登录名：**`alice`**，密码：**`alice`**）即可访问 Web 资源。点击登录按钮后，页面将被重定向回登录页面。
