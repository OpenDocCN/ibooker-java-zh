# 第七章：持久化

Quarkus 使用的底层持久化策略应该已经很熟悉了。事务、数据源、Java Persistence API（JPA）等都是存在多年的标准。Quarkus 使用这些标准，并在某些情况下在其基础上进行扩展，以便更轻松地处理持久化存储。在本章中，您将学习如何在 Quarkus 中处理持久化存储。我们涵盖传统的关系型数据库管理系统（RDBMS）和 NoSQL 数据库。

如果您使用传统的关系型数据库管理系统或 MongoDB，则 Quarkus 通过 Panache 提供了一些额外的宝贝，Panache 是一种基于实体或活动记录的 API，具有明确的观点。Panache 简化了大部分标准 JPA 语法，使您的应用程序更易于阅读和维护，从而帮助您提高生产力！

在本章中，您将学习如何完成以下任务：

+   配置数据源

+   处理事务

+   管理数据库模式迁移

+   利用 Panache API

+   与 NoSQL 数据存储交互

# 7.1 定义数据源

## 问题

您希望定义和使用数据源。

## 解决方案

使用 Agroal 扩展和 *application.properties*。

## 讨论

Agroal 是 Quarkus 中首选的数据源和连接池实现。Agroal 扩展与安全性、事务管理和健康指标集成。虽然它有自己的扩展，但如果您使用 Hibernate ORM 或 Panache，则 Agroal 扩展会通过传递方式引入。您还需要一个数据库驱动扩展。目前，H2、PostgreSQL、MariaDB、MySQL、Microsoft SQL Server 和 Derby 都有支持的扩展。您可以使用 Maven 的 `add-extension` 添加正确的数据库驱动：

```java
./mvnw quarkus:add-extension -Dextensions="jdbc-mariadb"
```

数据源的配置，就像 Quarkus 的所有其他配置一样，在 *src/main/resources/application.properties* 文件中完成：

```java
quarkus.datasource.url=jdbc::mariadb://localhost:3306/test
quarkus.datasource.driver=org.mariadb.jdbc.Driver
quarkus.datasource.username=username-default
quarkus.datasource.min-size=3
quarkus.datasource.max-size=13
```

###### 提示

敏感数据可以通过系统属性、环境属性、Kubernetes Secrets 或 Vault 传递，稍后章节将详细介绍。

如果您需要访问数据源，可以按以下方式注入它：

```java
@Inject
DataSource defaultDataSource;
```

###### 注意

您还可以使用 `AgroalDataSource` 类型，它是 `DataSource` 的子类型。

# 7.2 使用多个数据源

## 问题

当您的应用程序需要多个数据源时，您可以使用多个数据源。

## 解决方案

使用命名数据源。

Agroal 允许多个数据源。它们的配置与默认配置完全相同，只有一个显著的例外——名称：

```java
quarkus.datasource.driver=org.h2.Driver
quarkus.datasource.url=jdbc:h2:tcp://localhost/mem:default
quarkus.datasource.username=username-default
quarkus.datasource.min-size=3
quarkus.datasource.max-size=13

quarkus.datasource.users.driver=org.h2.Driver
quarkus.datasource.users.url=jdbc:h2:tcp://localhost/mem:users
quarkus.datasource.users.username=username1
quarkus.datasource.users.min-size=1
quarkus.datasource.users.max-size=11

quarkus.datasource.inventory.driver=org.h2.Driver
quarkus.datasource.inventory.url=jdbc:h2:tcp://localhost/mem:inventory
quarkus.datasource.inventory.username=username2
quarkus.datasource.inventory.min-size=2
quarkus.datasource.inventory.max-size=12
```

格式如下：`quarkus.*datasource*.[*optional name*.][*datasource property*]`。

## 讨论

注入的工作方式相同；但是，您需要一个限定符（请参阅 Recipe 5.10 获取有关限定符的更多信息）：

```java
@Inject
AgroalDataSource defaultDataSource;

@Inject
@DataSource("users")
AgroalDataSource dataSource1;

@Inject
@DataSource("inventory")
AgroalDataSource dataSource2;
```

# 7.3 添加数据源健康检查

## 问题

您希望为数据源添加健康检查条目。

## 解决方案

同时使用 `quarkus-agroal` 和 `quarkus-smallrye-health` 扩展。

## 讨论

当使用`quarkus-smallrye-health`扩展时，数据源的健康检查会自动添加。如果需要，可以通过`application.properties`中的`quarkus.datasource.health.enabled`属性（默认为`true`）禁用。要查看状态，请访问应用程序的`/health/ready`端点。该端点是从`quarkus-smallrye-health`扩展创建的。

## 参见

欲了解更多信息，请访问 GitHub 上的以下页面：

