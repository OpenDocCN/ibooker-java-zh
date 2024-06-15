# 第十四章 开发 Quarkus 应用程序使用 Spring API

到目前为止，您已经看到每个示例都是使用 CDI 注解（如 `@Inject` 或 `@Produces`）、JAX-RS 注解或 Java EE 安全性注解开发的。但是 Quarkus 也为一些最常用的 Spring 库提供了兼容层，因此您可以使用关于 Spring Framework 的所有知识来开发 Quarkus 应用程序。

本章将包括以下几种方法：

+   Spring 依赖注入

+   Spring REST Web

+   Spring Data JPA

+   Spring Security

+   Spring Boot 配置

# 14.1 使用 Spring 依赖注入

## 问题

您希望使用 Spring 依赖注入（DI）API 来开发 Quarkus 应用程序。

## 解决方案

Quarkus 提供了一个 API 兼容层（使用扩展）来使用 Spring DI 注解。

虽然我们鼓励您使用 CDI 注解，但您也可以自由使用 Spring 注解，因为最终应用程序的行为方式完全相同。

开发了一个问候服务，就像书的开头一样。如果您熟悉 Spring Framework，很多东西看起来会很眼熟。

要添加 Spring DI 扩展，请运行以下命令：

```java
./mvnw quarkus:add-extension -Dextensions="spring-di"
```

或者，您可以通过以下方式创建一个带有 Spring DI 扩展的项目：

```java
mvn io.quarkus:quarkus-maven-plugin:1.4.1.Final:create \
 -DprojectGroupId=org.acme.quickstart \
 -DprojectArtifactId=spring-di-quickstart \
 -DclassName="org.acme.quickstart.GreeterResource" \
 -Dpath="/greeting" \
 -Dextensions="spring-di"
```

打开 *application.properties* 文件并添加一个新的属性：

```java
greetings.message=Hello World
```

要将此配置值注入到任何类中，请使用 `@org.springframework.beans.factory.annotation.Value` Spring 注解：

```java
package org.acme.quickstart;

import java.util.function.Function;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration ![1](img/1.png)
public class AppConfiguration {

    @Bean(name = "suffix") ![2](img/2.png)
    public Function<String, String> exclamation() {
        return new Function<String, String>() { ![3](img/3.png)
            @Override
            public String apply(String t) {
                return t + "!!";
            }
        };
    }
}
```

