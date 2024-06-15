# 第十章：集成 Kubernetes

到目前为止，您一直在学习如何在裸机上开发和运行 Quarkus 应用程序，但 Quarkus 真正发光的地方是在 Kubernetes 集群中运行时。

在本章中，您将了解 Quarkus 与 Kubernetes 之间的集成，以及如何使用多个扩展来帮助开发和部署 Quarkus 服务到 Kubernetes。

Kubernetes 正在成为部署应用程序的事实标准平台；因此，了解 Kubernetes 及如何在其上正确开发和部署应用程序至关重要。

在本章中，您将学习如何完成以下任务：

+   构建并推送容器镜像。

+   生成 Kubernetes 资源

+   部署一个 Quarkus 服务

+   开发 Kubernetes 运算符。

+   在 Knative 中部署一个服务。

# 10.1 构建和推送容器镜像

## 问题

您希望构建并推送容器镜像。

## 解决方案

Kubernetes 中的工作单元是一个 *pod*。Pod 表示在同一主机上运行并共享 IP 和端口等资源的一组容器。要将服务部署到 Kubernetes，您需要创建一个 pod。由于 pod 由一个或多个容器组成，因此需要构建服务的容器镜像。

Quarkus 提供了用于构建和可选推送容器镜像的扩展。在撰写本文时，支持以下容器构建策略：

Jib

Jib 为您的 Java 应用程序构建 Docker 和 OCI 容器镜像，无需 Docker 守护程序（无 Docker）。这使得在容器内部运行进程时构建 Docker 镜像变得非常完美，因为您避免了 Docker-in-Docker (DinD) 进程的麻烦。此外，使用 Jib 与 Quarkus 将所有依赖项缓存到与实际应用程序不同的层中，使重建速度快且节省空间。这也改善了推送和构建时间。

Docker

使用 Docker 策略构建容器镜像，使用本地安装的 `docker` 二进制文件，默认情况下使用位于 *src/main/docker* 下的 `Dockerfiles` 来构建镜像。

S2I

Source-to-Image (S2I) 策略使用 `s2i` 二进制文件在 OpenShift 集群内执行容器构建。S2I 构建需要创建 `BuildConfig` 和两个 `ImageStream` 资源。这些资源的创建由 Quarkus Kubernetes 扩展来实现。

在本教程中，我们将使用 Jib 构建和推送容器；本教程的“讨论”部分将涉及 Docker 和 S2I。

要使用 Jib 构建并推送容器镜像，首先需要添加 Jib 扩展。

```java
./mvnw quarkus:add-extensions -Dextensions="quarkus-container-image-jib"
```

然后，您可以定制容器镜像构建过程。您可以在 *application.properties*、系统属性或环境变量中设置这些属性，就像在 Quarkus 中设置任何其他配置参数一样：

```java
quarkus.container-image.group=lordofthejars ![1](img/1.png)
quarkus.container-image.registry=quay.io ![2](img/2.png)
quarkus.container-image.username=lordofthejars ![3](img/3.png)
#quarkus.container-image.password= ![4](img/4.png)
```