+   [MicroProfile Health](https://oreil.ly/CDLOd)

# 7.4 声明事务边界

## 问题

您希望使用注解定义事务边界。

## 解决方案

使用`quarkus-narayana-jta`扩展中的`@javax.transaction.Transactional`注解。

## 讨论

`quarkus-narayana-jta`扩展添加了`@javax.transaction.Transactional`注解，以及`TransactionManager`和`UserTransaction`类。该扩展会自动添加到任何持久性扩展中。当然，如果需要，也可以手动添加。

`@Transactional`注解可以添加到任何 CDI bean 的方法或类级别，使这些方法具有事务性，这也包括 REST 端点：

```java
package org.acme.transaction;

import javax.inject.Inject;
import javax.transaction.Transactional;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

@Path("/tx")
@Transactional
public class Transact {
}
```

# 7.5 设置事务上下文

## 问题

您需要一个不同的事务上下文。

## 解决方案

`@Transactional`的`value`属性允许设置事务的范围。

## 讨论

指定的事务上下文将传播到已注释的方法内的所有嵌套调用。除非在堆栈中抛出运行时异常，否则事务将在方法调用结束时提交：

```java
package org.acme.transaction;

import javax.inject.Inject;
import javax.transaction.Transactional;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

@Path("/tx")
@Transactional
public class Transact {
    @Inject
    NoTransact noTx;

    @GET
    @Path("/no")
    @Produces(MediaType.TEXT_PLAIN)
    public String hi() {
        return noTx.word();
    }
}
```

```java
package org.acme.transaction;

import javax.enterprise.context.ApplicationScoped;
import javax.transaction.Transactional;

import static javax.transaction.Transactional.TxType.NEVER;

@ApplicationScoped
public class NoTransact {
    @Transactional(NEVER)
    public String word() {
        return "Hi";
    }
}
```

这可以通过`@⁠T⁠r⁠a⁠n⁠s​a⁠c⁠t⁠i⁠o⁠n⁠a⁠l`的`dontRollbackOn`或`rollbackOn`属性进行重写。如果需要手动回滚事务，还可以注入`TransactionManager`。

这是可用事务上下文的列表：

`@Transactional(REQUIRED)`（默认）

如果没有启动事务，则启动一个；否则，继续使用现有的事务。

`@Transactional(REQUIRES_NEW)`

如果没有启动事务，则启动一个；如果已经启动了现有事务，则将其挂起，并在该方法的边界内启动一个新事务。

`@Transactional(MANDATORY)`

如果没有启动事务，则失败；否则，在现有事务中工作。

`@Transactional(SUPPORTS)`

如果已经启动了事务，则加入它；否则，在没有事务的情况下工作。

`@Transactional(NOT_SUPPORTED)`

如果已经启动了事务，则暂停它，并在方法边界内没有事务地工作；否则，在没有事务的情况下工作。

`@Transactional(NEVER)`

如果已经启动了事务，则会引发异常；否则，会在没有事务的情况下工作。

# 7.6 编程式事务控制

## 问题

您希望更精细地控制事务。

## 解决方案

注入`UserTransaction`并使用该类的方法。

`UserTransaction`类具有非常简单的 API：

+   `begin()`

+   `commit()`

+   `rollback()`

+   `setRollbackOnly()`

+   `getStatus()`

+   `setTransactionTimeout(int)`

前三种方法将是主要使用的方法。`getStatus()`有助于确定事务在其生命周期中的位置。最后，您可以为事务设置超时时间。

###### 注意

如有必要，您还可以通过注入来使用`javax.transaction.TransactionManager`。

## 参见

欲获取更多信息，请访问以下网页：

+   [Jakarta EE 8 规范 API：接口 UserTransaction](https://oreil.ly/lmjR_)

# 7.7 设置和修改事务超时时间

## 问题

您希望在一定时间后使事务超时并回滚。

## 解决方案

如果使用声明性事务，请使用`@io.quarkus.narayana.jta.runtime.TransactionConfiguration`注解；否则，可以使用事务 API 进行编程式事务控制。您还可以通过`quarkus.transaction-manager.default-transaction-timeout`属性更改全局事务超时时间，该属性以`java.time.Duration`格式指定。

使用`@TransactionConfiguration`注解非常容易修改事务的超时时间。使用`timeout`属性设置超时时间（以秒为单位）。

## 讨论

如果应用程序中的每个事务都需要更长或更短的时间，可以在*application.properties*中使用`quarkus.transaction-manager.default-transaction-timeout`属性。该属性接受`java.time.Duration`，可以作为可通过`Duration#parse()`解析的字符串指定。您也可以从整数开始持续时间。Quarkus 将自动在值前面添加`PT`，以创建正确的格式。

## 参见

欲获取更多信息，请访问以下网站：

+   [Oracle: Java 平台，标准版 8 API 规范：`parse`](https://oreil.ly/8gvMZ)

# 7.8 使用 Persistence.xml 进行设置

## 问题

您希望使用带有*persistence.xml*文件的 JPA。

## 解决方案

您可以像平常一样使用 JPA；只需在*application.properties*中设置数据源即可。

## 讨论

在 Quarkus 中，JPA 的工作方式与其他设置完全相同，因此不需要进行任何更改。

###### 注意

如果使用*persistence.xml*，则不能使用`quarkus.hibernate-orm.*`属性。如果使用*persistence.xml*，则只能使用在*persistence.xml*文件中定义的持久单元。

# 7.9 在没有 persistence.xml 的情况下进行设置

## 问题

您想使用 JPA，但不使用*persistence.xml*文件。

## 解决方案

添加`quarkus-hibernate-orm`扩展，为您的关系型数据库（RDBMS）配置 JDBC 驱动程序，通过*application.properties*进行配置，最后使用`@Entity`注解您的实体。

## 讨论

使用 Quarkus 和 JPA 并没有什么特别的设置，除了数据库连接。Quarkus 将做出一些主观选择，但不用担心——它们可能是您本来会选择的选项——然后继续使用您的实体。您可以像平常一样注入和使用`EntityManager`。

简而言之，您可以像使用标准 JPA 一样继续进行，但不需要额外的 *persistence.xml* 配置。这是在 Quarkus 中使用 JPA 的首选方式。

# 使用来自不同 JAR 的实体

## 问题

您希望包含来自不同 jar 的实体。

## 解决方案

在包含实体的 jar 中包含一个空的 *META-INF/beans.xml* 文件。

## 讨论

Quarkus 依赖于实体的编译时字节码增强。如果这些实体定义在与应用程序的其余部分（jar）相同的项目中，则一切都会正常工作。

然而，如果其他类（如实体或其他 CDI bean）定义在外部库中，则该库必须包含一个空的 *META-INF/beans.xml* 文件，以便正确地被索引和增强。

# 使用 Panache 持久化数据

## 问题

您希望使用 Hibernate 和 Panache 持久化数据。

## 解决方案

在 `PanacheEntity` 上调用 `persist` 方法。

当然，您需要添加 `quarkus-hibernate-orm-panache` 扩展，并为您的数据存储添加相应的 JDBC 扩展。接下来，您需要定义一个实体。所有这些都包括创建一个类，用 `@javax.persistence.Entity` 注解，并扩展自 `PanacheEntity`。

## 讨论

Panache 是建立在传统 JPA 之上的一种主张 API。它更多地遵循一种活动记录的方法来处理数据实体；然而，在底层，它是使用传统的 JPA 来实现的。

正如您在探索 Panache 时将会发现的那样，很多功能都通过 `PanacheEntity` 或 `PanacheEntityBase` 父类传递给您的实体——`persist` 也不例外。`PanacheEntityBase` 包含 `persist()` 和 `persistAndFlush()` 方法。虽然 flush 选项会立即将数据发送到数据库，但这不是推荐的持久化数据的方式。

正如您在以下所见，持久化是非常简单的：

```java
    @POST
    public Response newLibrary(Library library) {
        library.persist();
        return Response.created(URI.create("/library/" + library.encodedName()))
                .entity(library).build();
    }
```

作为补充，这里是 `Library` 实体：

```java
package org.acme.panache;

import java.io.UnsupportedEncodingException;
import java.net.URLEncoder;
import java.util.List;

import javax.persistence.CascadeType;
import javax.persistence.Entity;
import javax.persistence.OneToMany;

import io.quarkus.hibernate.orm.panache.PanacheEntity;
import io.quarkus.panache.common.Parameters;

@Entity
public class Library extends PanacheEntity {
    public String name;

    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true,
               mappedBy = "library")
    public List<Inventory> inventory;
    public String encodedName() {
        String result;

        try {
            result = URLEncoder.encode(name, "UTF-8")
                    .replaceAll("\\+", "%20")
                    .replaceAll("\\%21", "!")
                    .replaceAll("\\%27", "'")
                    .replaceAll("\\%28", "(")
                    .replaceAll("\\%29", ")")
                    .replaceAll("\\%7E", "~");
        } catch (UnsupportedEncodingException e) {
            result = name;
        }

        return result;
    }
}
```

# 使用 Panache 的 listAll 方法来查找所有实体实例

## 问题

您想要找到实体的所有条目。

## 解决方案

使用 `PanacheEntityBase` 类的 `listAll()` 方法。

就像前一节中涵盖的 `persist()` 方法一样，`listAll()` 是 `PanacheEntityBase` 类的一个方法。它并没有什么特别之处；它查询给定实体的所有条目。它以 `List` 的形式返回这些实体：

```java
    @GET
    public List<Book> getAllBooks() {
        return Book.listAll();
    }
```

###### 注意

这实际上是一个 `findAll().list()` 链的快捷方式。有关更多信息，请参阅 `hibernate-orm-panache` 代码库。

# 使用 Panache 的 findById 方法来查找单个实体对象

## 问题

我想要根据其 ID 从数据库中找到并加载一个实体。

## 解决方案

使用 `PanacheEntityBase.findById(Object)`。

使用 `findById(Object)` 方法简化了通过 Panache 查找实体的过程。您只需传递对象的 ID，就能从数据库中返回正确的实例：

```java
    @GET
    @Path("/byId/{id}")
    public Book getBookById(@PathParam(value = "id") Long id) {
        Book b = Book.findById(id);
        return b;
    }
```

# 使用 Panache 查找和列出实体的方法

## 问题

您希望根据其属性查询数据库中的特定实体。

## 解决方案

使用 `PanacheEntityBase` 的各种 `find` 和 `list` 方法的实例。

根据结果返回的需要，您将使用 `PanacheEntityBase` 的 `list` 或 `find`。在内部，`list` 使用 `find`，因此它们本质上是相同的：

```java
    public static Book findByTitle(String title) {
        return find("title", title).firstResult();
    }

    public static List<Book> findByAuthor(String author) {
        return list("author", author);
    }

    public static List<Book> findByIsbn(String isbn) {
        return list("isbn", isbn);
    }
```

两种方法都有多个重载版本——它们根据排序的必要性和您希望发送参数的方式而变化。以下代码是使用 Hibernate 查询语言（HQL）（或 Java 持久化查询语言 [JPQL]）和使用 `Parameters` 类的示例：

```java
    public static Library findByName(String name) {
        return Library
                .find("SELECT l FROM Library l " +
                      "LEFT JOIN fetch l.inventory " +
                      "WHERE l.name = :name ",
                        Parameters.with("name", name)).firstResult();
    }
```

`Parameters` 类的覆盖可用于 `find` 和 `list` 方法。有关更多信息，请参考 API。

###### 注意

您可能会问，为什么需要完整的 JPQL 查询？简而言之，这是为了避免与列表的序列化问题。通过左连接，我们能够获取图书馆及其所有库存。

# 7.15 使用 Panache 的 count 方法获取实体计数

## 问题

您想要获取资源的项目计数。

## 解决方案

使用 `PanacheEntityBase` 的各种 `count` 方法。

就像之前讨论的 `find` 方法一样，Panache 提供了各种 `count` 方法的重载，用于获取数据库中给定类型的实体数量：

```java
Book.count()
Book.count("WHERE title = ?", )
```

# 7.16 使用 Panache 的 page 方法分页浏览实体列表

## 问题

您希望使用分页。

## 解决方案

Quarkus，特别是 Panache，内置了分页功能。`PanacheQuery` 对象上有许多支持分页的方法。

## 讨论

分页非常容易上手。第一步是获取 `PanacheQuery` 的实例。这与使用 `find` 方法一样简单：

```java
PanacheQuery<Book> authors = Book.find("author", author);
authors.page(Page.of(3, 25)).list();                        ![1](img/1.png)

authors.page(Page.sizeOf(10)).list();
```

![1](img/#co_persistence_CO1-1)

每页显示 25 个项目，从第 3 页开始

当然，还有其他方法，如 `firstPage()`、`lastPage()`、`nextPage()` 和 `previousPage()`。还存在支持布尔的方法：`hasNextPage()`、`hasPreviousPage()` 和 `pageCount()`。

## 参见

欲了解更多信息，请参阅 GitHub 上的以下页面：

+   [`Page` 对象](https://oreil.ly/KYlsJ)

+   [`PanacheQuery` 接口](https://oreil.ly/BtgHK)

# 7.17 通过 Panache 的 Stream 方法流式处理结果

## 问题

您希望使用数据流。

## 解决方案

在使用 Panache 时，所有的 `list` 方法都有对应的 `stream` 方法。下面您将看到它们的使用方式，与 `list` 方法的使用方式没有任何不同：

```java
Book.streamAll();
...
Book.stream("author", "Alex Soto");
```

`stream` 和 `streamAll` 方法各自返回一个 `java.util.Stream` 实例。

###### 注意

`stream` 方法需要事务才能正常工作。

## 参见

欲了解更多信息，请访问以下网站：

+   [io.quarkus：`PanacheEntityBase`](https://oreil.ly/OW2fV)

# 7.18 测试 Panache 实体

## 问题

您希望在测试中使用嵌入式数据库。

## 解决方案

Quarkus 提供了用于内存数据库 H2 和 Derby 的助手，以正确地将数据库作为单独的进程引导。

确保将`io.quarkus:quarkus-test-h2:1.4.1.Final`或`i⁠o​.⁠q⁠u⁠a⁠r⁠k⁠u⁠s⁠:⁠q⁠u⁠a⁠r⁠k⁠u⁠s⁠-⁠t⁠e⁠s⁠t⁠-⁠d⁠e⁠r⁠b⁠y⁠:⁠1⁠.⁠4⁠.⁠1⁠.⁠F⁠i⁠n⁠a⁠l`这些构件添加到您的构建文件中。

下一步是对使用嵌入式数据库的任何测试进行注释，使用`@⁠Q⁠u⁠a⁠r⁠k⁠u⁠s​T⁠e⁠s⁠t⁠R⁠e⁠s⁠o⁠u⁠r⁠c⁠e⁠(⁠H⁠2⁠D⁠a⁠t⁠a⁠b⁠a⁠s⁠e⁠T⁠e⁠s⁠t⁠R⁠e⁠s⁠o⁠u⁠r⁠c⁠e⁠.⁠c⁠l⁠a⁠s⁠s⁠)`或`@QuarkusTestResource(DerbyDatabaseTestResource.class)`。最后，请确保在*src/test/resources/META-INF/application.properties*中为所选数据库设置正确的数据库 URL 和驱动程序。

以下是 H2 的示例：

```java
package my.app.integrationtests.db;

import io.quarkus.test.common.QuarkusTestResource;
import io.quarkus.test.h2.H2DatabaseTestResource;

@QuarkusTestResource(H2DatabaseTestResource.class)
public class TestResources {
}
```

```java
quarkus.datasource.url=jdbc:h2:tcp://localhost/mem:test
quarkus.datasource.driver=org.h2.Driver
```

###### 注意

此辅助程序仅将数据库添加到本机映像中，而不是客户端代码。但是，在 JVM 模式或本机映像模式下的应用程序测试中，可以自由使用此辅助程序。

## 参见

欲了解更多信息，请访问以下网站：

+   [H2 Database Engine](https://oreil.ly/_MMus)

+   [Apache Derby](https://oreil.ly/FUFeH)

# 7.19 使用数据访问对象（DAO）或存储库模式

## 问题

欲使用 DAO 或存储库模式。

## 解决方案

Quarkus 不限于 Panache；您可以像以前描述的那样使用实体模式，DAO 或存储库模式。

要使用存储库，您需要理解`PanacheRepository`和`PanacheRepositoryBase`这两个接口。只有在您的主键不是`Long`时，才需要基本接口。`PanacheEntity`上可用的所有操作在`PanacheRepository`上也都可以使用。存储库是一个 CDI bean，在使用时必须注入它。以下是一些基本示例：

```java
package org.acme.panache;

import java.util.Collections;

import javax.enterprise.context.ApplicationScoped;

import io.quarkus.hibernate.orm.panache.PanacheQuery;
import io.quarkus.hibernate.orm.panache.PanacheRepository;
import io.quarkus.panache.common.Parameters;
import io.quarkus.panache.common.Sort;

@ApplicationScoped
public class LibraryRepository implements PanacheRepository<Library> {
    public Library findByName(String name) {
        return find("SELECT l FROM Library l " +
                    "left join fetch l.inventory where l.name = :name ",
                Parameters.with("name", name)).firstResult();
    }

    @Override
    public PanacheQuery<Library> findAll() {
        return find("from Library l left join fetch l.inventory");
    }

    @Override
    public PanacheQuery<Library> findAll(Sort sort) {
        return find("from Library l left join fetch l.inventory",
                sort, Collections.emptyMap());
    }
}
```

DAO 将按您的预期工作。您需要注入一个`EntityManager`并像平常一样查询。关于在 Java 中使用 DAO 的解决方案和示例非常多，可以在线或其他书籍中找到。所有这些示例在 Quarkus 中都将正常运行。

## 参见

欲了解更多信息，请访问以下网站：

+   [io.quarkus: `PanacheRepositoryBase`](https://oreil.ly/H6kTU)

# 7.20 使用 Amazon DynamoDB

## 问题

欲在 Quarkus 应用程序中使用 DynamoDB。

## 解决方案

使用 DynamoDB 扩展并设置配置。DynamoDB 扩展允许同步和异步客户端使用 Apache Amazon Web Service 软件开发工具包（AWS SDK）客户端。在项目中启用并配置此功能需要一些必要的设置。首先当然是依赖项：

```java
 <dependency>
     <groupId>software.amazon.awssdk</groupId>
     <artifactId>url-connection-client</artifactId>
 </dependency>
```

###### 注意

Quarkus 没有 AWS 连接客户端的扩展。

默认情况下，扩展使用 URLConnection HTTP 客户端。您需要向构建脚本添加正确的客户端（URLConnection，Apache 或 Netty NIO）：

```java
 <dependency>
     <groupId>software.amazon.awssdk</groupId>
     <artifactId>apache-client</artifactId>
     <exclusions>    ![1](img/1.png)
         <exclusion>
             <groupId>commons-logging</groupId>
             <artifactId>commons-logging</artifactId>
         </exclusion>
     </exclusions>
 </dependency>
```

![1](img/#co_persistence_CO2-1)

您必须排除`commons-logging`以强制客户端使用 Quarkus 日志记录器

如果使用 Apache 客户端，则还需要对*application.properties*文件进行调整，因为`url`是默认值：

```java
 quarkus.dynamodb.sync-client.type=apache
```

在 *application.properties* 中还有客户端的配置（请参阅[属性引用](https://oreil.ly/HZ4A-)以获取更多信息）：

```java
quarkus.dynamodb.endpoint-override=http://localhost:8000 ![1](img/1.png)
quarkus.dynamodb.aws.region=eu-central-1 ![2](img/2.png)
quarkus.dynamodb.aws.credentials.type=static ![3](img/3.png)
quarkus.dynamodb.aws.credentials.static-provider.access-key-id=test-key
quarkus.dynamodb.aws.credentials.static-provider.secret-access-key=test-secret
```

![1](img/#co_persistence_CO3-1)

如果使用非标准端点（例如本地 DynamoDB 实例），则很有用

![2](img/#co_persistence_CO3-2)

正确且有效的地区

![3](img/#co_persistence_CO3-3)

`static` 或 `default`

`default` 凭证类型将按顺序查找凭证：

+   系统属性 `aws.accessKeyId` 和 `aws.secretKey`

+   环境变量 `AWS_ACCESS_KEY_ID` 和 `AWS_SECRET_ACCESS_KEY`

+   凭证配置文件位于默认位置（*$HOME/.aws/credentials*）

+   通过 Amazon EC2 容器服务提供的凭据

+   通过 Amazon EC2 元数据服务传递的实例配置凭据

## 讨论

下面的示例来自 Recipe 7.11，但使用 DynamoDB 作为持久存储。

下面是用于与 DynamoDB 通信并创建可注入服务以在 REST 端点中使用的两个类：

```java
package org.acme.dynamodb;

import java.util.List;
import java.util.stream.Collectors;

import javax.enterprise.context.ApplicationScoped;
import javax.inject.Inject;

import software.amazon.awssdk.services.dynamodb.DynamoDbClient;

@ApplicationScoped
public class BookSyncService extends AbstractBookService {
    @Inject
    DynamoDbClient dynamoDbClient;

    public List<Book> findAll() {
        return dynamoDbClient.scanPaginator(scanRequest()).items().stream()
                .map(Book::from)
                .collect(Collectors.toList());
    }

    public List<Book> add(Book b) {
        dynamoDbClient.putItem(putRequest(b));
        return findAll();
    }

    public Book get(String isbn) {
        return Book.from(dynamoDbClient.getItem(getRequest(isbn)).item());
    }
}
```

下面的抽象类包含了与 DynamoDB 通信以及持久化和查询 `Book` 实例所需的样板代码：

```java
package org.acme.dynamodb;

import java.util.HashMap;
import java.util.Map;

import software.amazon.awssdk.services.dynamodb.model.AttributeValue;
import software.amazon.awssdk.services.dynamodb.model.GetItemRequest;
import software.amazon.awssdk.services.dynamodb.model.PutItemRequest;
import software.amazon.awssdk.services.dynamodb.model.ScanRequest;

public abstract class AbstractBookService {

    public final static String BOOK_TITLE = "title";
    public final static String BOOK_ISBN = "isbn";
    public final static String BOOK_AUTHOR = "author";

    public String getTableName() {
        return "QuarkusBook";
    }

    protected ScanRequest scanRequest() {
        return ScanRequest.builder().tableName(getTableName()).build();
    }

    protected PutItemRequest putRequest(Book book) {
        Map<String, AttributeValue> item = new HashMap<>();
        item.put(BOOK_ISBN, AttributeValue.builder()
                            .s(book.getIsbn()).build());
        item.put(BOOK_AUTHOR, AttributeValue.builder()
                              .s(book.getAuthor()).build());
        item.put(BOOK_TITLE, AttributeValue.builder()
                             .s(book.getTitle()).build());

        return PutItemRequest.builder()
                .tableName(getTableName())
                .item(item)
                .build();
    }

    protected GetItemRequest getRequest(String isbn) {
        Map<String, AttributeValue> key = new HashMap<>();
        key.put(BOOK_ISBN, AttributeValue.builder().s(isbn).build());

        return GetItemRequest.builder()
                .tableName(getTableName())
                .key(key)
                .build();
    }
}
```

这个最后的类是表示 `Book` 实体的类：

```java
package org.acme.dynamodb;

import java.util.Map;
import java.util.Objects;

import io.quarkus.runtime.annotations.RegisterForReflection;
import software.amazon.awssdk.services.dynamodb.model.AttributeValue;

@RegisterForReflection      ![1](img/1.png)
public class Book {
    private String isbn;
    private String author;
    private String title;

    public Book() {         ![2](img/2.png)
    }

    public static Book from(Map<String, AttributeValue> item) {
        Book b = new Book();
        if (item != null && !item.isEmpty()) {
            b.setAuthor(item.get(AbstractBookService.BOOK_AUTHOR).s());
            b.setIsbn(item.get(AbstractBookService.BOOK_ISBN).s());
            b.setTitle(item.get(AbstractBookService.BOOK_TITLE).s());
        }
        return b;
    }

    public String getIsbn() {
        return isbn;
    }

    public void setIsbn(String isbn) {
        this.isbn = isbn;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Book book = (Book) o;
        return Objects.equals(isbn, book.isbn) &&
                Objects.equals(author, book.author) &&
                Objects.equals(title, book.title);
    }

    @Override
    public int hashCode() {
        return Objects.hash(isbn, author, title);
    }
}
```

![1](img/#co_persistence_CO4-1)

在本机应用程序中必须具有反射

![2](img/#co_persistence_CO4-2)

DynamoDB 客户端所需

大部分是标准的 DynamoDB 代码，唯一的例外是 Quarkus 注解为反射注册 `Book` 类，仅在创建本机镜像时才需要。

正如您所看到的，当使用 Quarkus 时，您以前使用 DynamoDB 时已经掌握的技能仍然可以在工作中使用，而无需进行太多修改，这有助于提高您的工作效率。

# 7.21 使用 MongoDB

## 问题

您希望使用 MongoDB 作为持久存储。

## 解决方案

Quarkus MongoDB 扩展使用了 MongoDB 驱动程序和客户端。

## 讨论

到目前为止，您应该对 RESTful 资源和 Quarkus 配置的基础知识很熟悉。在这里，我们将展示用于与本地 MongoDB 实例通信的代码和示例配置。

自然地，您需要将连接信息添加到您的应用程序中：

```java
quarkus.mongodb.connection-string = mongodb://localhost:27017
```

`Book` 类是 MongoDB 中文档的表示：

```java
package org.acme.mongodb;

import java.util.HashSet;
import java.util.Objects;
import java.util.Set;

import org.bson.Document;

public class Book {
    public String id;
    public String title;
    public String isbn;
    public Set<String> authors;

    // Needed for JSON-B
    public Book() {}

    public Book(String title) {
        this.title = title;
    }

    public Book(String title, String isbn) {
        this.title = title;
        this.isbn = isbn;
    }

    public Book(String title, String isbn, Set<String> authors) {
        this.title = title;
        this.isbn = isbn;
        this.authors = authors;
    }

    public Book(String id, String title, String isbn, Set<String> authors) {
        this.id = id;
        this.title = title;
        this.isbn = isbn;
        this.authors = authors;
    }

    public static Book from(Document doc) {
        return new Book(doc.getString("id"),
                        doc.getString("title"),
                        doc.getString("isbn"),
                        new HashSet<>(doc.getList("authors", String.class)));
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Book book = (Book) o;
        return Objects.equals(id, book.id) &&
                Objects.equals(title, book.title) &&
                Objects.equals(isbn, book.isbn) &&
                Objects.equals(authors, book.authors);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, title, isbn, authors);
    }
}
```

这个服务类充当 DAO，一种进入 MongoDB 实例的方式：

```java
package org.acme.mongodb;

import java.util.ArrayList;
import java.util.List;
import java.util.Objects;

import javax.enterprise.context.ApplicationScoped;
import javax.inject.Inject;

import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoCursor;
import com.mongodb.client.model.Filters;
import org.bson.Document;

@ApplicationScoped
public class BookService {

    @Inject
    MongoClient mongoClient;

    public List<Book> list() {
        List<Book> list = new ArrayList<>();

        try (MongoCursor<Document> cursor = getCollection()
                                            .find()
                                            .iterator()) {
            cursor.forEachRemaining(doc -> list.add(Book.from(doc)));
        }

        return list;
    }

    public Book findSingle(String isbn) {
        Document document = Objects.requireNonNull(getCollection()
                .find(Filters.eq("isbn", isbn))
                .limit(1).first());
        return Book.from(document);
    }

    public void add(Book b) {
        Document doc = new Document()
                .append("isbn", b.isbn)
                .append("title", b.title)
                .append("authors", b.authors);
        getCollection().insertOne(doc);
    }

    private MongoCollection<Document> getCollection() {
        return mongoClient.getDatabase("book").getCollection("book");
    }
}
```

最后，RESTful 资源利用了前面两个类：

```java
package org.acme.mongodb;

import java.util.List;

import javax.inject.Inject;
import javax.ws.rs.Consumes;
import javax.ws.rs.GET;
import javax.ws.rs.POST;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;

@Path("/book")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class BookResource {
    @Inject
    BookService service;

    @GET
    public List<Book> getAll() {
        return service.list();
    }

    @GET
    @Path("{isbn}")
    public Book getSingle(@PathParam("isbn") String isbn) {
        return service.findSingle(isbn);
    }

    @POST
    public Response add(Book b) {
        service.add(b);
        return Response.status(Response.Status.CREATED)
                .entity(service.list()).build();
    }
}
```

我们将其留给读者作为一个练习，来创建并使用 BSON 编解码器。MongoDB 扩展的另一个有用功能是，在使用 `quarkus-smallrye-health` 扩展时自动运行的自动健康检查。当然，可读性检查是可配置的。

Quarkus MongoDB 扩展还包括一个响应式客户端，将在 Recipe 15.12 中详细介绍。

# 7.22 使用 Panache 与 MongoDB

## 问题

您希望使用 Panache 与 MongoDB。

## 解决方案

添加`mongodb-panache`扩展，并使用`PanacheMongoEntity`的所有 Panache 能力。

MongoDB 的 Panache 与 Hibernate 的 Panache 工作方式相同，我们在 7.7 到 7.17 的示例中看到了。它显著简化了您的实体代码：

```java
package org.acme.mongodb.panache;

import java.time.LocalDate;
import java.util.List;

import io.quarkus.mongodb.panache.MongoEntity;
import io.quarkus.mongodb.panache.PanacheMongoEntity;
import org.bson.codecs.pojo.annotations.BsonProperty;

@MongoEntity(collection = "book", database = "book")    ![1](img/1.png)
public class Book extends PanacheMongoEntity {          ![2](img/2.png)
    public String title;
    public String isbn;
    public List<String> authors;

    @BsonProperty("pubDate")                            ![3](img/3.png)
    public LocalDate publishDate;

    public static Book findByIsbn(String isbn) {
        return find("isbn", isbn).firstResult(); ![4](img/4.png)
    }

    public static List<Book> findPublishedOn(LocalDate date) {
        return list("pubDate", date);
    }

}
```

![1](img/#co_persistence_CO5-1)

可选的`@MongoEntity`注解允许您自定义使用的数据库和/或集合。

![2](img/#co_persistence_CO5-2)

必需部分——将您的字段添加为公共字段

![3](img/#co_persistence_CO5-3)

使用`@BsonProperty`自定义序列化字段名。

![4](img/#co_persistence_CO5-4)

使用 PanacheQL（JPQL 的子集）进行查询，就像使用 JPA 一样。

## 讨论

Panache MongoDB 扩展使用`PojoCodecProvider`将实体映射到 MongoDB 的`Document`。除了`@BsonProperty`，您还可以使用`@BsonIgnore`忽略字段。您还可以使用`@BsonId`设置自定义 ID，并扩展`PanacheMongoEntityBase`。

当然，如果您需要编写访问器方法，Panache 并不会阻止您这样做；事实上，在构建时所有字段调用都会替换为相应的访问器/变异器调用。就像 Hibernate 的 Panache 一样，MongoDB 版本支持分页、排序、流和 Panache API 的其余部分。

您在前面的示例中看到的 PanacheQL 查询易于使用和理解；但如果您更喜欢使用常规的 MongoDB 查询，也是支持的，只要查询以`{`开头。

与 Hibernate 和 MongoDB Panache 变体之间的轻微差异在于 MongoDB 能够在`find()`方法的返回上使用查询投影。这允许您限制从数据库返回哪些字段。以下是我们的`Book`实体的一个非常基本的示例：

```java
import io.quarkus.mongodb.panache.ProjectionFor;

@ProjectionFor(Book.class)
public class BookTitle {
    public String title;
}

PanacheQuery<BookTitle> query = Book.find("isbn", "978-1-492-06265-3")
                                    .project(BookTitle.class);
```

如果您具有投影类的层次结构，则父类（们）也需要使用`@ProjectionFor`进行注释。

# 7.23 使用 Neo4j 与 Quarkus

## 问题

您希望连接并使用 Neo4j。

## 解决方案

使用基于 Neo4j Java Driver 的 Quarkus Neo4j 扩展。

以下示例使用基于 JDK 的 CompletableFuture 的异步编程模型。驱动程序还使用类似于 JDBC 的阻塞模型和响应式模型。响应式模型仅适用于 Neo4j 4 及以上版本。

到目前为止，您已经看到如何向项目添加额外的扩展，因此我们这里不再详述。以下示例还管理书籍，就像之前的示例一样：

```java
package org.acme.neo4j;

import java.util.HashSet;
import java.util.Set;
import java.util.StringJoiner;

import org.neo4j.driver.Values;
import org.neo4j.driver.types.Node;

public class Book {
  public Long id;
  public String title;
  public String isbn;
  public Set<String> authors;

  // Needed for JSON-B
  public Book() {}

  public Book(String title) {
    this.title = title;
  }

  public Book(String title, String isbn) {
    this.title = title;
    this.isbn = isbn;
  }

  public Book(String title, String isbn, Set<String> authors) {
    this.title = title;
    this.isbn = isbn;
    this.authors = authors;
  }

  public Book(Long id, String title, String isbn, Set<String> authors) {
    this.id = id;
    this.title = title;
    this.isbn = isbn;
    this.authors = authors;
  }

  public static Book from(Node node) {
    return new Book(node.id(),
        node.get("title").asString(),
        node.get("isbn").asString(),
        new HashSet<>(
          node.get("authors")
          .asList(Values.ofString())
          )
        );
  }

  public String toJson() {
    final StringJoiner authorString =
      new StringJoiner("\",\"", "[\"", "\"]");

    authors.forEach(authorString::add);

    return "{" +
      "\"title\":\"" + this.title + "\"," +
      "\"isbn\":\"" + this.isbn + "\"," +
      "\"authors\":" + authorString.toString() +
      "}";
  }
}
```

当然，您需要配置客户端。这可以很简单，设置`quarkus.neo4j.uri`、`quarkus.neo4j.authentication.username`和`quarkus.neo4j.authentication.password`属性。您可以查阅扩展以获取更多属性信息。

配置客户端的第一件事是 Neo4j 驱动程序。该扩展提供了一个可注入的实例：

```java
@Inject
Driver driver;
```

接下来，创建一个新的 REST 资源并添加 Driver 注入点，然后添加基本的 CRUD 操作：

```java
@GET
public CompletionStage<Response> getAll() {
    AsyncSession session = driver.asyncSession();   ![1](img/1.png)

    return session
            .runAsync("MATCH (b:Book) RETURN b ORDER BY b.title")   ![2](img/2.png)
            .thenCompose(cursor -> cursor.listAsync(record ->
                    Book.from(record.get("b").asNode()))) ![3](img/3.png)
            .thenCompose(books -> session.
                    closeAsync().thenApply(signal -> books))  ![4](img/4.png)
            .thenApply(Response::ok)    ![5](img/5.png)
            .thenApply(Response.ResponseBuilder::build);
}
```

![1](img/#co_persistence_CO6-1)

从驱动程序获取`AsyncSession`

![2](img/#co_persistence_CO6-2)

执行 Cypher（Neo4j 的查询语言）来获取数据

![3](img/#co_persistence_CO6-3)

检索游标，从节点创建`Book`实例

![4](img/#co_persistence_CO6-4)

在完成后关闭会话

![5](img/#co_persistence_CO6-5)

构建 JAX-RS 响应

类/代码的其余部分遵循相同的模式：

```java
@POST
public CompletionStage<Response> create(Book b) {
    AsyncSession session = driver.asyncSession();
    return session
            .writeTransactionAsync(tx ->
                    {
                        String query = "CREATE (b:Book " +
                        "{title: $title, isbn: $isbn, authors: $authors})" +
                        " RETURN b";
                        return tx.runAsync(query,
                                Values.parameters("title", b.title,
                                        "isbn", b.isbn,
                                        "authors", b.authors))
                                .thenCompose(ResultCursor::singleAsync);
                    }
            )
            .thenApply(record -> Book.from(record.get("b").asNode()))
            .thenCompose(persistedBook -> session.closeAsync()
                    .thenApply(signal -> persistedBook))
            .thenApply(persistedBook -> Response.created(
                    URI.create("/book/" + persistedBook.id)).build());
}
```

```java
@DELETE
@Path("{id}")
public CompletionStage<Response> delete(@PathParam("id") Long id) {
    AsyncSession session = driver.asyncSession();
    return session
            .writeTransactionAsync(tx -> tx
                    .runAsync("MATCH (b:Book) WHERE id(b) = $id DELETE b",
                            Values.parameters("id", id))
                    .thenCompose(ResultCursor::consumeAsync))
            .thenCompose(resp -> session.closeAsync())
            .thenApply(signal -> Response.noContent().build());
}
```

最后一个有点不同，因为它处理错误：

```java
@GET
@Path("{id}")
public CompletionStage<Response> getById(@PathParam("id") Long id) {
    AsyncSession session = driver.asyncSession();
    return session.readTransactionAsync(tx ->
            tx.runAsync("MATCH (b:Book) WHERE id(b) = $id RETURN b",
                                Values.parameters("id", id))
                    .thenCompose(ResultCursor::singleAsync))
            .handle(((record, err) -> {
                if (err != null) {
                    Throwable source = err;
                    if (err instanceof CompletionException)
                        source = ((CompletionException) err).getCause();
                    Response.Status status = Response.Status.
                                                INTERNAL_SERVER_ERROR;
                    if (source instanceof NoSuchRecordException)
                        status = Response.Status.NOT_FOUND;

                    return Response.status(status).build();
                } else {
                    return Response.ok(Book.from(record.get("b")
                                        .asNode())).build();
                }
            }))
            .thenCompose(response -> session.closeAsync()
                                            .thenApply(signal -> response));
}
```

## 参见

[Neo4j Cypher 手册](https://oreil.ly/ITHPx)在您学习和尝试 Cypher 的新功能时会很有帮助。

# 7.24 在启动时使用 Flyway

## 问题

您希望使用 Flyway 来迁移我的数据库架构。

## 解决方案

使用`quarkus-flyway`集成扩展。

## 讨论

Quarkus 对使用 Flyway 进行模式迁移具有一流支持。您需要做五件事来在应用程序启动时使用 Flyway 与 Quarkus：

1.  添加 Flyway 扩展。

1.  添加适用于你的数据库的 JDBC 驱动程序。

1.  设置数据源。

1.  将迁移添加到*src/main/resources/db/migration*。

1.  将`quarkus.flyway.migrate-at-start`设置为`true`。

Flyway 迁移的默认命名模式是`V.<version>__<description>.sql`。其他所有事项都已处理。

您还可以使用多个数据源来使用 Flyway。任何需要为每个数据源配置的设置都以与数据源名称相同的模式命名：`quarkus.flyway.*数据源名称*.*设置*`。例如，可能是`quarkus.flyway.users.migrate-at-start`。

# 7.25 以编程方式使用 Flyway

## 问题

您希望以编程方式使用 Flyway。有时，您可能希望控制模式何时迁移，而不是在应用程序启动时进行迁移。

## 解决方案

使用`quarkus-flyway`扩展并注入`Flyway`实例：

```java
@Inject
Flyway flyway
```

## 讨论

这将注入针对默认数据源配置的默认`org.flywaydb.core.Flyway`实例。如果您有多个数据源和 Flyway 实例，可以使用`@FlywayDataSource`或`@Named`注解来注入特定的实例。当使用`@FlywayDataSource`时，值是数据源的名称。如果改用`@Named`，则值应该是带有`flyway_`前缀的数据源名称：

```java
@Inject
@FlywayDataSource("books")
Flyway flywayBooks;

@Inject
@Named("flyway_users")
Flyway flywayUsers;
```

自然而然，您将能够运行所有标准的 Flyway 操作，如`clean`，`migrate`，`validate`，`info`，`baseline`和`repair`。
