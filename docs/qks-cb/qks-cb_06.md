# 第六章：打包 Quarkus 应用程序

在本章中，您将学习如何将 Quarkus 服务打包成 JVM 或本地格式，以便分发和部署。如今，随着容器成为分发应用程序的标准方式，您需要了解如何将其容器化。

我们将讨论以下主题：

+   如何将 Quarkus 应用程序打包以在 JVM 中运行

+   如何将 Quarkus 应用打包成本地可执行文件

+   如何将 Quarkus 应用程序容器化

# 6.1 在命令模式下运行

## 问题

您想创建一个 CLI 应用程序。

## 解决方案

使用 Quarkus，您还可以编写运行后可选择退出的应用程序。

要在 Quarkus 中启用命令模式，您需要创建一个实现 `io.quarkus.runtime.QuarkusApplication` 接口的类：

```java
package org.acme.quickstart;

import io.quarkus.runtime.Quarkus;
import io.quarkus.runtime.QuarkusApplication;

public class GreetingMain implements QuarkusApplication { ![1](img/1.png)

    @Override
    public int run(String... args) throws Exception { ![2](img/2.png)
        System.out.println("Hello World");
        Quarkus.waitForExit(); ![3](img/3.png)
        return 0;
    }

}
```

![1](img/#co_packaging_quarkus_applications_CO1-1)

设置 Quarkus 以命令模式运行的接口

![2](img/#co_packaging_quarkus_applications_CO1-2)

当调用 `main` 方法时执行的方法

![3](img/#co_packaging_quarkus_applications_CO1-3)

不要退出，而是等待 Quarkus 进程停止

然后，您可以实现众所周知的 Java `main` 方法。其中一个要求是带有 `main` 方法的类必须用 `@io.quarkus.runtime.annotations.QuarkusMain` 注解进行注释：

```java
package org.acme.quickstart;

import io.quarkus.runtime.Quarkus;
import io.quarkus.runtime.annotations.QuarkusMain;

@QuarkusMain ![1](img/1.png)
public class JavaMain {

    public static void main(String... args) {
        Quarkus.run(GreetingMain.class, args); ![2](img/2.png)
    }

}
```

![1](img/#co_packaging_quarkus_applications_CO2-1)

将类设置为 `main`

![2](img/#co_packaging_quarkus_applications_CO2-2)

启动进程

如果你想访问命令行参数，可以使用`@io.quarkus.runtime.annotations.CommandLineArguments`注解来注入它们：

```java
package org.acme.quickstart;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

import io.quarkus.runtime.annotations.CommandLineArguments;

@Path("/hello")
public class GreetingResource {

    @CommandLineArguments ![1](img/1.png)
    String[] args;

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String hello() {
        return args[0];
    }
}
```

![1](img/#co_packaging_quarkus_applications_CO3-1)

注入命令行参数

最后，您可以构建项目并运行它：

```java
./mvnw clean package -DskipTests

java -jar target/greeting-started-cli-1.0-SNAPSHOT-runner.jar Aloha

curl localhost:8080/hello
Aloha
```

## 讨论

可以使用两种不同的方法来实现退出的应用程序。我们在前一节中解释了第一种方法；第二种方法是通过将实现 `io.quarkus.runtime.QuarkusApplication` 接口的类用 `@io.quarkus.runtime.annotations.QuarkusMain` 注解进行注释。

第二种解决方案的缺点是无法从 IDE 中运行，这也是我们建议您使用前一种方法的原因。

正如您在示例中看到的那样，如果您希望在启动时运行一些逻辑，然后像普通应用程序一样运行它（即不退出），则应从主线程调用 `Quarkus.waitForExit` 方法。如果您不调用此方法，则 Quarkus 应用程序将启动然后终止，因此您的应用程序实际上会像任何其他 CLI 程序一样运行。

# 6.2 创建一个可运行的 JAR 文件

## 问题

您想创建一个可分发/容器化到安装了 JVM 的机器上的可运行 JAR 文件。

## 解决方案

使用*Quarkus Maven 插件*创建可运行的 JAR 包。

如果您使用之前提到的任何启动器生成了项目，Quarkus Maven 插件将会默认安装：

```java
<plugin>
  <groupId>io.quarkus</groupId>
  <artifactId>quarkus-maven-plugin</artifactId>
  <version>${quarkus.version}</version>
  <executions>
    <execution>
      <goals>
        <goal>build</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

然后运行 `package` 目标以构建 JAR 文件：

```java
./mvnw clean package
```

`target` 目录包含以下内容：

```java
target ├── classes ├── generated-sources ├── generated-test-sources ├── getting-started-1.0-SNAPSHOT-runner.jar ![1](img/1.png)
├── getting-started-1.0-SNAPSHOT.jar ![2](img/2.png)
├── lib ![3](img/3.png)
├── maven-archiver ├── maven-status ├── test-classes ├── transformed-classes └── wiring-classes
```

![1](img/#co_packaging_quarkus_applications_CO4-1)

一个可执行 JAR（而不是超级 JAR）

![2](img/#co_packaging_quarkus_applications_CO4-2)

依赖项的位置

![3](img/#co_packaging_quarkus_applications_CO4-3)

应用依赖的 Lib 文件夹

如果要部署应用程序，重要的是将 *可执行 JAR* 与 `lib` 目录一起复制。

您可以通过运行以下命令来运行应用程序：

```java
java -jar target/getting-started-1.0-SNAPSHOT-runner.jar
```

以 JVM 模式运行 Quarkus 被称为在 *JVM 模式* 下运行 Quarkus。这意味着您没有进行本地编译，而是在 JVM 内运行应用程序。

###### 提示

如果您想将 JVM 模式下的 Quarkus 应用程序打包到容器中，我们建议使用这种方法，因为在容器构建阶段创建的层将被缓存以供将来重复使用。库通常不会改变，因此这个依赖层可能在未来的执行中被多次重用，加快容器构建时间。

## 讨论

使用 Gradle 创建可运行的 JAR 文件，您可以运行 `quarkusBuild` 任务：

```java
./gradlew quarkusBuild
```

## 参见

如果您想了解如何创建超级 JAR 或如何将 Quarkus 应用程序容器化，请参阅 Recipe 6.3。

# 6.3 超级 JAR 打包

## 问题

您希望创建一个 Quarkus 应用程序的超级 JAR。

## 解决方案

Quarkus Maven 插件通过在 *pom.xml* 中设置 `uberJar` 配置选项来支持生成超级 JAR：

要创建一个包含您的代码 `可运行类` 和所有必需依赖项的超级 JAR，您需要在 *application.properties* 文件中相应地配置 Quarkus，将 `quarkus.package.uber-jar` 设置为 `true`：

```java
quarkus.package.uber-jar=true
```

# 6.4 构建本地可执行文件

## 问题

您希望将 Quarkus 应用程序构建为本地可执行文件。

## 解决方案

使用 Quarkus 和 GraalVM 构建适用于容器和无服务器负载的本地可运行文件。

本地可执行文件使 Quarkus 应用程序非常适合于容器和无服务器工作负载。Quarkus 依赖于 [GraalVM](https://www.graalvm.org) 来将 Java 应用程序构建为本地可执行文件。

在构建原生可执行文件之前，请确保设置了 `GRAALVM_HOME` 环境变量，指向 GraalVM 19.3.1 或 20.0.0 的安装目录。

###### 重要提示

如果您使用 macOS，该变量应指向 *Home* 子目录：`export GRAALVM_HOME=<installation_dir>/Development/graalvm/Contents/Home/`。

当使用之前解释过的任何方法生成 Quarkus 项目时，它会注册一个名为 `native` 的默认 Maven 配置文件，可用于构建 Quarkus 本地可执行应用程序：

```java
<profile>
  <id>native</id>
  <activation>
    <property>
      <name>native</name>
    </property>
  </activation>
  <build>
    <plugins>
      <plugin>
        <artifactId>maven-failsafe-plugin</artifactId>
        <version>${surefire-plugin.version}</version>
        <executions>
          <execution>
            <goals>
              <goal>integration-test</goal>
              <goal>verify</goal>
            </goals>
            <configuration>
              <systemProperties>
                <native.image.path>
                  ${project.build.directory}/${project.build.finalName}-runner
                </native.image.path>
              </systemProperties>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
  <properties>
    <quarkus.package.type>native</quarkus.package.type>
  </properties>
</profile>
```

然后需要使用启用了 `native` 配置文件的项目构建：

```java
./mvnw package -Pnative
```

几分钟后，`target` 目录中将出现一个本地可执行文件：

```java
[INFO] --- quarkus-maven-plugin:1.4.1.Final:native-image (default) @
 getting-started ---
[INFO] [io.quarkus.creator.phase.nativeimage.NativeImagePhase] Running Quarkus
 native-image plugin on Java HotSpot(TM) 64-Bit Server VM
...
[getting-started-1.0-SNAPSHOT-runner:19]    classlist:  13,614.07 ms
[getting-started-1.0-SNAPSHOT-runner:19]        (cap):   2,306.78 ms
[getting-started-1.0-SNAPSHOT-runner:19]        setup:   4,793.43 ms
...
Printing list of used packages to
 /project/reports/used_packages_getting-started-1.0-SNAPSHOT-
 runner_20190927_134032.txt
[getting-started-1.0-SNAPSHOT-runner:19]    (compile):  42,452.12 ms
[getting-started-1.0-SNAPSHOT-runner:19]      compile:  62,356.07 ms
[getting-started-1.0-SNAPSHOT-runner:19]        image:   2,939.16 ms
[getting-started-1.0-SNAPSHOT-runner:19]        write:     696.65 ms
[getting-started-1.0-SNAPSHOT-runner:19]      [total]: 151,743.29 ms

target/getting-started-1.0-SNAPSHOT-runner
```

## 讨论

要在 Gradle 中构建本机可执行文件，可以使用 `buildNative` 任务：

```java
./gradlew buildNative
```

# 6.5 为 JAR 文件构建 Docker 容器

## 问题

您希望构建一个包含在 Recipe 6.2 配方中构建的 JAR 的容器。

## 解决方案

使用提供的 *Dockerfile.jvm* 文件构建容器。

当使用之前解释的任何方法生成 Quarkus 项目时，会在 *src/main/docker* 中创建两个 Dockerfile：一个用于使用 JVM 模式下的 Quarkus 生成 Docker 容器，另一个用于本机可执行文件。

要生成一个用于在 JVM 中运行 Quarkus 的容器（非本机），您可以使用 *Dockerfile.jvm* 文件来构建容器。此 Dockerfile 添加了 *lib* 目录和可运行的 JAR 并暴露了 JMX。

要构建 Docker 镜像，您需要按照 Recipe 6.2 中所示的步骤对项目进行打包，然后构建容器：

```java
./mvnw clean package

docker build -f src/main/docker/Dockerfile.jvm -t example/greetings-app .
```

容器可以通过运行以下命令启动：

```java
docker run -it --rm -p 8080:8080 example/greetings-app
```

# 6.6 为本机文件构建 Docker 容器

## 问题

您希望构建一个本机可执行文件容器映像。

## 解决方案

要生成用于运行 Quarkus 本机可执行文件的容器，可以使用 *Dockerfile.native* 文件来构建容器。

要构建 Docker 镜像，您需要创建一个可以在 Docker 容器中运行的本机文件。因此，请勿使用本地 GraalVM 来构建本机可执行文件，因为结果文件将针对您的操作系统进行特定的处理，将无法在容器中运行。

要在终端中使用以下命令创建可以在容器中运行的可执行文件：

```java
./mvnw clean package -Pnative -Dquarkus.native.container-build=true
```

此命令创建一个包含 GraalVM 安装的 Docker 镜像，用于从您的代码生成 64 位 Linux 可执行文件。

###### 重要

您需要在 *pom.xml* 中定义 `native` 配置文件，如 Recipe 6.4 配方所述。

最后一步是使用在上一步中生成的本机可执行文件构建 Docker 镜像：

```java
docker build -f src/main/docker/Dockerfile.native -t example/greetings-app .
```

然后可以通过运行以下命令来启动容器：

```java
docker run -it --rm -p 8080:8080 example/greetings-app
```

## 讨论

默认情况下，Quarkus 使用 `docker` 来构建容器。可以使用 `quarkus.native.container-runtime` 属性更改容器运行时。在编写本书时，`docker` 和 `podman` 是支持的选项：

```java
./mvnw package -Pnative -Dquarkus.native.container-build=true \
               -Dquarkus.native.container-runtime=podman
```

# 6.7 构建和 Docker 化本机 SSL 应用程序

## 问题

在构建本机可执行文件时，您希望保护连接，以防止攻击者窃取敏感信息。

## 解决方案

在本机可执行文件中启用 Quarkus 使用 SSL 来保护连接。

如果你在 JVM 模式下运行 Quarkus 应用程序，则 SSL 支持没有任何问题，就像任何其他 JVM 应用程序一样。但是在本机可执行文件的情况下，SSL 并不是默认支持的，必须执行一些额外的步骤（特别是在 Docker 化应用程序时）来启用 SSL 支持。

通过在 *application.properties* 中添加 `quarkus.ssl.native` 配置属性来启用 Quarkus 在本机可执行文件中使用 SSL 保护连接：

```java
quarkus.ssl.native=true
```

启用此属性允许 Graal VM 的`native-image`过程启用 SSL。使用以下命令创建本地可执行文件：

```java
./mvnw clean package -Pnative -DskipTests -Dquarkus.native.container-build=true

...

docker run -v \
 gretting-started/target/gretting-started-1.0-SNAPSHOT-native-image-source-jar: \
 /project:z
```

下面是由进程自动添加以启用 SSL 的重要标志：

```java
-H:EnableURLProtocols=http,https --enable-all-security-services -H:+JNI
```

要将此本地可执行文件 Docker 化，需要稍微修改与 SSL 支持相关的 Docker 脚本。

打开`.dockeringnore`并将`keystore.jks`文件添加为不可忽略的文件，以便将其添加到生成的容器中。这是必要的，因为 keystore 文件需要与可执行文件一起复制：

```java
*
!target/classes/keystore.jks
!target/*-runner
!target/*-runner.jar
!target/lib/*
```

*src/main/docker/Dockerfile.native*文件还必须适应以下元素的打包：

+   SunEC 库

+   收集用于验证应用程序中使用的证书的受信任证书颁发机构文件集合。

```java
FROM quay.io/quarkus/ubi-quarkus-native-image:19.2.1 as nativebuilder ![1](img/1.png)
FROM quay.io/quarkus/ubi-quarkus-native-image:19.3.1-java11 as nativebuilder
RUN mkdir -p /tmp/ssl \
  && cp /opt/graalvm/lib/security/cacerts /tmp/ssl/

FROM registry.access.redhat.com/ubi8/ubi-minimal
WORKDIR /work/
COPY --from=nativebuilder /tmp/ssl/ /work/ ![2](img/2.png)
COPY target/*-runner /work/application
RUN chmod 775 /work /work/application \ ![3](img/3.png)
  && chown -R 1001 /work \
  && chmod -R "g+rwX" /work \
  && chown -R 1001:root /work
EXPOSE 8080 8443 ![4](img/4.png)
USER 1001
CMD ["./application",
"-Dquarkus.http.host=0.0.0.0",
"-Djavax.net.ssl.trustStore=/work/cacerts"] ![5](img/5.png)
```

![1](img/#co_packaging_quarkus_applications_CO5-1)

从 GraalVM Docker 镜像获取 SunEC 库和`cacerts`

![2](img/#co_packaging_quarkus_applications_CO5-2)

将自定义的`keystore.jks`复制到根工作目录

![3](img/#co_packaging_quarkus_applications_CO5-3)

设置权限

![4](img/#co_packaging_quarkus_applications_CO5-4)

暴露 HTTPS 端口

![5](img/#co_packaging_quarkus_applications_CO5-5)

在运行应用程序时加载 SunEC 和`cacerts`

可通过运行以下命令构建容器镜像：

```java
docker build -f src/main/docker/Dockerfile.native -t greeting-ssl .
```

## 讨论

安全和 SSL 现在很常见，始终使所有服务使用 SSL 进行通信是一个良好的做法。因此，当注册以下任何扩展时，Quarkus 会自动启用 SSL 支持：

+   Agroal 连接池

+   Amazon DynamoDB

+   Hibernate Search Elasticsearch

+   Infinispan 客户端

+   Jaeger

+   JGit

+   Keycloak

+   Kubernetes 客户端

+   Mailer

+   MongoDB

+   Neo4j

+   OAuth2

+   REST 客户端

只要在项目中有其中一个扩展，`quarkus.native.ssl`属性默认设置为`true`。