![1](img/#co_integrating_with_kubernetes_CO1-1)

设置镜像的组部分；默认情况下为 `${user.name}`

![2](img/#co_integrating_with_kubernetes_CO1-2)

镜像推送的注册表；默认情况下，镜像被推送到`docker.io`

![3](img/#co_integrating_with_kubernetes_CO1-3)

登录容器注册表的用户名

![4](img/#co_integrating_with_kubernetes_CO1-4)

登录容器注册表的密码

要为项目构建并推送容器镜像，您需要将`quarkus.container-image.push`参数设置为`true`，并且在`package`阶段期间，将创建并推送容器：

```java
./mvnw clean package -Dquarkus.container-image.push=true 
... [INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ greeting-jib --- [INFO] Building jar: /greeting-jib/target/greeting-jib-1.0-SNAPSHOT.jar [INFO] [INFO] --- quarkus-maven-plugin:1.3.0.CR2:build (default) @ greeting-jib --- [INFO] [org.jboss.threads] JBoss Threads version 3.0.1.Final [INFO] [io.quarkus.deployment.pkg.steps.JarResultBuildStep] Building thin jar:
 greeting-jib/target/greeting-jib-1.0-SNAPSHOT-runner.jar [INFO] [io.quarkus.container.image.jib.deployment.JibProcessor]
 Starting container image build ![1](img/1.png)
[WARNING] [io.quarkus.container.image.jib.deployment.JibProcessor]
 Base image 'fabric8/java-alpine-openjdk8-jre' does not use a specific image digest - build may not be reproducible [INFO] [io.quarkus.container.image.jib.deployment.JibProcessor] LogEvent
 [level=INFO, message=trying docker-credential-desktop for quay.io] [INFO] [io.quarkus.container.image.jib.deployment.JibProcessor] LogEvent
 [level=LIFECYCLE, message=Using credentials from Docker config ($HOME/.docker/config.json) for quay.io/lordofthejars/greeting-jib:1.0-SNAPSHOT] [INFO] [io.quarkus.container.image.jib.deployment.JibProcessor] The base image
 requires auth. Trying again for fabric8/java-alpine-openjdk8-jre... [INFO] [io.quarkus.container.image.jib.deployment.JibProcessor] Using base
 image with digest: sha256:a5d31f17d618032812ae85d12426b112279f02951fa92a7ff8a9d69a6d3411b1 [INFO] [io.quarkus.container.image.jib.deployment.JibProcessor] Container
 entrypoint set to [java, -Dquarkus.http.host=0.0.0.0, -Djava.util.logging.manager=org.jboss.logmanager.LogManager, -cp, /app/resources:/app/classes:/app/libs/*, io.quarkus.runner.GeneratedMain] [INFO] [io.quarkus.container.image.jib.deployment.JibProcessor] Pushed container image quay.io/lordofthejars/greeting-jib:1.0-SNAPSHOT (sha256:e173e0b49bd5ec1f500016f46f2cde03a055f558f72ca8ee1d6cb034a385a657)![2](img/2.png)

[INFO] [io.quarkus.deployment.QuarkusAugmentor] Quarkus augmentation completed
 in 12980ms
```

![1](img/#co_integrating_with_kubernetes_CO2-1)

容器镜像已构建

![2](img/#co_integrating_with_kubernetes_CO2-2)

容器镜像被推送到 quay.io

## 讨论

除了 Jib 之外，还有两个其他选项可用于构建容器镜像；要使用它们，您只需注册该扩展：

Docker

`quarkus-container-image-docker`

S2I

`quarkus-container-image-s2i`

每个扩展都提供特定的配置参数来修改构建过程。这些参数允许您更改用于构建容器镜像的基础镜像，并允许您设置环境变量、传递给可执行文件的参数或 Dockerfile 的位置。

您还可以构建镜像，但不将其推送到注册表。为此，您需要将`quarkus.container-image.build`属性设置为`true`，并且不设置`quarkus.container-image.push`属性：

```java
./mvnw clean package -Dquarkus.container-image.build=true
```

###### 重要提示

如果使用了`Jib`并且将`push`设置为`false`，该扩展将创建一个容器镜像并将其注册到 Docker 守护程序。这意味着虽然不使用 Docker 构建镜像，但它仍然是必要的。

容器镜像扩展可以从*build/output*目录中的 JAR 包（用于 JVM 模式）或本地可执行文件创建容器，具体取决于目录中的内容。如果您希望创建一个可以在 Linux 容器中运行的本地可执行文件，并在其中创建包含生成的本地可执行文件的容器镜像，您可以运行以下命令：

```java
./mvnw clean package -Dquarkus.container-image.push=true -Pnative \
 -Dquarkus.native.container-build=true
```

将`quarkus.native.container-build`属性设置为`true`，可以在 Docker 容器内创建本地可执行文件。

## 参见

欲获取更多信息，请访问 GitHub 上的以下页面：

+   [Google 容器工具：Jib](https://oreil.ly/6vrh5)

+   [源码到镜像（Source-To-Image）](https://oreil.ly/8PEgn)

# 10.2 生成 Kubernetes 资源

## 问题

您希望自动生成 Kubernetes 资源。

## 解决方案

Quarkus 具有一个 Kubernetes 扩展，能够根据合理的默认值和可选的用户提供的配置自动生成 Kubernetes 资源。目前，该扩展可以为 Kubernetes 和 OpenShift 生成资源。

要启用生成 Kubernetes 资源的功能，您需要注册`quarkus-kubernetes`扩展：

```java
./mvnw quarkus:add-extension -Dextensions="quarkus-kubernetes"
```

要生成 Kubernetes 资源，请在新的终端中执行 `./mvnw package`。然后在 `target` 目录中，构建工具通常生成的文件中，会在 *target/kubernetes* 目录下生成两个新文件。这些新文件分别命名为 *kubernetes.json* 和 *kubernetes.yaml*，每个文件都包含 `Deployment` 和 `Service` 的定义：

```java
{
  "apiVersion" : "v1",
  "kind" : "List",
  "items" :  {
    "apiVersion" : "v1",
    "kind" : "Service",
    "metadata" : {
      "labels" : {
        "app" : "getting-started", ![1        "version" : "1.0-SNAPSHOT", ![2](img/2.png)
        "group" : "alex" ![3](img/3.png)
      },
      "name" : "getting-started"
    },
    "spec" : {
      "ports" : [ {
        "name" : "http",
        "port" : 8080,
        "targetPort" : 8080
      } ],
      "selector" : {
        "app" : "getting-started",
        "version" : "1.0-SNAPSHOT",
        "group" : "alex"
      },
      "type" : "ClusterIP"
    }
  }, {
    "apiVersion" : "apps/v1",
    "kind" : "Deployment",
    "metadata" : {
      "labels" : {
        "app" : "getting-started",
        "version" : "1.0-SNAPSHOT",
        "group" : "alex"
      },
      "name" : "getting-started"
    },
    "spec" : {
      "replicas" : 1,
      "selector" : {
        "matchLabels" : {
          "app" : "getting-started",
          "version" : "1.0-SNAPSHOT",
          "group" : "alex"
        }
      },
      "template" : {
        "metadata" : {
          "labels" : {
            "app" : "getting-started",
            "version" : "1.0-SNAPSHOT",
            "group" : "alex"
          }
        },
        "spec" : {
          "containers" : [ {
            "env" : [ {
              "name" : "KUBERNETES_NAMESPACE",
              "valueFrom" : {
                "fieldRef" : {
                  "fieldPath" : "metadata.namespace"
                }
              }
            } ],
            "image" : "alex/getting-started:1.0-SNAPSHOT",
            "imagePullPolicy" : "IfNotPresent",
            "name" : "getting-started",
            "ports" : [ {
              "containerPort" : 8080,
              "name" : "http",
              "protocol" : "TCP"
            } ]
          } ]
        }
      }
    }
  } ]
}
```

![1](img/#co_integrating_with_kubernetes_CO3-1)

默认为项目名称

![2](img/#co_integrating_with_kubernetes_CO3-2)

默认为版本字段

![3](img/#co_integrating_with_kubernetes_CO3-3)

默认为操作系统用户名

## 讨论

您可以通过将这些属性添加到 *application.properties* 中，自定义生成清单中使用的组和名称：

```java
quarkus.container-image.group=redhat
quarkus.application.name=message-app
```

Kubernetes 扩展允许向清单的不同部分提供用户自定义：

```java
quarkus.kubernetes.replicas=3 ![1](img/1.png)

quarkus.container-image.registry=http://my.docker-registry.net ![2](img/2.png)
quarkus.kubernetes.labels.environment=prod ![3](img/3.png)

quarkus.kubernetes.readiness-probe.initial-delay-seconds=10 ![4](img/4.png)
quarkus.kubernetes.readiness-probe.period-seconds=30
```

![1](img/#co_integrating_with_kubernetes_CO4-1)

设置副本数

![2](img/#co_integrating_with_kubernetes_CO4-2)

添加 Docker 注册表以拉取镜像

![3](img/#co_integrating_with_kubernetes_CO4-3)

添加新的标签

![4](img/#co_integrating_with_kubernetes_CO4-4)

设置就绪性探针

您可以通过在 *application.properties* 文件中设置属性 `quarkus.kubernetes.deployment-target` 或作为系统属性来生成不同的资源。

该属性的默认值是 `kubernetes`，但在撰写时还支持以下值：`kubernetes`、`openshift` 和 `knative`。

## 参见

以下网页列出了所有 Kubernetes 配置选项，以修改生成的文件：

+   [Quarkus: Kubernetes 扩展](https://oreil.ly/oLxhT)

# 10.3 使用健康检查生成 Kubernetes 资源

## 问题

您希望自动创建带有存活检查和就绪检查的 Kubernetes 资源。

## 解决方案

默认情况下，输出文件中不会生成健康检查探针，但如果存在 `quarkus-smallrye-health` 扩展（如 配方 9.1 中所述），则会自动生成就绪性和存活性探针部分：

```java
"image" : "alex/getting-started:1.0-SNAPSHOT",
"imagePullPolicy" : "IfNotPresent",
"livenessProbe" : { ![1](img/1.png)
    "failureThreshold" : 3,
    "httpGet" : {
        "path" : "/health/live", ![2](img/2.png)
        "port" : 8080,
        "scheme" : "HTTP"
        },
    "initialDelaySeconds" : 0,
    "periodSeconds" : 30,
    "successThreshold" : 1,
    "timeoutSeconds" : 10
},
"name" : "getting-started",
"ports" : [ {
    "containerPort" : 8080,
    "name" : "http",
    "protocol" : "TCP"
    } ],
"readinessProbe" : { ![3](img/3.png)
    "failureThreshold" : 3,
    "httpGet" : {
        "path" : "/health/ready", ![4](img/4.png)
        "port" : 8080,
        "scheme" : "HTTP"
    },
    "initialDelaySeconds" : 0,
    "periodSeconds" : 30,
    "successThreshold" : 1,
    "timeoutSeconds" : 10
}
```

![1](img/#co_integrating_with_kubernetes_CO5-1)

定义存活性探针

![2](img/#co_integrating_with_kubernetes_CO5-2)

路径由 MicroProfile Health 规范定义

![3](img/#co_integrating_with_kubernetes_CO5-3)

定义就绪性探针

![4](img/#co_integrating_with_kubernetes_CO5-4)

路径由 MicroProfile Health 规范定义

## 讨论

Kubernetes 使用探针来确定服务的健康状态，并自动采取行动解决任何问题。

Quarkus 自动为 Kubernetes 生成两个探针：

存活性检查

Kubernetes 使用存活性探针来确定是否必须重新启动服务。如果应用程序变得无响应，可能是因为死锁或内存问题，重新启动容器可能是解决问题的良好解决方案。

就绪性检查

Kubernetes 使用就绪探针来确定服务是否可以接受流量。有时，服务在接受请求之前可能需要执行一些操作。例如更新本地缓存系统、将更改应用到数据库架构、应用批处理过程或连接到像 Kafka Streams 这样的外部服务。

## 参见

欲了解更多信息，请访问以下网站：

+   [Kubernetes：配置存活性、就绪性和启动探测](https://oreil.ly/PWl_B)

# 10.4 在 Kubernetes 上部署服务

## 问题

您希望在 Kubernetes 上部署服务。

## 解决方案

使用`kubectl`和 Quarkus 提供的所有功能在 Kubernetes 上创建和部署服务。

使用以下步骤，Quarkus 使得创建并在 Kubernetes 上部署 Java 应用程序变得非常简单：

1.  如 Recipe 6.6 中所述，生成企业应用程序的容器本地可执行文件。

1.  提供*Dockerfile.native*文件以构建 Docker 容器

1.  使用`quarkus-kubernetes`扩展生成 Kubernetes 资源文件，详见 Recipe 10.2

现在是时候看到所有这些步骤如何一起运作了。

若要创建可在容器内运行的本机可执行文件：

```java
./mvnw package -DskipTests -Pnative -Dquarkus.native.container-build=true ![1](img/1.png)

docker build -f src/main/docker/Dockerfile.native \
 -t alex/geeting-started:1.0-SNAPSHOT . ![2](img/2.png)
docker push docker build -f src/main/docker/Dockerfile.native \
 -t alex/getting-started:1.0-SNAPSHOT . ![3](img/3.png)

kubectl apply -f target/kubernetes/kubernetes.json ![4](img/4.png)

kubectl patch svc getting-started --type='json' \
 -p '[{"op":"replace","path":"/spec/type","value":"NodePort"}]' ![5](img/5.png)

curl $(minikube service getting-started --url)/hello ![6](img/6.png)
```

![1](img/#co_integrating_with_kubernetes_CO6-1)

在 Docker 容器内创建本机可执行文件

![2](img/#co_integrating_with_kubernetes_CO6-2)

使用之前生成的本机可执行文件创建 Docker 镜像

![3](img/#co_integrating_with_kubernetes_CO6-3)

将镜像推送到 Docker 注册表（在 minikube 中，这是`eval $(minikube docker-env)`，因此无需推送。）

![4](img/#co_integrating_with_kubernetes_CO6-4)

将应用程序部署到 Kubernetes

![5](img/#co_integrating_with_kubernetes_CO6-5)

更改为`NodePort`

![6](img/#co_integrating_with_kubernetes_CO6-6)

获取访问服务的 URL

请注意，仅因服务部署在 minikube 中，才需要执行第 5 和第 6 步。根据您用于部署服务的 Kubernetes 平台，您可能需要执行不同的操作。

## 讨论

如果使用多阶段 Docker 构建功能，步骤 1 和 2 可以简化为一个步骤。在第一阶段生成本机可执行文件，第二阶段创建运行时镜像：

```java
FROM quay.io/quarkus/centos-quarkus-maven:19.2.1 AS build ![1](img/1.png)
COPY src /usr/src/app/src
COPY pom.xml /usr/src/app
USER root
RUN chown -R quarkus /usr/src/app
USER quarkus
RUN mvn -f /usr/src/app/pom.xml -Pnative clean package

FROM registry.access.redhat.com/ubi8/ubi-minimal ![2](img/2.png)
WORKDIR /work/
COPY --from=build /usr/src/app/target/*-runner /work/application
RUN chmod 775 /work
EXPOSE 8080
CMD ["./application", "-Dquarkus.http.host=0.0.0.0"]
```

![1](img/#co_integrating_with_kubernetes_CO7-1)

生成本机可执行文件

![2](img/#co_integrating_with_kubernetes_CO7-2)

从上一阶段的输出创建运行时镜像

从项目根目录中删除*.dockerignore*文件（`rm .dockerignore`）。这是必要的，因为默认情况下会忽略*src*目录，而构建本机可执行文件需要*src*目录：

```java
docker build -f src/main/docker/Dockerfile.multistage -t docker \
 build -f src/main/docker/Dockerfile.multistage -t ![1](img/1.png)
```

![1](img/#co_integrating_with_kubernetes_CO8-1)

使用捆绑本机可执行文件创建运行时镜像

# 10.5 在 OpenShift 上部署服务

## 问题

您希望在 OpenShift 上部署服务。

## 解决方案

OpenShift 与前一个示例生成的资源完美配合，因此即使您在使用 OpenShift，仍然可以使用之前提供的所有内容。但是，如果您想使用 OpenShift 提供的某些功能，可以将`kubernetes.deployment.target`属性设置为`openshift`。

生成的两个文件分别位于*target/kubernetes/openshift.json*和*target/kubernetes/openshift.yaml*：

```java
{
  "apiVersion" : "v1",
  "kind" : "List",
  "items" : [ {
    "apiVersion" : "v1",
    "kind" : "Service",
    "metadata" : {
      "labels" : {
        "app" : "getting-started",
        "version" : "1.0-SNAPSHOT",
        "group" : "alex"
      },
      "name" : "getting-started"
    },
    "spec" : {
      "ports" : [ {
        "name" : "http",
        "port" : 8080,
        "targetPort" : 8080
      } ],
      "selector" : {
        "app" : "getting-started",
        "version" : "1.0-SNAPSHOT",
        "group" : "alex"
      },
      "type" : "ClusterIP"
    }
  }, {
    "apiVersion" : "image.openshift.io/v1",
    "kind" : "ImageStream",
    "metadata" : {
      "labels" : {
        "app" : "getting-started",
        "version" : "1.0-SNAPSHOT",
        "group" : "alex"
      },
      "name" : "getting-started"
    }
  }, {
    "apiVersion" : "image.openshift.io/v1",
    "kind" : "ImageStream",
    "metadata" : {
      "labels" : {
        "app" : "getting-started",
        "version" : "1.0-SNAPSHOT",
        "group" : "alex"
      },
      "name" : "s2i-java"
    },
    "spec" : {
      "dockerImageRepository" : "fabric8/s2i-java"
    }
  }, {
    "apiVersion" : "build.openshift.io/v1",
    "kind" : "BuildConfig",
    "metadata" : {
      "labels" : {
        "app" : "getting-started",
        "version" : "1.0-SNAPSHOT",
        "group" : "alex"
      },
      "name" : "getting-started"
    },
    "spec" : {
      "output" : {
        "to" : {
          "kind" : "ImageStreamTag",
          "name" : "getting-started:1.0-SNAPSHOT"
        }
      },
      "source" : {
        "binary" : { }
      },
      "strategy" : {
        "sourceStrategy" : {
          "from" : {
            "kind" : "ImageStreamTag",
            "name" : "s2i-java:2.3"
          }
        }
      }
    }
  }, {
    "apiVersion" : "apps.openshift.io/v1",
    "kind" : "DeploymentConfig",
    "metadata" : {
      "labels" : {
        "app" : "getting-started",
        "version" : "1.0-SNAPSHOT",
        "group" : "alex"
      },
      "name" : "getting-started"
    },
    "spec" : {
      "replicas" : 1,
      "selector" : {
        "app" : "getting-started",
        "version" : "1.0-SNAPSHOT",
        "group" : "alex"
      },
      "template" : {
        "metadata" : {
          "labels" : {
            "app" : "getting-started",
            "version" : "1.0-SNAPSHOT",
            "group" : "alex"
          }
        },
        "spec" : {
          "containers" : [ {
            "env" : [ {
              "name" : "KUBERNETES_NAMESPACE",
              "valueFrom" : {
                "fieldRef" : {
                  "fieldPath" : "metadata.namespace"
                }
              }
            }, {
              "name" : "JAVA_APP_JAR",
              "value" : "/deployments/getting-started-1.0-SNAPSHOT.jar"
            } ],
            "image" : "",
            "imagePullPolicy" : "IfNotPresent",
            "name" : "getting-started",
            "ports" : [ {
              "containerPort" : 8080,
              "name" : "http",
              "protocol" : "TCP"
            } ],
          } ]
        }
      },
      "triggers" : [ {
        "imageChangeParams" : {
          "automatic" : true,
          "containerNames" : [ "getting-started" ],
          "from" : {
            "kind" : "ImageStreamTag",
            "name" : "getting-started:1.0-SNAPSHOT"
          }
        },
        "type" : "ImageChange"
      } ]
    }
  } ]
}
```

# 10.6 构建和自动部署容器镜像

## 问题

您希望自动构建、推送和部署容器镜像。

## 解决方案

Quarkus 通过`container-image`扩展提供了构建和推送容器镜像的功能，并通过`kubernetes`扩展提供了部署到 Kubernetes 的功能。

要构建、推送和部署容器镜像，首先需要添加所需的扩展：

```java
./mvnw quarkus:add-extensions \
    -Dextensions="quarkus-container-image-jib, quarkus-kubernetes"
```

然后，您可以自定义容器镜像构建过程。您可以在*application.properties*、系统属性或环境变量中设置这些属性，就像 Quarkus 中的任何其他配置参数一样：

```java
quarkus.container-image.group=lordofthejars ![1](img/1.png)
quarkus.container-image.registry=quay.io ![2](img/2.png)
quarkus.container-image.username=lordofthejars ![3](img/3.png)
#quarkus.container-image.password= ![4](img/4.png)
```

![1](img/#co_integrating_with_kubernetes_CO9-1)

设置镜像的组部分；默认情况下为`${user.name}`

![2](img/#co_integrating_with_kubernetes_CO9-2)

将镜像推送到的注册表；默认情况下，镜像推送到`docker.io`

![3](img/#co_integrating_with_kubernetes_CO9-3)

登录容器注册表的用户名

![4](img/#co_integrating_with_kubernetes_CO9-4)

登录容器注册表的密码

最后，使用以下命令将部署到 Kubernetes：

```java
./mvnw clean package -Dquarkus.kubernetes.deploy=true
```

## 讨论

请注意，将`quarkus.kubernetes.deploy`设置为`true`会隐式地将`quarkus.container-image.push`属性设置为`true`，因此您无需手动设置它。

Kubernetes 扩展使用位于*~/.kube/config*的标准`kubectl`配置文件，以了解在哪里部署应用程序。

###### 提示

您还可以使用`-Pnative -Dquarkus.native.container-build=true`标志使用本地编译创建和部署容器镜像。

# 10.7 从 Kubernetes 配置应用程序

## 问题

您希望通过 Kubernetes 而不是配置文件来配置应用程序。

## 解决方案

使用`ConfigMaps`配置在 Pod 内运行的应用程序。

在此示例中，您将使用`ConfigMap`和 Kubernetes 扩展配置服务。要启用在`Pod`中注入`ConfigMap`的 Kubernetes 资源生成，您需要注册`quarkus-kubernetes`扩展：

```java
./mvnw quarkus:add-extension -Dextensions="quarkus-kubernetes"
```

当调用`/hello`端点时，服务返回`greeting.message`配置值：

```java
@ConfigProperty(name = "greeting.message")
String message;

@GET
@Produces(MediaType.TEXT_PLAIN)
public String hello() {
    return "hello " + message;
}
```

创建包含键值对的`ConfigMap`资源：

```java
apiVersion: v1
kind: ConfigMap ![1](img/1.png)
metadata:
    name: greeting-config
data:
    greeting: "Kubernetes" ![2](img/2.png)
```

![1](img/#co_integrating_with_kubernetes_CO10-1)

定义`ConfigMap`类型

![2](img/#co_integrating_with_kubernetes_CO10-2)

对于键`greeting`定义值为`Kubernetes`

然后，必须通过在终端窗口中运行以下命令将资源应用于 Kubernetes 集群：

```java
kubectl apply -f src/main/kubernetes/config-greeting.yaml
```

最后，在*application.properties*中设置 Kubernetes 扩展属性，以便生成的 Kubernetes 部署文件包含将 config map 作为环境变量注入的部分：

```java
greeting.message=local
quarkus.container-image.group=quarkus ![1](img/1.png)
quarkus.container-image.name=greeting-app
quarkus.container-image.tag=1.0-SNAPSHOT
quarkus.kubernetes.env-vars.greeting-message.value=greeting ![2](img/2.png)
quarkus.kubernetes.env-vars.greeting-message.configmap=greeting-config ![3](img/3.png)
quarkus.kubernetes.image-pull-policy=if-not-present
```

![1](img/#co_integrating_with_kubernetes_CO11-1)

配置 Docker 镜像

![2](img/#co_integrating_with_kubernetes_CO11-2)

将环境变量设置为覆盖`greeting.message`属性

![3](img/#co_integrating_with_kubernetes_CO11-3)

设置要加载的 config map 资源名称

生成的 Kubernetes 文件将包含一个新条目，定义容器定义中的键值对，称为`configMapKeyRef`。

要部署应用程序，请打开新的终端窗口，打包应用程序，创建 Docker 容器，并应用生成的 Kubernetes 资源：

```java
./mvnw clean package -DskipTests

docker build -f src/main/docker/Dockerfile.jvm \
 -t quarkus/greeting-app:1.0-SNAPSHOT .
kubectl apply -f target/kubernetes/kubernetes.yml

kubectl patch svc greeting-app --type='json' \
 -p '[{"op":"replace","path":"/spec/type","value":"NodePort"}]'
curl $(minikube service greeting-app --url)/hello
```

## 讨论

`ConfigMap`由 Kubernetes 注入到 pod 的容器中，以文件或环境变量的形式组成键值对，使应用程序可以读取并相应地进行配置。通过`ConfigMaps`，可以将应用程序的配置与业务逻辑解耦，从而使其在不同环境中具备可移植性。

###### 重要

`ConfigMaps`用于存储和共享非敏感配置属性。

MicroProfile Config 规范允许通过等效的环境变量（大写并将点[`.`]改为下划线[`_`]）来覆盖任何配置属性。`ConfigMap`包含配置属性。在*application.properties*中，配置了 Kubernetes 扩展以生成部署描述符，将这些属性设置为环境变量，因此当在 Kubernetes 集群内启动容器时，使用适用于该集群的特定配置。

## 参见

要了解更多关于 Kubernetes 中`ConfigMaps`的信息，请访问以下网站：

+   [Kubernetes: 配置 Pod 使用 ConfigMap](https://oreil.ly/BPmo5)

# 10.8 通过 Config 扩展从 Kubernetes 配置应用程序

## 问题

您希望通过 Kubernetes 而不是配置文件来配置您的应用程序，使用 MicroProfile Config 规范。

## 解决方案

Quarkus 提供了一个 Kubernetes 配置扩展，可以从 Kubernetes API 服务器中读取 secrets 和 config maps 元素，并使用`@ConfigProperty`注解进行注入。

要启用生成 Kubernetes 资源，请注册`quarkus-kubernetes-config`扩展。

该扩展支持注入`ConfigMaps`，可以是单个键/值形式，也可以是键为文件名的形式（仅支持`application.properties`或`application.yaml`），其值是文件的内容。

让我们创建一个带有单个键/值的 config map：

```java
apiVersion: v1
kind: ConfigMap
metadata:
    name: my-config ![1](img/1.png)
data:
    greeting: "Kubernetes"
```

![1](img/#co_integrating_with_kubernetes_CO12-1)

对于扩展来说，配置名称很重要。

然后注册前述的 `ConfigMap` 资源：

```java
kubectl apply -f src/main/kubernetes/my-config.yaml
```

对于此示例，还注册了一个名为 *application.properties* 的 `ConfigMap` 文件。

添加的配置文件包含以下内容：

```java
some.property1=prop1
some.property2=prop2
```

然后将前述文件注册到名为 `my-file-config` 的 `ConfigMap` 中：

```java
kubectl create configmap my-file-config \
 --from-file=./src/main/kubernetes/application.properties
```

在注入这些值之前的最后一步是配置扩展程序以从这些 `ConfigMaps` 中读取值：

```java
quarkus.kubernetes-config.enabled=true ![1](img/1.png)
quarkus.kubernetes-config.config-maps=my-config,my-file-config ![2](img/2.png)
```

![1](img/#co_integrating_with_kubernetes_CO13-1)

启用扩展

![2](img/#co_integrating_with_kubernetes_CO13-2)

设置 `ConfigMap` 的名称

这些配置值像其他配置值一样注入：

```java
@ConfigProperty(name = "greeting") ![1](img/1.png)
String greeting;

@ConfigProperty(name = "some.property1") ![2](img/2.png)
String property1;

@ConfigProperty(name = "some.property2")
String property2;
```

![1](img/#co_integrating_with_kubernetes_CO14-1)

注入简单键

![2](img/#co_integrating_with_kubernetes_CO14-2)

也注入了 *application.properties* 文件中的键

要部署应用程序，请打开新的终端窗口，打包应用程序，创建 Docker 容器，并应用生成的 Kubernetes 资源：

```java
./mvnw clean package -DskipTests

docker build -f src/main/docker/Dockerfile.jvm \
 -t quarkus/greeting-app:1.0-SNAPSHOT .
kubectl apply -f target/kubernetes/kubernetes.yml

kubectl patch svc greeting-app-config-ext --type='json' \
 -p '[{"op":"replace","path":"/spec/type","value":"NodePort"}]'

curl $(minikube service greeting-app-config-ext --url)/hello
Kubernetes

curl $(minikube service greeting-app-config-ext --url)/hello/p1
prop1

curl $(minikube service greeting-app-config-ext --url)/hello/p2
prop2
```

# 10.9 以编程方式与 Kubernetes 集群交互

## 问题

您希望以编程方式与 Kubernetes API 服务器进行交互。

## 解决方案

使用 `kubernetes-client` 扩展程序开始观察和响应 Kubernetes 资源的更改。

要添加`kubernetes-extension`，运行以下命令：

```java
./mvnw quarkus:add-extension -Dextensions="kubernetes-client"
```

连接到 Kubernetes 集群的主要类是 `io.fabric8.kubernetes.client.KubernetesClient`。扩展程序生成此实例以便在代码中注入。可以使用各种属性配置客户端，将它们设置在 *application.properties* 中。

此处开发的示例是一个端点，返回给定命名空间上所有部署的 Pod 的名称：

```java
package org.acme.quickstart;

import java.util.List;
import java.util.stream.Collectors;

import javax.inject.Inject;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

import io.fabric8.kubernetes.client.KubernetesClient;

@Path("/pod")
public class PodResource {

    @Inject ![1](img/1.png)
    KubernetesClient kubernetesClient;

    @GET
    @Produces(MediaType.APPLICATION_JSON)
    @Path("/{namespace}")
    public List<String> getPods(@PathParam("namespace") String namespace) {
        return kubernetesClient.pods() ![2](img/2.png)
                                .inNamespace(namespace) ![3](img/3.png)
                                .list().getItems()
                                .stream()
                                .map(p -> p.getMetadata().getGenerateName()) ![4](img/4.png)
                                .collect(Collectors.toList());
    }
}
```

![1](img/#co_integrating_with_kubernetes_CO15-1)

`KubernetesClient` 被注入，就像任何其他 CDI bean 一样

![2](img/#co_integrating_with_kubernetes_CO15-2)

选择所有的 Pods

![3](img/#co_integrating_with_kubernetes_CO15-3)

来自给定命名空间

![4](img/#co_integrating_with_kubernetes_CO15-4)

仅获取 Pod 的生成名称

访问 Kubernetes 的 REST API 推荐的方式是使用 `kubectl` 代理模式，因为这样可以避免中间人攻击。

另一种方法是直接提供位置和凭据，但为了避免中间人攻击，可能需要导入根证书。

因为代理模式是推荐的方式，所以本示例中使用了此方法。

指向应用程序必须连接的集群的 `kubectl`，打开新的终端窗口并运行以下命令：

```java
kubectl proxy --port=8090
```

此命令以反向代理方式运行 `kubectl`，在 [*http://localhost:8090*](http://localhost:8090) 上公开远程 Kubernetes API 服务器。

配置 `KubernetesClient` 以通过 *application.properties* 中的 `quarkus.kubernetes-client.master-url` 属性连接到 [*http://localhost:8090*](http://localhost:8090)：

```java
%dev.quarkus.kubernetes-client.master-url=http://localhost:8090 ![1](img/1.png)
```

![1](img/#co_integrating_with_kubernetes_CO16-1)

设置 Kubernetes API 服务器的 URL

最后，运行服务并请求`/pod/default`端点，以获取在默认命名空间中部署的所有 pod：

```java
./mvnw compile quarkus:dev

curl http://localhost:8080/pod/default
["getting-started-5cd97ddd4d-"]
```

## 讨论

在某些情况下，您需要以编程方式创建新的 Kubernetes 资源或获取有关 Kubernetes 集群/资源的一些信息（已部署的 pod、配置参数、设置密码等）。`kubernetes-client`真正出色的地方在于在 Java 中实现*Kubernetes Operator*。由于 Quarkus 的能力可以生成本机可执行文件，这是在 Java 中实现 Kubernetes Operator 的绝佳方式。

在本例中，服务是在 Kubernetes 集群外部部署的，并且您使用 Kubernetes API 服务器连接到它。

如果部署了需要访问的 Kubernetes 集群中的服务，则必须将`quarkus.kubernetes-client.master-url`属性设置为`https://kubernetes.default.svc`。

可以通过简单声明返回已配置`KubernetesClient`实例的 CDI 提供程序工厂方法来重写`KubernetesClient`的创建：

```java
@ApplicationScoped
public class KubernetesClientProducer {

    @Produces
    public KubernetesClient kubernetesClient() {
        Config config = new ConfigBuilder()
                              .withMasterUrl("https://mymaster.com")
                              .build(); ![1](img/1.png)
        return new DefaultKubernetesClient(config); ![2](img/2.png)
    }
}
```

![1](img/#co_integrating_with_kubernetes_CO17-1)

配置客户端

![2](img/#co_integrating_with_kubernetes_CO17-2)

创建`KubernetesClient`的实例

在大多数情况下，要访问 Kubernetes API 服务器，需要`ServiceAccount`、`Role`和`RoleBinding`。以下可能是在本节提供的示例中工作的起点：

```java
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: greeting-started
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: greeting-started
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: greeting-started
  namespace: default
roleRef:
  kind: Role
  name: greeting-started
  apiGroup: rbac.authorization.k8s.io
subjects:
  - kind: ServiceAccount
    name: greeting-started
    namespace: default
```

## 参见

要了解有关 Fabric8 Kubernetes 客户端的更多信息，请访问 GitHub 上的以下页面：

+   [Kubernetes & OpenShift Java Client](https://oreil.ly/_QNSL)

# 10.10 测试 Kubernetes 客户端交互

## 问题

您想要测试 Kubernetes 客户端代码。

## 解决方案

Quarkus 实现了一个 Quarkus 测试资源，用于启动 Kubernetes API 服务器的模拟，并设置正确的配置以使 Kubernetes 客户端使用模拟服务器实例而不是*application.properties*中提供的值。此外，您可以设置模拟服务器以响应任何特定测试所需的任何 canned 请求：

```java
package org.acme.quickstart;

import io.fabric8.kubernetes.api.model.Pod;
import io.fabric8.kubernetes.api.model.PodBuilder;
import io.fabric8.kubernetes.api.model.PodListBuilder;
import io.fabric8.kubernetes.client.server.mock.KubernetesMockServer;
import io.quarkus.test.common.QuarkusTestResource;
import io.quarkus.test.junit.QuarkusTest;
import io.quarkus.test.kubernetes.client.KubernetesMockServerTestResource;
import io.quarkus.test.kubernetes.client.MockServer;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import static io.restassured.RestAssured.given;
import static org.hamcrest.CoreMatchers.is;

@QuarkusTest
@QuarkusTestResource(KubernetesMockServerTestResource.class) ![1](img/1.png)
public class PodResourceTest {

    @MockServer ![2](img/2.png)
    KubernetesMockServer mockServer;

    @BeforeEach ![3](img/3.png)
    public void prepareKubernetesServerAPI() {
        final Pod pod1 = new PodBuilder()
                .withNewMetadata()
                .withName("pod1")
                .withNamespace("test")
                .withGenerateName("pod1-12345")
                .and()
                .build(); ![4](img/4.png)

        mockServer
                .expect()
                  .get()
                    .withPath("/api/v1/namespaces/test/pods")
                    .andReturn(200, new PodListBuilder()
                        .withNewMetadata()
                        .withResourceVersion("1")
                        .endMetadata()
                        .withItems(pod1).build()) ![5](img/5.png)
                .always();

    }

    @Test
    public void testHelloEndpoint() {
        given()
          .when().get("/pod/test")
          .then()
             .statusCode(200)
             .body(is("[\"pod1-12345\"]"));
    }

}
```

![1](img/#co_integrating_with_kubernetes_CO18-1)

设置 Kubernetes 测试资源模拟服务器

![2](img/#co_integrating_with_kubernetes_CO18-2)

注入 Kubernetes 模拟服务器实例以记录任何交互

![3](img/#co_integrating_with_kubernetes_CO18-3)

为了保持测试隔离性，在每次测试之前，再次记录交互

![4](img/#co_integrating_with_kubernetes_CO18-4)

构建要返回的 pod

![5](img/#co_integrating_with_kubernetes_CO18-5)

通过查询`test`命名空间中的所有 pod，返回一个 pod 作为结果

# 10.11 实现 Kubernetes 运算符

## 问题

您想要实现一个 Kubernetes 运算符，以使用自定义资源扩展 Kubernetes 以管理 Java 中的应用程序。

## 解决方案

使用`kubernetes-client`扩展和 Quarkus 在 Java 中实现 Kubernetes 运算符，并将其编译为本机可执行文件。

操作员的一个用例是创建一个模板（*自定义资源*），其中在创建时设置一些值。文件模板和操作员之间最大的区别在于，通用内容（在*模板*的情况下）是静态的，而在操作员中是以编程方式设置的，这意味着您可以自由地动态更改公共部分的定义。这被称为*自定义资源*，其中，您不是使用一个众所周知的 Kubernetes 资源，而是使用您自己的字段实现自己的自定义 Kubernetes 资源。

另一个用例可能是在集群内部发生某事时做出反应/操作。假设您在集群上部署了一些内存数据网格，并且其中一个实例停止运行。也许在这种情况下，您希望通知所有存活的实例，数据网格集群的一个元素已经停止。

正如您所看到的，这不仅仅是创建资源，还涉及到应用一些特定于您的应用程序的任务，这些任务需要在 Kubernetes 已经在执行的任务之上执行。

Kubernetes 操作员使用 Kubernetes API 来决定何时以及如何运行其中一些定制化内容。

从逻辑实现的角度来看，以下简单示例并没有太多意义，但它将帮助您理解编写 Kubernetes 操作员的基础知识。将其用作实现自己的 Kubernetes 操作员的起点。

要编写 Kubernetes 操作员，可能需要以下元素：

1.  解析自定义资源的类。

1.  注册并生成客户端以操作自定义资源的工厂方法。

1.  当将自定义资源应用于集群时，反应的观察者。您可以将其视为操作员控制器或操作员实现。

1.  具有所有先前代码的 Docker 镜像。

1.  用于定义自定义资源（`CustomResourceDefinition`）的 YAML/JSON 文件。

1.  部署文件以部署自定义操作员。

让我们实现一个简单的 Kubernetes 操作员，配置在容器中运行的命令，并使用此配置实例化 pod。

用于示例的基础镜像是 [Whalesay](https://oreil.ly/98T7t)，它基本上会在容器控制台中打印您在`run`命令中传递的消息，就像这样：

```java
docker run docker/whalesay cowsay boo
 _____
< boo >
 -----
 \
 \
 \
 ##        .
 ## ## ##       ==
 ## ## ## ##      ===
 /""""""""""""""""___/ ===
 ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
 \______ o          __/
 \    \        __/
 \____\______/
```

使用此镜像的 pod 资源的示例可能如下所示：

```java
apiVersion: v1
kind: Pod
metadata:
  name: whalesay
spec:
  containers:
  - name: whalesay
    image: docker/whalesay
    imagePullPolicy: "IfNotPresent"
    command: ["cowsay","Hello Alex"] ![1](img/1.png)
```

![1](img/#co_integrating_with_kubernetes_CO19-1)

设置输出消息

此操作员的目标是只需提供要打印的消息。其余内容（例如 Docker 镜像、容器配置等）将由 Kubernetes 操作员自动设置。

要创建自定义操作员，需要 Kubernetes 客户端和 Jackson 依赖项：

```java
./mvnw quarkus:add-extension \
-Dextensions="io.quarkus:quarkus-kubernetes-client, io.quarkus:quarkus-jackson"
```

首先要做的是定义自定义资源的外观。例如，对于这个示例，它看起来如下所示：

```java
apiVersion: acme.org/v1alpha1
kind: Hello ![1](img/1.png)
metadata:
  name: example-hello
spec:
  message: Hello Alex ![2](img/2.png)
```

![1](img/#co_integrating_with_kubernetes_CO20-1)

使用自定义 `kind` 架构（稍后在本文档中定义）

![2](img/#co_integrating_with_kubernetes_CO20-2)

设置要打印的消息

需要对象模型来解析自定义资源。 在这种情况下，使用 Jackson 库将 YAML 映射到 Java 对象。 需要三个类，一个用于整个资源，另一个用于 `spec` 部分，另一个用于 `status` 部分，后者为空但是需要，因为集群可能会自动填充它。

在 `org.acme.quickstart.cr` 包内的 *src/main/java* 中创建它们：

```java
package org.acme.quickstart.cr;

import com.fasterxml.jackson.databind.annotation.JsonDeserialize;

import io.fabric8.kubernetes.client.CustomResource;

@JsonDeserialize ![1](img/1.png)
public class HelloResource extends CustomResource { ![2](img/2.png)

    private HelloResourceSpec spec; ![3](img/3.png)
    private HelloResourceStatus status; ![4](img/4.png)

    public HelloResourceStatus getStatus() {
        return status;
    }

    public void setStatus(HelloResourceStatus status) {
        this.status = status;
    }

    public HelloResourceSpec getSpec() {
        return spec;
    }

    public void setSpec(HelloResourceSpec spec) {
        this.spec = spec;
    }

    @Override
    public String toString() {
        return "name=" + getMetadata().getName()
                + ", version=" + getMetadata().getResourceVersion()
                + ", spec=" + spec;
    }
}
```

![1](img/#co_integrating_with_kubernetes_CO21-1)

将 POJO 设置为可反序列化

![2](img/#co_integrating_with_kubernetes_CO21-2)

继承通用自定义资源字段，如 `kind`、`apiVersion` 或 `metadata`

![3](img/#co_integrating_with_kubernetes_CO21-3)

自定义 `spec` 部分

![4](img/#co_integrating_with_kubernetes_CO21-4)

`status` 部分

`spec` 部分映射如下：

```java
package org.acme.quickstart.cr;

import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.databind.annotation.JsonDeserialize;

@JsonDeserialize
public class HelloResourceSpec {

    @JsonProperty("message") ![1](img/1.png)
    private String message;

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    @Override
    public String toString() {
        return "HelloResourceSpec [message=" + message + "]";
    }

}
```

![1](img/#co_integrating_with_kubernetes_CO22-1)

自定义规格仅包含一个 `message` 字段

空的 `status` 部分映射如下：

```java
package org.acme.quickstart.cr;

import com.fasterxml.jackson.databind.annotation.JsonDeserialize;

@JsonDeserialize
public class HelloResourceStatus {
}
```

从模型观点仍然需要两个类。

当不仅仅是将单个自定义资源（如前所示）应用于集群时，而是提供了自定义资源列表（使用`items`数组）时，使用 `One class`。

```java
package org.acme.quickstart.cr;

import com.fasterxml.jackson.databind.annotation.JsonDeserialize;

import io.fabric8.kubernetes.client.CustomResourceList;

@JsonDeserialize
public class HelloResourceList extends CustomResourceList<HelloResource> { ![1](img/1.png)
}
```

![1](img/#co_integrating_with_kubernetes_CO23-1)

`CustomResourceList` 继承了支持自定义资源列表的所有必需字段

另一个类用于从操作员实现中使自定义资源可编辑：

```java
package org.acme.quickstart.cr;

import io.fabric8.kubernetes.api.builder.Function;
import io.fabric8.kubernetes.client.CustomResourceDoneable;

public class HelloResourceDoneable
    extends CustomResourceDoneable<HelloResource> { ![1](img/1.png)
    public HelloResourceDoneable(HelloResource resource, Function<HelloResource,
                                 HelloResource> function) {
        super(resource, function);
    }
}
```

![1](img/#co_integrating_with_kubernetes_CO24-1)

`CustomResourceDoneable` 类使资源可编辑

接下来所需的另一件重要事情是一个 CDI 工厂 bean，它提供了操作员所需的所有机制。 在 *src/main/java* 的 `org.acme.quickstart` 包中创建此类：

```java
package org.acme.quickstart;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;

import javax.enterprise.inject.Produces;
import javax.inject.Named;
import javax.inject.Singleton;

import org.acme.quickstart.cr.HelloResource;
import org.acme.quickstart.cr.HelloResourceDoneable;
import org.acme.quickstart.cr.HelloResourceList;

import io.fabric8.kubernetes.api.model.apiextensions.CustomResourceDefinition;
import io.fabric8.kubernetes.client.DefaultKubernetesClient;
import io.fabric8.kubernetes.client.KubernetesClient;
import io.fabric8.kubernetes.client.dsl.MixedOperation;
import io.fabric8.kubernetes.client.dsl.Resource;
import io.fabric8.kubernetes.internal.KubernetesDeserializer;

public class KubernetesProducer {

  @Produces
  @Singleton
  @Named("namespace")
  String findMyCurrentNamespace() throws IOException { ![1](img/1.png)
    return new String(Files.readAllBytes(
          Paths
            .get("/var/run/secrets/kubernetes.io/serviceaccount/namespace")));
  }

  @Produces
  @Singleton
  KubernetesClient makeDefaultClient(@Named("namespace") String namespace) {
    return new DefaultKubernetesClient().inNamespace(namespace); ![2](img/2.png)
  }

  @Produces
  @Singleton
  MixedOperation<HelloResource,
                 HelloResourceList,
                 HelloResourceDoneable,
                 Resource<HelloResource, HelloResourceDoneable>>
  makeCustomHelloResourceClient(KubernetesClient defaultClient) { ![3](img/3.png)
    KubernetesDeserializer
        .registerCustomKind("acme.org/v1alpha1",
                            "Hello", HelloResource.class); ![4](img/4.png)
    CustomResourceDefinition crd = defaultClient.customResourceDefinitions()
                                      .list()
                                      .getItems()
                                      .stream()
                                      .findFirst()
      .orElseThrow(RuntimeException::new); ![5](img/5.png)
    return defaultClient.customResources(crd, HelloResource.class,
        HelloResourceList.class,
        HelloResourceDoneable.class); ![6](img/6.png)
  }
}
```

![1](img/#co_integrating_with_kubernetes_CO25-1)

获取操作员正在运行的命名空间

![2](img/#co_integrating_with_kubernetes_CO25-2)

配置 `KubernetesClient` 与当前命名空间；默认适用于 Kubernetes Operator 开发

![3](img/#co_integrating_with_kubernetes_CO25-3)

`MixedOperation` 用于监视有关自定义资源的事件（例如，应用新的自定义资源时）

![4](img/#co_integrating_with_kubernetes_CO25-4)

注册要由 `org.acme.quickstart.cr.HelloResource` 解析的 `apiVersion` 和 `Kind`

![5](img/#co_integrating_with_kubernetes_CO25-5)

获取自定义资源的定义；因为只有一个（即我们正在开发的），可以使用 `findFirst`

![6](img/#co_integrating_with_kubernetes_CO25-6)

注册客户端资源、解析器、列表解析器和 `doneable` 类

最后需要实现的 Java 类是控制器。这个控制器（或者观察者/操作者）负责检查集群内部的运行情况，并对订阅的事件做出响应，例如，一个新的 Pod 已经被创建或销毁，或者应用了一个名为`Hello`的自定义资源。

在这个实现中，控制器会监视当一个新的`Hello`资源被添加时。当自定义资源被应用时，然后从模型中检索消息，并使用 Kubernetes 客户端 API 提供的所有构建器创建 Pod 定义。最终，将 Pod 部署到 Kubernetes 集群中。

在`org.acme.quickstart`包内的*src/main/java*中创建这个类：

```java
package org.acme.quickstart;

import java.util.HashMap;
import java.util.Map;

import javax.enterprise.event.Observes;
import javax.inject.Inject;

import org.acme.quickstart.cr.HelloResource;
import org.acme.quickstart.cr.HelloResourceDoneable;
import org.acme.quickstart.cr.HelloResourceList;

import io.fabric8.kubernetes.api.model.ContainerBuilder;
import io.fabric8.kubernetes.api.model.HasMetadata;
import io.fabric8.kubernetes.api.model.ObjectMetaBuilder;
import io.fabric8.kubernetes.api.model.Pod;
import io.fabric8.kubernetes.api.model.PodBuilder;
import io.fabric8.kubernetes.api.model.PodSpecBuilder;
import io.fabric8.kubernetes.client.KubernetesClient;
import io.fabric8.kubernetes.client.KubernetesClientException;
import io.fabric8.kubernetes.client.Watcher;
import io.fabric8.kubernetes.client.dsl.MixedOperation;
import io.fabric8.kubernetes.client.dsl.Resource;
import io.quarkus.runtime.StartupEvent;

public class HelloResourceWatcher {

  @Inject
  KubernetesClient defaultClient; ![1](img/1.png)

  @Inject
  MixedOperation<HelloResource,
    HelloResourceList,
    HelloResourceDoneable,
    Resource<HelloResource,
    HelloResourceDoneable>> crClient; ![2](img/2.png)

  void onStartup(@Observes StartupEvent event) { ![3](img/3.png)
    crClient.watch(new Watcher<HelloResource>() { ![4](img/4.png)
      @Override
      public void eventReceived(Action action, HelloResource resource) {
        System.out.println("Received " + action
            + " event for resource " + resource);
        if (action == Action.ADDED) {
          final String app = resource.getMetadata().getName(); ![5](img/5.png)
          final String message = resource.getSpec().getMessage();

          final Map<String, String> labels = new HashMap<>(); ![6](img/6.png)
          labels.put("app", app);

          final ObjectMetaBuilder objectMetaBuilder =
            new ObjectMetaBuilder().withName(app + "-pod")
            .withNamespace(resource.getMetadata()
                .getNamespace())
            .withLabels(labels);

          final ContainerBuilder containerBuilder =
            new ContainerBuilder().withName("whalesay")
            .withImage("docker/whalesay")
            .withCommand("cowsay", message); ![7](img/7.png)

          final PodSpecBuilder podSpecBuilder =
            new PodSpecBuilder()
            .withContainers(containerBuilder.build())
            .withRestartPolicy("Never");

          final PodBuilder podBuilder =
            new PodBuilder()
            .withMetadata(objectMetaBuilder.build())
            .withSpec(podSpecBuilder.build());

          final Pod pod = podBuilder.build(); ![8](img/8.png)
          HasMetadata result = defaultClient
            .resource(pod)
            .createOrReplace(); ![9](img/9.png)

          if (result == null) {
            System.out.println("Pod " + pod
                + " couldn't be created");
          } else {
            System.out.println("Pod " + pod + " created");
          }
        }
      }

      @Override
      public void onClose(KubernetesClientException e) { ![10](img/10.png)
        if (e != null) {
          e.printStackTrace();
          System.exit(-1);
        }
      }
    });
  }
}
```

![1](img/#co_integrating_with_kubernetes_CO26-1)

注入 KubernetesClient

![2](img/#co_integrating_with_kubernetes_CO26-2)

注入特定于开发的自定义资源的操作

![3](img/#co_integrating_with_kubernetes_CO26-3)

在应用程序启动时执行逻辑

![4](img/#co_integrating_with_kubernetes_CO26-4)

监视任何涉及`HelloResource`的操作

![5](img/#co_integrating_with_kubernetes_CO26-5)

获取自定义资源中提供的信息

![6](img/#co_integrating_with_kubernetes_CO26-6)

程序化地开始创建 Pod 定义

![7](img/#co_integrating_with_kubernetes_CO26-7)

设置自定义资源提供的消息

![8](img/#co_integrating_with_kubernetes_CO26-8)

构建 Pod

![9](img/#co_integrating_with_kubernetes_CO26-9)

将 Pod 添加到集群中

![10](img/#co_integrating_with_kubernetes_CO26-10)

如果在关闭时出现任何关键错误，则停止容器

## 讨论

在 Java 端，这就是你需要做的一切；然而，还有一些未完成的部分，例如打包和容器化操作符，或者在集群中定义自定义操作符。

开发 Kubernetes Operator 时首要考虑的事项是与 Kubernetes API 服务器的通信通过 HTTPS 进行，并且这意味着如果默认情况下未提供加密库，则必须在 Docker 镜像中提供它们。

在撰写本文时，Quarkus 提供的*Dockerfile.jvm*文件中不包含与 Kubernetes 服务器通信所需的加密库。要解决这个问题，只需打开*src/main/docker/Dockerfile.jvm*文件并添加`nss`（Network Security Services）包：

```java
FROM fabric8/java-alpine-openjdk8-jre

RUN apk add --no-cache nss
```

然后通过运行 Maven 和 Docker 将操作符容器化：

```java
./mvnw clean package

docker build -f src/main/docker/Dockerfile.jvm \
 -t lordofthejars/quarkus-operator-example:1.0.0 .
```

然后将自定义资源定义注册到 Kubernetes 集群中，以便它了解新资源类型、自定义资源的范围或组名称等信息。

创建一个名为*custom-resource-definition.yaml*的文件，在*src/main/kubernetes*中定义所有集群注册新资源所需的信息：

```java
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: hellos.acme.org ![1](img/1.png)
spec:
  group: acme.org ![2](img/2.png)
  names:
    kind: Hello ![3](img/3.png)
    listKind: HelloList ![4](img/4.png)
    plural: hellos ![5](img/5.png)
    singular: hello ![6](img/6.png)
  scope: Namespaced ![7](img/7.png)
  subresources:
    status: {}
  version: v1alpha1 ![8](img/8.png)
```

![1](img/#co_integrating_with_kubernetes_CO27-1)

`plural` 加上 `group`

![2](img/#co_integrating_with_kubernetes_CO27-2)

设置自定义资源的组（用于自定义资源的 `apiVersion` 字段）

![3](img/#co_integrating_with_kubernetes_CO27-3)

`kind` 的名称

![4](img/#co_integrating_with_kubernetes_CO27-4)

当种类是此自定义资源列表时的名称

![5](img/#co_integrating_with_kubernetes_CO27-5)

复数名字

![6](img/#co_integrating_with_kubernetes_CO27-6)

单数名字

![7](img/#co_integrating_with_kubernetes_CO27-7)

资源的范围

![8](img/#co_integrating_with_kubernetes_CO27-8)

资源的版本（用于自定义资源的 `apiVersion` 字段）

最后要创建的是部署操作者的部署文件。在 *src/main/kubernetes* 下创建一个名为 *deploy.yaml* 的新文件：

```java
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole ![1](img/1.png)
metadata:
  name: quarkus-operator-example
rules:
- apiGroups:
  - ''
  resources:
  - pods ![2](img/2.png)
  verbs:
  - get
  - list
  - watch
  - create
  - update
  - delete
  - patch
- apiGroups:
  - apiextensions.k8s.io
  resources:
  - customresourcedefinitions
  verbs:
  - list
- apiGroups:
  - acme.org ![3](img/3.png)
  resources:
  - hellos
  verbs:
  - list
  - watch
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: quarkus-operator-example
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: quarkus-operator-example
subjects:
- kind: ServiceAccount
  name: quarkus-operator-example
  namespace: default
roleRef:
  kind: ClusterRole
  name: quarkus-operator-example
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: apps/v1
kind: Deployment ![4](img/4.png)
metadata:
  name: quarkus-operator-example
spec:
  selector:
    matchLabels:
      app: quarkus-operator-example
  replicas: 1
  template:
    metadata:
      labels:
        app: quarkus-operator-example
    spec:
      serviceAccountName: quarkus-operator-example ![5](img/5.png)
      containers:
      - image: lordofthejars/quarkus-operator-example:1.0.0 ![6](img/6.png)
        name: quarkus-operator-example
        imagePullPolicy: IfNotPresent
```

![1](img/#co_integrating_with_kubernetes_CO28-1)

为 Kubernetes 资源的基于角色的访问控制（RBAC）定义了一个集群角色

![2](img/#co_integrating_with_kubernetes_CO28-2)

添加了获取、列表、观察、创建、更新、删除和修补 Pod 的权限

![3](img/#co_integrating_with_kubernetes_CO28-3)

对于自定义资源（`hellos.acme.org`），所需的操作是 `list` 和 `watch`

![4](img/#co_integrating_with_kubernetes_CO28-4)

操作者使用 `Deployment` 部署

![5](img/#co_integrating_with_kubernetes_CO28-5)

设置与文件中定义的集群角色关联的服务帐户

![6](img/#co_integrating_with_kubernetes_CO28-6)

设置包含操作者的容器镜像

在操作者启动之前的最后一步是应用所有这些创建的资源：

```java
kubectl apply -f src/main/kubernetes/custom-resource-definition.yaml
kubectl apply -f src/main/kubernetes/deploy.yaml

kubectl get pods

NAME                                       READY   STATUS    RESTARTS   AGE
quarkus-operator-example-fb77dc468-8v9xk   1/1     Running   0          5s
```

操作者现已安装并运行。要测试操作者，只需创建一种 `Hello` 种类的自定义资源，并提供要显示的消息：

```java
apiVersion: acme.org/v1alpha1
kind: Hello ![1](img/1.png)
metadata:
  name: example-hello
spec:
  message: Hello Alex ![2](img/2.png)
```

![1](img/#co_integrating_with_kubernetes_CO29-1)

使用自定义 `kind` 模式

![2](img/#co_integrating_with_kubernetes_CO29-2)

设置要打印的消息

然后按如下方式应用：

```java
kubectl apply -f src/main/kubernetes/custom-resource.yaml

kubectl get pods

NAME                                       READY   STATUS      RESTARTS   AGE
example-hello-pod                          0/1     Completed   0          2m57s
quarkus-operator-example-fb77dc468-8v9xk   1/1     Running     0          3m24s
```

完成后，请检查 Pod 日志，验证消息是否已打印到控制台上：

```java
kubectl logs example-hello-pod

 ____________
< Hello Alex >
 ------------
 \
 \
 \
 ##        .
 ## ## ##       ==
 ## ## ## ##      ===
 /""""""""""""""""___/ ===
 ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
 \______ o          __/
 \    \        __/
 \____\______/
```

尽管操作者和自定义资源通常是相关的，但没有自定义资源定义的操作者仍然是可能的——例如，创建一个监视器类来拦截影响 Pod 的任何事件并应用一些逻辑。

## 参见

要了解更多关于操作者的信息，请查看以下网站：

+   [CoreOS：Operators](https://oreil.ly/NV2dN)

+   [Kubernetes：操作者模式](https://oreil.ly/6Z77K)

# 10.12 使用 Knative 部署和管理无服务器工作负载

## 问题

您想要部署和管理无服务器工作负载。

## 解决方案

使用 Knative，基于 Kubernetes 的平台，部署和管理现代无服务器工作负载。

`quarkus-kubernetes` 扩展提供了自动生成 Knative 资源的支持，具有合理的默认设置和可选的用户提供的配置。

要启用 Kubernetes 资源的生成，您需要注册 `quarkus-kubernetes` 扩展：

```java
./mvnw quarkus:add-extension \
 -Dextensions="quarkus-kubernetes, quarkus-container-image-docker"
```

对于本示例，使用 `quarkus-container-image-docker` 扩展来使用 `docker` 二进制文件构建容器镜像，因此镜像直接在 `minikube` 集群内构建并在内部注册表中注册，因此不需要外部注册表。

您需要运行 `eval $(minikube docker-env)` 来配置 `docker` 使用 minikube docker 主机。

然后，您需要将 `quarkus.kubernetes.deployment-target` 属性设置为 `knative`，并在打包阶段设置为构建 Docker 容器，以及其他有关容器镜像创建的配置属性：

```java
quarkus.kubernetes.deployment-target=knative ![1](img/1.png)
quarkus.container-image.build=true ![2](img/2.png)
quarkus.container-image.group=lordofthejars
quarkus.container-image.registry=dev.local ![3](img/3.png)
```

![1](img/#co_integrating_with_kubernetes_CO30-1)

将目标部署设置为 `knative`

![2](img/#co_integrating_with_kubernetes_CO30-2)

使用 `lordofthejars` 组构建容器镜像

![3](img/#co_integrating_with_kubernetes_CO30-3)

在部署本地容器镜像时设置为 `dev.local`

Knative 控制器将图像标签解析为摘要，以确保修订版本的不可变性。 当使用普通注册表时，这很有效； 但是，当与 minikube 和本地图像一起使用时，可能会导致问题。

默认情况下，Knative 控制器会跳过以 `dev.local` 或 `ko.local` 为前缀的图像的摘要。 如果您在 minikube 中运行此示例，则必须将注册表属性设置为其中的任何一个选项，以使 Knative 找到要部署的图像。

要生成 Kubernetes 资源，请在新终端中执行 `./mvnw package`。 然后，在 *target* 目录中构建工具生成的常规文件中，会在 *target/kubernetes* 目录内创建两个名为 *knative.json* 和 *knative.yaml* 的新文件，其中包含 Knative 服务定义：

```java
{
  "apiVersion" : "v1",
  "kind" : "ServiceAccount",
  "metadata" : {
    "annotations" : {
      "app.quarkus.io/vcs-url" :
       "https://github.com/lordofthejars/quarkus-cookbook.git",
      "app.quarkus.io/build-timestamp" : "2020-03-10 - 22:55:08 +0000",
      "app.quarkus.io/commit-id" : "17b19a409c41cc933770b20009f635a65f69440e"
    },
    "labels" : {
      "app.kubernetes.io/name" : "greeting-knative",
      "app.kubernetes.io/version" : "1.0-SNAPSHOT"
    },
    "name" : "greeting-knative"
  }
}{
  "apiVersion" : "serving.knative.dev/v1alpha1",
  "kind" : "Service",
  "metadata" : {
    "annotations" : {
      "app.quarkus.io/vcs-url" :
        "https://github.com/lordofthejars/quarkus-cookbook.git",
      "app.quarkus.io/build-timestamp" : "2020-03-10 - 22:55:08 +0000",
      "app.quarkus.io/commit-id" : "17b19a409c41cc933770b20009f635a65f69440e"
    },
    "labels" : {
      "app.kubernetes.io/name" : "greeting-knative",
      "app.kubernetes.io/version" : "1.0-SNAPSHOT"
    },
    "name" : "greeting-knative"
  },
  "spec" : {
    "runLatest" : {
      "configuration" : {
        "revisionTemplate" : {
          "spec" : {
            "container" : {
              "image" :"dev.local/lordofthejars/greeting-knative:1.0-SNAPSHOT",
              "imagePullPolicy" : "IfNotPresent"
            }
          }
        }
      }
    }
  }
}
```

然后部署生成的 Knative 服务：

```java
kubectl apply -f target/kubernetes/knative.json

serviceaccount/greeting-knative created
service.serving.knative.dev/greeting-knative created

kubectl get ksvc
NAME               URL                                                \
greeting-knative   http://greeting-knative.default.127.0.0.1.nip.io   \

LATESTCREATED            LATESTREADY              READY   REASON
greeting-knative-j8n76   greeting-knative-j8n76   True
```

从 `Unknown` 状态移动到 `True` 状态可能需要几秒钟的时间。 如果出现失败，也就是 `ready` 状态仍然为 `false`，您可以通过运行以下命令来检查原因和事件序列：

```java
kubectl get events --sort-by=.metadata.creationTimestamp
```

要测试服务是否已正确部署，请打开一个新的终端窗口，并在本地机器和 Knative 网关之间进行端口转发：

```java
kubectl port-forward --namespace kourier-system $(kubectl get pod \
 -n kourier-system -l "app=3scale-kourier-gateway" \
 --output=jsonpath="{.items[0].metadata.name}") \
 8080:8080 19000:19000 8443:8443

Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
Forwarding from 127.0.0.1:19000 -> 19000
Forwarding from [::1]:19000 -> 19000
Forwarding from 127.0.0.1:8443 -> 8443
Forwarding from [::1]:8443 -> 8443
Handling connection for 8080
Handling connection for 8080
```

注意，这仅在服务部署在 minikube 中时才需要。 根据您部署服务的 Kubernetes 平台的不同，您可能需要执行不同的操作。

最后，您可以向服务发送请求：

```java
curl -v -H "Host: greeting-knative.default.127.0.0.1.nip.io" \
 http://localhost:8080/greeting

hello
```

要取消部署示例，您需要运行以下命令：

```java
kubectl delete -f target/kubernetes/knative.json

serviceaccount "greeting-knative" deleted
service.serving.knative.dev "greeting-knative" deleted
```

## 讨论

您可以将 `container-image` 和 `kubernetes` 扩展组合起来，自动构建容器镜像并将其推送到 Kubernetes，如配方 10.6 所示，因此无需手动步骤。

## 参见

要了解更多，请访问以下网页：

+   [Knative Serving](https://oreil.ly/RBv52)

+   [GitHub: Kourier](https://oreil.ly/3bSDL)