![1](img/#co_developing_quarkus_applications___span_class__keep_together__using_spring_apis__span__CO1-1)

使用 `@Configuration` 注解将类定义为配置对象

![2](img/#co_developing_quarkus_applications___span_class__keep_together__using_spring_apis__span__CO1-2)

创建一个向给定消息添加后缀的 bean

![3](img/#co_developing_quarkus_applications___span_class__keep_together__using_spring_apis__span__CO1-3)

实现服务

请注意，在这两种情况下都使用了 Spring 注解。

可以使用 `@org.springframework.beans.factory.annotation.Autowired` 和 `@org.springframework.beans.factory.annotation.Qualifier` 注解来注入 bean：

```java
package org.acme.quickstart;

import org.springframework.stereotype.Service;

@Service ![1](img/1.png)
public class PrefixService {

    public String appendPrefix(String message) {
        return "- " + message;
    }

}
```

![1](img/#co_developing_quarkus_applications___span_class__keep_together__using_spring_apis__span__CO2-1)

将此类设置为服务

并且可以使用构造函数而不是 `@Autowired` 进行注入：

```java
private PrefixService prefixService;

public GreetingResource(PrefixService prefixService) { ![1](img/1.png)
    this.prefixService = prefixService;
}
```

![1](img/#co_developing_quarkus_applications___span_class__keep_together__using_spring_apis__span__CO3-1)

使用构造函数注入实例；请注意，不需要 `@Autowired`

最后，可以将所有这些操作结合起来产生以下输出：

```java
@GET
@Produces(MediaType.TEXT_PLAIN)
public String hello() {
    String prefixed = prefixService.appendPrefix(message);
    return this.suffixComponent.apply(prefixed);
}
```

如果运行项目，您将能够看到所有对象都正确创建和注入，即使使用了 Spring DI 注解：

```java
./mvnw compile quarkus:dev

curl http://localhost:8080/greeting

- Hello World!!
```

## 讨论

需要注意的是，Quarkus 不会启动一个 Spring 应用上下文实例。这是因为它的集成仅在 API 层面（注解、返回类型等），这意味着以下内容：

+   使用任何其他 Spring 库将没有任何效果。稍后你会看到 Quarkus 提供了对其他 Spring 库的集成。

+   `org.springframework.beans.factory.config.BeanPostProcessor` 将不会被执行。

表 14-1 显示了 MicroProfile/CDI 与 Spring 之间的等效注解。

表 14-1\. MicroProfile/CDI 与 Spring 的等效注解

| Spring | CDI / MicroProfile |
| --- | --- |

|

`@Autowired`

|

`@Injecct`

|

|

`@Qualifier`

|

`@Named`

|

|

`@Value`

|

`@ConfigProperty`。典型用例的表达式语言是支持的。

|

|

`@Component`

|

`@Singleton`

|

|

`@Service`

|

`@Singleton`

|

|

`@Repository`

|

`@Singleton`

|

|

`@Configuration`

|

`@ApplicationScoped`

|

|

`@Bean`

|

`@Produces`

|

# 14.2 使用 Spring Web

## 问题

你希望使用 Spring Web API 在 Quarkus 中进行开发。

## 解决方案

Quarkus 提供了一个 API 兼容层（使用扩展）来使用 Spring Web 注解/类。

要添加 Spring Web 扩展，运行以下命令：

```java
./mvnw quarkus:add-extension -Dextensions="spring-web"
```

仅使用 Spring Web 注解创建一个新的资源：

```java
package org.acme.quickstart;

import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController ![1](img/1.png)
@RequestMapping("/greeting") ![2](img/2.png)
public class SpringController {

  @GetMapping ![3](img/3.png)
  public ResponseEntity<String> getMessage() { ![4](img/4.png)
    return ResponseEntity.ok("Hello");
  }

  @GetMapping("/{name}")
  public String hello(@PathVariable(name = "name") String name) { ![5](img/5.png)
    return "hello " + name;
  }
}
```

![1](img/#co_developing_quarkus_applications___span_class__keep_together__using_spring_apis__span__CO4-1)

REST 资源定义

![2](img/#co_developing_quarkus_applications___span_class__keep_together__using_spring_apis__span__CO4-2)

映射根路径

![3](img/#co_developing_quarkus_applications___span_class__keep_together__using_spring_apis__span__CO4-3)

设置 `GET` HTTP 方法

![4](img/#co_developing_quarkus_applications___span_class__keep_together__using_spring_apis__span__CO4-4)

返回 Spring 的 `ResponseEntity`

![5](img/#co_developing_quarkus_applications___span_class__keep_together__using_spring_apis__span__CO4-5)

获取路径信息

仅使用 Spring 注解和类来实现资源。没有 JAX-RS API 的痕迹。

## 讨论

你已经看到如何在 Quarkus 应用中使用 Spring 依赖注入注解。Quarkus 通过扩展与 Spring Web 集成。

虽然我们鼓励你使用 JAX_RS 注解，但你可以自由使用 Spring Web 类和注解。无论使用哪种，最终应用的行为都会是一样的。

Quarkus 仅支持 Spring Web 的与 REST 相关的特性。总之，所有 `@RestController` 的特性都得到了支持，除了与通用 `@Controller` 相关的那些特性。

支持以下 Spring Web 注解：`@RestController`、`@RequestMapping`、`@GetMapping`、`@PostMapping`、`@PutMapping`、`@DeleteMapping`、`@PatchMapping`、`@RequestParam`、`@RequestHeader`、`@MatrixVariable`、`@PathVariable`、`@CookieValue`、`@RequestBody`、`@ResponseStatus`、`@ExceptionHandler`（仅用于 `@RestControllerAdvice` 类）、以及 `@RestControllerAdvice`（仅支持 `@ExceptionHandler` 能力）。

默认支持的 REST 控制器返回类型包括：基本类型、`String`（作为文字，而非 MVC 支持）、POJO 和 `org.springframework.http.ResponseEntity`。

默认支持的 REST 控制器方法参数包括：基本类型、`String`、POJO、`javax.servlet.http.HttpServletRequest` 和 `javax.servlet.http.HttpServletResponse`。

默认支持的异常处理程序返回类型包括：`org.springframework.http.ResponseEntity` 和 `java.util.Map`。

下列是默认支持的方法参数，用于异常处理：`java.lang.Exception` 或任何其他子类型，`javax.servlet.http.HttpServletRequest` 和 `javax.servlet.http.HttpServletResponse`。

###### 重要提示

要使用 `javax.servlet` 类，您需要注册 `quarkus-undertow` 依赖项。

表 14-2 显示了 JAX-RS 和 Spring Web 之间的等效注解。

表 14-2\. JAX-RS 和 Spring Web 中的等效注解

| Spring | JAX-RS |
| --- | --- |

|

`@RequestController`

|  |
| --- |

|

`@RequestMapping(path="/api")`

|

`@Path("/api")`

|

|

`@RequestMapping(consumes="application/json")`

|

`@Consumes("application/json")`

|

|

`@RequestMapping(produces="application/json")`

|

`@Produces("application/json")`

|

|

`@RequestParam`

|

`@QueryParam`

|

|

`@PathVariable`

|

`@PathParam`

|

|

`@RequestBody`

|  |
| --- |

|

`@RestControllerAdvice`

|  |
| --- |

|

`@ResponseStatus`

|

使用 `javax.ws.rs.core.Response` 类

|

|

`@ExceptionHandler`

|

实现 `javax.ws.rs.ext.ExceptionMapper` 接口

|

# 14.3 使用 Spring Data JPA

## 问题

您想要使用 Spring Data JPA API 在 Quarkus 中开发持久层。

## 解决方案

Quarkus 提供了一个 API 兼容层（使用扩展）来使用 Spring Data JPA 类。

要添加 Spring Web 扩展，请运行以下命令：

```java
./mvnw quarkus:add-extension -Dextensions="spring-data-jpa"
```

使用 Panache 或 Spring Data JPA 的主要区别在于，您的 repository 类必须实现 Spring Data 的 `org.springframework.data.repository.CrudRepository` 接口，而不是 `io.quarkus.hibernate.orm.panache.PanacheRepository`。但其他部分，如在 *application.properties* 中定义实体或配置数据源，完全相同。

创建一个名为 `org.acme.quickstart.DeveloperRepository` 的新类：

```java
package org.acme.quickstart;

import java.util.List;

import org.springframework.data.repository.CrudRepository;

public interface DeveloperRepository extends CrudRepository<Developer, Long> { ![1](img/1.png)
    List<Developer> findByName(String name); ![2](img/2.png)
}
```

![1](img/#co_developing_quarkus_applications___span_class__keep_together__using_spring_apis__span__CO5-1)

定义一个 Spring Data JPA CRUD repository

![2](img/#co_developing_quarkus_applications___span_class__keep_together__using_spring_apis__span__CO5-2)

派生查询方法

运行项目并发送一些请求来验证对象是否使用 Spring Data 接口持久化。要执行此操作，请在终端中运行以下命令：

```java
./mvnw compile quarkus:dev

curl -d '{"name":"Ada"}' -H "Content-Type: application/json" \
 -X POST http://localhost:8080/developer -v

< HTTP/1.1 201 Created
< Content-Length: 0
< Location: http://localhost:8080/developer/1
<
* Connection #0 to host localhost left intact
```

## 讨论

在 第七章 中，您了解了如何使用 Quarkus 开发持久性代码，特别是使用 Panache 框架。Quarkus 还通过扩展与 Spring Data JPA 集成。

尽管我们鼓励您使用 Panache 框架，但您也可以自由地使用 Spring Data JPA 类，因为最终应用程序的行为完全相同。

Quarkus 支持 Spring Data JPA 的一部分功能，基本上是最常用的功能。

支持定义存储库的以下接口：

+   `org.springframework.data.repository.Repository`

+   `org.springframework.data.repository.CrudRepository`

+   `org.springframework.data.repository.CrudRepository`

+   `org.springframework.data.repository.PagingAndSortingRepository`

+   `org.springframework.data.jpa.repository.JpaRepository`

###### 重要

更新数据库的方法会自动用 `@Transactional` 注释。如果您使用 `@org.springframework.data.jpa.repository.Query`，则需要使用 `@org.springframework.data.jpa.repository.Modifying` 注释方法使其具有事务性。

在撰写本文时，以下功能不受支持：

+   `org.springframework.data.repository.query.QueryByExampleExecutor` 的方法

+   QueryDSL 支持

+   自定义代码库中所有存储库接口的基本存储库

+   `java.util.concurrent.Future` 或其子类作为存储库方法的返回类型

+   使用 `@Query` 时的本机和命名查询

这些限制可能在不久的将来得到修复。

# 14.4 使用 Spring Security

## 问题

您想要使用 Spring Security API 来保护资源。

## 解决方案

Quarkus 提供了一个 API 兼容性层（使用扩展）来使用 Spring Security 类。

要添加 Spring Security 扩展（以及身份提供者），请运行以下命令：

```java
./mvnw quarkus:add-extension \
 -Dextensions="quarkus-spring-security, quarkus-spring-web, \
 quarkus-elytron-security-properties-file"
```

从这一点开始，可以使用 Spring Security 注解 (`org.springframework.security.access.annotation.Secured` 和 `org.springframework.security.access.prepost.PreAuthorize`) 代替 Java EE 注解 (`@javax.annotation.security.RolesAllowed`) 来保护代码：

```java
package org.acme.quickstart;

import org.springframework.security.access.annotation.Secured;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/hello")
public class GreetingController {
    @GetMapping
    @Secured("admin") ![1](img/1.png)
    public String helloAdmin() {
        return "hello from admin";
    }

    @PreAuthorize("hasAnyRole('user')") ![2](img/2.png)
    @GetMapping
    @RequestMapping("/any")
    public String helloUsers() {
        return "hello from users";
    }
}
```

![1](img/#co_developing_quarkus_applications___span_class__keep_together__using_spring_apis__span__CO6-1)

`@Secured` 注解

![2](img/#co_developing_quarkus_applications___span_class__keep_together__using_spring_apis__span__CO6-2)

`@PreAuthorize` 注解以添加表达式支持

在 *application.properties* 文件中注册有效用户和角色，因为 Elytron 文件属性扩展已注册为身份提供者：

```java
quarkus.security.users.embedded.enabled=true
quarkus.security.users.embedded.plain-text=true
quarkus.security.users.embedded.users.alexandra=aixa
quarkus.security.users.embedded.roles.alexandra=admin,user
quarkus.security.users.embedded.users.ada=dev
quarkus.security.users.embedded.roles.ada=user
```

只有 `alexandra` 可以访问两个端点，而 `ada` 只能访问用户端点。

## 讨论

在 第十一章 中，您学习了如何使用 Java EE 安全注解（`@RolwsAllowed`）保护 RESTful Web 服务。 Quarkus 通过扩展集成了 Spring Security，允许使用 Spring Security 注解。

需要注意的是，Spring 安全集成发生在 API 层面，像 Elytron 文件属性这样的身份提供者实现仍然是必需的。

Quarkus 支持 Spring Security `@PreAuthorze` 表达式语言的子集，基本上是最常用功能的集合：

```java
@PreAuthorize("hasRole('admin')")
@PreAuthorize("hasRole(@roles.USER)") ![1](img/1.png)

@PreAuthorize("hasAnyRole(@roles.USER, 'view')")

@PreAuthorize("permitAll()")
@PreAuthorize("denyAll()")

@PreAuthorize("isAnonymous()")
@PreAuthorize("isAuthenticated()")
```

![1](img/#co_developing_quarkus_applications___span_class__keep_together__using_spring_apis__span__CO7-1)

其中 `roles` 是使用 `@Component` 注解定义的 bean，而 `USER` 是类的公共字段

还支持条件表达式：

```java
@PreAuthorize("#person.name == authentication.principal.username")
public void doSomethingElse(Person person){}

@PreAuthorize("@personChecker.check(#person,
 authentication.principal.username)")
public void doSomething(Person person){}

@Component
public class PersonChecker {
    public boolean check(Person person, String username) {
        return person.getName().equals(username);
    }
}

@PreAuthorize("hasAnyRole('user', 'admin') AND #user == principal.username")
public void allowedForUser(String user) {}
```

# 14.5 使用 Spring Boot 属性

## 问题

您想要使用 Spring Boot 将配置属性映射到 Java 对象。

## 解决方案

Quarkus 提供了一个 API 兼容性层（使用扩展），用于使用 Spring Boot 配置属性。

Quarkus 与 Spring Boot 集成为一个扩展，因此可以使用 `@org.springframework.boot.context.properties.ConfigurationProperties` 注解将配置属性映射到 Java 对象。

要添加 Spring Boot 扩展（以及其他 Spring 集成），请运行以下命令：

```java
./mvnw quarkus:add-extension \
 -Dextensions="quarkus-spring-di, quarkus-spring-web, \
 quarkus-spring-boot-properties,
 quarkus-hibernate-validator"
```

添加一些要绑定到 Java 对象的配置属性：

```java
greeting.message=Hello World
greeting.configuration.uppercase=true
```

下一步是创建带有 getter/setter 的 POJOs，将文件中的配置属性绑定到 Java 对象。 需要注意的是，属性 `uppercase` 定义在名为 `configuration` 的子类别中，这影响到了 POJO 类的创建方式，因为每个子类别必须添加到自己的类中：

```java
package org.acme.quickstart;

import javax.validation.constraints.Size;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "greeting") ![1](img/1.png)
public class GreetingConfiguration {

    @Size(min = 2) ![2](img/2.png)
    private String message;
    private Configuration configuration; ![3](img/3.png)

    public void setMessage(String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

    public void setConfiguration(Configuration configuration) {
        this.configuration = configuration;
    }

    public Configuration getConfiguration() {
        return configuration;
    }

}
```

![1](img/#co_developing_quarkus_applications___span_class__keep_together__using_spring_apis__span__CO8-1)

使用 `ConfigurationProperties` 注解父类，并设置配置属性的前缀

![2](img/#co_developing_quarkus_applications___span_class__keep_together__using_spring_apis__span__CO8-2)

支持 Bean 验证注释

![3](img/#co_developing_quarkus_applications___span_class__keep_together__using_spring_apis__span__CO8-3)

子类别 `configuration` 映射到具有相同名称的字段中

子类别 POJO 只是具有 `uppercase` 属性的 Java 类。

```java
package org.acme.quickstart;

public class Configuration {

    private boolean uppercase;

    public boolean isUppercase() {
        return uppercase;
    }

    public void setUppercase(boolean uppercase) {
        this.uppercase = uppercase;
    }
}
```

配置对象被注入到任何类中，就像使用 `@Inject` 或 `@Autowired` 注解其他 bean 一样：

```java
@Autowired ![1](img/1.png)
GreetingConfiguration greetingConfiguration;

@GetMapping
public String hello() {
    if (greetingConfiguration.getConfiguration().isUppercase()) { ![2](img/2.png)
        return greetingConfiguration.getMessage().toUpperCase();
    }
    return greetingConfiguration.getMessage();
}
```

![1](img/#co_developing_quarkus_applications___span_class__keep_together__using_spring_apis__span__CO9-1)

注入配置对象并将数据绑定到其中

![2](img/#co_developing_quarkus_applications___span_class__keep_together__using_spring_apis__span__CO9-2)

配置属性会自动填充到 Java 对象中
