# 第四章\. 基于 Kubernetes 的软件开发平台

在前一章中，我们概述了我们围绕现代化的方法论以及设计和开发现代架构所需的步骤。我们描述了像 Kubernetes 这样的平台的必要性，它可以帮助您满足将应用程序云原生化的需求，以便根据业务需求比例进行扩展。

我们还展示了通常使用容器技术实现的微服务架构，这使得应用程序具有可移植性和一致性。现在让我们详细看看 Kubernetes 如何帮助我们现代化我们的 Java 应用程序，并通过其声明性方法和丰富的 API 集实现这一目标的步骤。

# 开发人员和 Kubernetes

[Kubernetes](https://kubernetes.io)，在希腊语中翻译为“领航员”或“统治者”，是当前现代架构的事实标准目标环境和最流行的容器编排平台；简单示意图见图 4-1。始于 Google 2015 年在管理其软件堆栈的分布式复杂应用程序方面的经验，今天它是最大的开源社区之一；由云原生计算基金会（CNCF）管理，并受到供应商和个人贡献者的欢迎。

![一个运行在节点上的 Kubernetes 集群](img/moej_0401.png)

###### 图 4-1\. 一个运行在节点上的 Kubernetes 集群

作为一个容器编排平台，其主要关注点是确保我们的应用程序正确运行，提供开箱即用的自愈、恢复功能以及强大的 API 来控制这一机制。您现在可能会问：作为开发人员，如果 Kubernetes 如此自足，我为什么要关心它呢？

这是一个很好的问题，也许一个好的答案是类比：您有一辆配备自动驾驶的一级方程式赛车，但如果您想赢得比赛，您需要调整和设置您的车辆以与其他优秀车辆竞争。对于您的应用程序而言也是如此，它可以受益于平台提供的所有功能，使其以最佳方式运行。

## Kubernetes 的功能

当您将 Kubernetes 作为目标平台来运行您的应用程序时，您可以依赖于一套 API 和组件生态系统，以使部署更加简单，使开发人员只需关注最重要的部分：编码。Kubernetes 为您提供了[一个可靠运行分布式系统的框架](https://oreil.ly/DNRQS)。

实际操作中，这意味着当涉及到以下情况时，您无需重新实现定制解决方案：

服务发现

Kubernetes 使用内部 DNS 解析来公开您的应用程序；这是自动分配的，也可以用于将流量发送到您应用程序的多个实例。

负载均衡

Kubernetes 负责管理应用程序的负载、平衡流量并相应地分发用户请求。

自愈能力

Kubernetes 可自动发现并替换失败的容器，自带健康检查和自愈机制。

部署和回滚

Kubernetes 确保您的应用程序始终以所需状态一致运行，提供控制以扩展和缩减工作负载。此外，它还提供了滚动部署或回滚到应用程序特定版本的能力。

## Kubernetes 不做什么

开发者通常在生产环境中需要处理的许多头痛问题已经被解决并委托给一个平台，其主要目标是确保应用程序正常运行。但这是否提供了您现代化应用程序所需的一切呢？可能不。

正如我们在前一章讨论的，朝着云原生方法的现代化步骤更加紧密地与方法论而非特定技术联系在一起。一旦您将思维转变为构建微服务而不是单片应用，我们就可以开始有远大眼光地思考了。如今，许多应用程序在面向 Kubernetes 的云平台上运行，并且这些应用程序正在运行具有全球覆盖范围的工作负载。以下是一些需要考虑的事项：

+   Kubernetes 不知道如何处理您的应用程序。如果应用程序失败，它可以重新启动，但无法理解失败原因，因此我们需要确保我们完全控制基于微服务的架构，并能够调试每个容器。这在大规模部署的情况下尤为重要。

+   Kubernetes 不提供任何中间件或应用程序级服务。需要通过与 Kubernetes API 交互或依赖于 Kubernetes 之上的某些服务（如服务网格框架）来解决细粒度发现服务。它没有开箱即用的开发者生态系统。

+   Kubernetes 不会构建您的应用程序。您需要负责将您的应用程序编译打包为容器镜像或依赖于 Kubernetes 之上的其他组件。

有了这些想法，让我们开始深入探讨开发者在 Kubernetes 旅程中的步骤，以便为将我们的应用程序带入下一个云原生生产环境迈出第一步。

## 基础设施即代码

Kubernetes 提供一组 API 来管理我们应用程序的期望状态以及整个平台。Kubernetes 中的每个组件都有一个可以使用的 API 表示。Kubernetes 提供了一种[声明式部署模式](https://oreil.ly/cURvG)，允许您自动化执行一组 Pod 的升级和回滚过程。声明式方法是细粒度的，也用于通过*自定义资源*概念扩展 Kubernetes API。

###### 注意

自定义资源是 Kubernetes API 的扩展。*自定义资源*代表对特定 Kubernetes 安装的定制化，引入了额外的对象以扩展集群功能。您可以从官方 Kubernetes [文档](https://oreil.ly/cVBnl)获取更多信息。

应用程序在 Kubernetes 中管理的一些核心对象包括：

Pod

一组一个或多个容器部署到 Kubernetes 集群中。这是 Kubernetes 管理和编排的实体，因此任何打包为容器的应用程序都需要声明为 Pod。

Service

负责服务发现和负载均衡的资源。为了使任何 Pod 可被发现和使用，它需要映射到一个 Service。

部署

这允许描述应用程序的生命周期，驱动 Pod 的创建，确定使用哪些镜像来构建应用程序，应该有多少个 Pod，以及它们应该如何更新。此外，它还有助于定义健康检查和约束应用程序的资源。

每个这些对象以及集群中的所有其他资源，可以通过 YAML 表示或通过 Kubernetes API 定义和控制。还有其他有用的 API 对象，如与存储相关的（PersistentVolume）或专门用于管理有状态应用程序的对象（StatefulSet）。在本章中，我们将重点关注在 Kubernetes 平台上使您的应用程序启动和运行所需的基本对象。

# 容器映像

在这段旅程中，您的第一步是将您的微服务容器化，以便它们可以作为 Pod 部署到 Kubernetes 中，这由使用 YAML 文件，调用 API 或使用 Kubernetes Java 客户端控制。

您可以使用 Coolstore 的 Inventory Quarkus 微服务作为示例来创建您的第一个容器映像。容器由一个称为 Dockerfile 或 Containerfile 的清单定义，您将在其中定义您的软件堆栈作为一个层，从操作系统层到应用程序二进制层。这种方法的好处是多方面的：易于跟踪版本，继承现有层，添加层和扩展容器。图层的图示如 图 4-2 所示。

![容器映像图层](img/moej_0402.png)

###### 图 4-2\. 容器映像图层

## Dockerfile

对于简单的用例来说，编写 Dockerfile 来将我们的应用程序打包为容器非常简单。有一些基本指令称为 *Instructions*，如在 表 4-1 中列出的那些。

表 4-1\. Dockerfile 指令

| 指令 | 描述 |
| --- | --- |
| `FROM` | 用于继承基础镜像。例如，可以是像`fedora`、`centos`、`rhel`、`ubuntu`这样的 Linux 发行版。 |
| `ENV` | 为容器使用环境变量。这些变量对应用程序可见，并且可以在运行时设置。 |
| `RUN` | 在当前层中执行命令，如安装软件包或执行应用程序。 |
| `ADD` | 将文件从工作站复制到容器层，如 JAR 文件或配置文件。 |
| `EXPOSE` | 如果你的应用程序监听某个端口，可以将其暴露给容器网络，以便 Kubernetes 将其映射到一个 Pod 和一个 Service。 |
| `CMD` | 启动应用程序的命令：容器镜像构建过程的最后一步，其中您在各层中拥有所有依赖项，可以安全运行应用程序。 |

从您的 Dockerfile 创建容器的过程也在图 4-3 中描述。

![构建容器镜像](img/moej_0403.png)

###### 图 4-3\. 构建容器镜像

在第二章中创建的库存 Quarkus Java 微服务的 Dockerfile 示例如下，并且您可以在本[书的 GitHub 仓库](https://oreil.ly/D9u1k)中找到它：

```java
FROM registry.access.redhat.com/ubi8/openjdk-11 ![1](img/1.png) ENV PROFILE=prod ![2](img/2.png) ADD target/*.jar app.jar ![3](img/3.png) EXPOSE 8080 ![4](img/4.png) CMD java -jar app.jar ![5](img/5.png)
```

![1](img/#co_a_kubernetes_based_software_development_platform_CO1-1)

我们从 OpenJDK 11 层开始构建我们的容器镜像。

![2](img/#co_a_kubernetes_based_software_development_platform_CO1-2)

设置一个环境变量，在应用程序内部可以使用它来区分不同的配置文件或配置项。

![3](img/#co_a_kubernetes_based_software_development_platform_CO1-3)

复制在编译过程中构建的 JAR 工件到容器镜像中。 这假设您已经编译了一个包含所有依赖项的“fat-jar”或“uber-jar”，它们在同一个 JAR 文件中。

![4](img/#co_a_kubernetes_based_software_development_platform_CO1-4)

将端口 8080 映射到容器网络。

![5](img/#co_a_kubernetes_based_software_development_platform_CO1-5)

运行应用程序以调用我们复制到层中的工件。

在本节中，我们定义了一个 Dockerfile，其中包含用于构建容器镜像的最小指令集。 现在让我们看看如何从 Dockerfile 创建容器镜像。

## 构建容器镜像

现在您需要创建容器镜像。 [Docker](https://www.docker.com) 是一个流行的开源项目，用于创建容器；您可以下载适用于您操作系统的版本，并开始使用它来构建和运行容器。 [Podman](https://podman.io) 是另一个开源替代方案，也可以生成 Kubernetes 对象。

当您的工作站上安装了 Docker 或 Podman 后，您可以使用以下命令从 Dockerfile 开始构建您的容器：

```java
docker build -f Dockerfile -t docker.io/modernizingjavaappsbook/
  inventory-quarkus:latest
```

这将通过读取 Dockerfile 中的指令来生成您的容器镜像。 然后，它将标记您的容器镜像为 `<repository>/<name>:<tag>` 的形式，例如 `docker.io/modernizingjavaappsbook/inventory-quarkus:latest`。 您将看到类似于此的输出：

```java
STEP 1: FROM registry.access.redhat.com/ubi8/openjdk-11
Getting image source signatures
Copying blob 57562f1b61a7 done
Copying blob a6b97b4963f5 done
Copying blob 13948a011eec done
Copying config 5d89ab50b9 done
Writing manifest to image destination
Storing signatures
STEP 2: ENV PROFILE=prod
STEP 3: ADD target/*.jar app.jar
STEP 4: EXPOSE 8080
STEP 5: CMD java -jar app.jar
STEP 6: COMMIT inventory-quarkus:latest
Getting image source signatures
Copying blob 3aa55ff7bca1 skipped: already exists
Copying blob 00af10937683 skipped: already exists
Copying blob 7f08faf4d929 skipped: already exists
Copying blob 1ab317e3c719 done
Copying config b2ae304e3c done
Writing manifest to image destination
Storing signatures
--> b2ae304e3c5
b2ae304e3c57216e42b11d8be9941dc8411e98df13076598815d7bc376afb7a1
```

您的容器镜像现在存储在 Docker 或 Podman 的本地存储中，称为*Docker 缓存*或*容器缓存*，已准备好在本地使用。

###### 注意

您可以使用此命令为库存服务创建一个生产用的 Uber-Jar：`./mvnw package -Dquarkus.profile=prod`。 您可以让 Docker 或 Podman 编译您的软件，并使用称为[多阶段](https://oreil.ly/HzhDj)的特定类型容器镜像构建来创建容器。 查看[此 Dockerfile](https://oreil.ly/UK3c2) 作为示例。

## 运行容器

*运行容器* 指的是从容器缓存中拉取容器镜像以运行应用程序。这个过程将由容器运行时（如 Docker 或 Podman）从我们工作站中的其他进程隔离开来，提供了一个便携式的应用程序，所有依赖项都在容器内管理，而不是在我们的工作站上。

要开始测试打包为容器镜像的 Inventory 微服务，您可以运行以下命令：

```java
docker run -ti docker.io/modernizingjavaappsbook/inventory-quarkus:latest
```

Quarkus 微服务已在容器中启动并运行，监听 8080 端口。Docker 或 Podman 负责将容器网络映射到您的工作站；在[*http://localhost:8080*](http://localhost:8080) 打开浏览器，您将看到 Quarkus 欢迎页面（如图 2-4 所示）。

###### 提示

[Docker 网络文档](https://oreil.ly/ja9Iu) 包含了关于如何在运行 Docker 的容器和主机内映射端口和网络的更多信息。

## 注册表

正如我们在前一节中所描述的，容器镜像存储在本地缓存中。然而，如果您想要将它们在工作站外部提供，您需要以某种便捷的方式发送它们。容器镜像的大小通常为几百兆字节。这就是为什么您需要一个容器镜像注册表。

注册表基本上充当存储容器镜像并通过上传（推送）和下载（拉取）分享它们的地方。一旦镜像位于另一个系统上，其中包含的原始应用程序也可以在该系统上运行。

注册表可以是公共的或私有的。流行的公共注册表包括[Docker Hub](https://hub.docker.com)和[Quay.io](https://quay.io)。它们作为互联网上的 SaaS 提供，并允许以公开或不需要身份验证的方式提供镜像。私有注册表通常专为特定用户而设，并且不能公开使用。然而，您可以将它们提供给私有环境，例如私有 Kubernetes 集群。

在这个例子中，我们在 DockerHub 上为这本书创建了一个名为 `modernizingjavaappsbook` 的组织，它映射到这个公共注册表的一个仓库，我们希望将我们的容器镜像推送到这里。

首先，您需要登录到注册表。您需要进行身份验证以能够推送新内容，然后您将使得容器镜像公开可用：

```java
docker login docker.io
```

成功登录后，您可以开始将 Inventory 容器镜像上传到注册表：

```java
docker push docker.io/modernizingjavaappsbook/inventory-quarkus:latest
```

此命令将镜像推送到注册表，并且您应该得到类似以下内容的输出作为确认：

```java
Getting image source signatures
Copying blob 7f08faf4d929 done
Copying blob 1ab317e3c719 done
Copying blob 3aa55ff7bca1 skipped: already exists
Copying blob 00af10937683 skipped: already exists
Copying config b2ae304e3c done
Writing manifest to image destination
Storing signatures
```

已打包为容器镜像的 Quarkus 微服务现已准备好在任何地方部署！

# 部署到 Kubernetes

通过与 Kubernetes API 交互来部署应用程序，以创建代表应用程序所需状态的对象，这是在 Kubernetes 集群中完成的。正如我们讨论的那样，Pod、Service 和 Deployment 是在 Kubernetes 管理整个应用程序生命周期和连接性所创建的最小对象。

###### 注意

如果您尚未拥有 Kubernetes 集群，可以下载并使用 [minikube](https://oreil.ly/n2Kgx)，这是一个专为本地开发设计的独立 Kubernetes 集群。

Kubernetes 中的每个对象都包含以下值：

apiVersion

用于创建此对象的 Kubernetes API 版本

kind

对象类型（例如 Pod、Service）

metadata

有助于唯一标识对象的信息片段，如名称或 UID

spec

对象的期望状态

在本节中，我们定义了任何 Kubernetes 对象的基本结构。现在，让我们探讨在 Kubernetes 上运行应用程序所需的基本对象。

## Pod

[Pod](https://oreil.ly/KQk2T) 是一个或多个容器的组合，共享存储和网络资源，并具有运行容器的规范。在 图 4-4 中，您可以看到 Kubernetes 集群中两个 Pod 的表示，每个 Pod 都分配了 Kubernetes 分配的示例 IP 地址。

![Pod 和容器](img/moej_0404.png)

###### 图 4-4\. Pod 和容器

Kubernetes 不直接与容器一起工作；它依赖 Pod 概念来编排容器。因此，您需要提供与您的容器匹配的 Pod 定义：

```java
apiVersion: v1
kind: Pod
metadata:
  name: inventory-quarkus ![1](img/1.png)
  labels:
    app: inventory-quarkus ![2](img/2.png)
spec:
  containers: ![3](img/3.png)
    - name: inventory-quarkus
      image: docker.io/modernizingjavaappsbook/inventory-quarkus:latest ![4](img/4.png)
      ports:
        - containerPort: 8080 ![5](img/5.png)
```

![1](img/#co_a_kubernetes_based_software_development_platform_CO2-1)

Pod 对象的名称，在每个命名空间中唯一

![2](img/#co_a_kubernetes_based_software_development_platform_CO2-2)

一组要应用于此对象的键/值对

![3](img/#co_a_kubernetes_based_software_development_platform_CO2-3)

用于此 Pod 中的容器列表

![4](img/#co_a_kubernetes_based_software_development_platform_CO2-4)

容器镜像 URI，在本例中是 Docker Hub 上公开可用的存储库

![5](img/#co_a_kubernetes_based_software_development_platform_CO2-5)

由此容器公开的端口，要映射到 Pod 中

###### 提示

通常，一个 Pod 包含一个容器，因此映射为 1 Pod：1 应用程序。尽管对于某些用例（例如边车），您可以在一个 Pod 中有多个容器，但最佳实践是将 1 Pod 映射到 1 个应用程序，因为这确保了可扩展性和可维护性。

您可以将前面描述的任何 Kubernetes 对象作为 YAML 文件使用 Kubernetes CLI `kubectl` 创建。运行下面显示的命令，部署您的第一个微服务作为单个 Pod。您可以在本书的 [GitHub 代码库](https://oreil.ly/YF8bT) 中找到它。

```java
kubectl create -f pod.yaml
```

要检查它是否在 Kubernetes 上运行：

```java
kubectl get pods
```

您应该会得到类似以下输出：

```java
NAME               READY  STATUS   RESTARTS  AGE
inventory-quarkus  1/1    Running  0         30s
```

如果查看 `STATUS` 列，它显示 Pod 正确运行，并且所有默认健康检查都正确满足。

###### 提示

如果你想进一步了解如何进行更精细的健康检查，请参考官方 Kubernetes 文档中的 [活跃检测和准备检测](https://oreil.ly/sOdnL)。

## 服务

[Kubernetes 服务](https://oreil.ly/fOfpN)用于暴露运行在一组 Pod 上的应用程序。这很有用，因为 Pod 从 Kubernetes 网络中获得随机 IP 地址，如果重新启动或移动到 Kubernetes 集群中的另一个节点，该地址可能会更改。服务提供了一种更一致的方式与 Pod 进行通信，充当 DNS 服务器和负载均衡器。

一个服务映射到一个或多个 Pod；它使用内部 DNS 解析到一个来自助记短主机名（例如 `inventory-quarkus`）的内部 IP，并根据 图 4-5 中显示的方式平衡流量。每个服务从专用 IP 地址范围中获得自己的 IP 地址，这与 Pod 的 IP 地址范围不同。

![Kubernetes 服务](img/moej_0405.png)

###### 图 4-5\. Kubernetes 服务

###### 注意

Kubernetes 服务提供的负载均衡方法是第 4 层（TCP/UDP）。唯一可用的两种策略是轮询和源 IP。对于应用层负载均衡（例如 HTTP），还有其他对象如 `Ingress`，本书未涵盖，但你可以在这里找到它们的文档 [here](https://oreil.ly/VBvOu)。

让我们看一下可能映射我们 Pod 的服务：

```java
apiVersion: v1
kind: Service
metadata:
  name: inventory-quarkus-service ![1](img/1.png)
spec:
  selector:
    app: inventory-quarkus ![2](img/2.png)
  ports:
    - protocol: TCP ![3](img/3.png)
      port: 8080 ![4](img/4.png)
      targetPort: 8080 ![5](img/5.png)
```

![1](img/#co_a_kubernetes_based_software_development_platform_CO3-1)

服务对象的名称

![2](img/#co_a_kubernetes_based_software_development_platform_CO3-2)

Pod 暴露的标签，用于匹配服务

![3](img/#co_a_kubernetes_based_software_development_platform_CO3-3)

使用的 L4 协议，TCP 或 UDP

![4](img/#co_a_kubernetes_based_software_development_platform_CO3-4)

这个服务使用的端口

![5](img/#co_a_kubernetes_based_software_development_platform_CO3-5)

Pod 使用的端口，并映射到服务中

要创建你的服务，请按照下面显示的命令运行。你也可以在这本书的 [GitHub 仓库](https://oreil.ly/e13Dd) 中找到它。

```java
kubectl create -f service.yaml
```

在 Kubernetes 上检查它是否在运行：

```java
kubectl get svc
```

你应该得到类似如下的输出：

```java
NAME                       TYPE       CLUSTER-IP    EXTERNAL-IP  PORT(S)   AGE
inventory-quarkus-service  ClusterIP  172.30.34.73  <none>       8080/TCP  6s
```

###### 注意

你刚刚定义了一个服务，映射到一个 Pod。这只能从内部 Kubernetes 网络访问，除非你使用像 Ingress 这样的对象来接受来自集群外部的流量，使其暴露出去。

## 部署

部署是用于管理应用程序生命周期的 Kubernetes 对象。部署描述了一个期望的状态，Kubernetes 将使用“滚动”或“重新创建”部署策略来实现它。部署的部署生命周期包括进行中、完成和失败状态。当部署正在执行更新任务（例如更新或扩展 Pod）时，部署处于进行中状态。

Kubernetes 部署在基本 Pod 和服务概念之上提供了一组功能，如下列出的内容和 图 4-6 中显示的内容：

+   部署 ReplicaSet 或 Pod

+   更新 Pods 和 ReplicaSets

+   回滚到先前的部署版本

+   扩展一个部署

+   暂停或继续一个部署

+   定义健康检查

+   定义资源约束

![部署管理应用程序的生命周期和更新](img/moej_0406.png)

###### 图 4-6\. 部署管理应用程序的生命周期和更新

使用 Kubernetes 部署管理应用程序包括应用程序应该如何更新的方式。部署的一个主要优势是能够可预测地启动和停止一组 Pods。在 Kubernetes 中部署应用程序有两种策略：

滚动更新

它提供了对应用程序的 Pod 的可控、分阶段的替换，确保始终有一定数量的 Pod 可用。这对于应用程序的业务连续性非常有用，直到部署的所需数量的 Pod 的健康检查（探针）得到满足，流量才会被路由到应用程序的新版本中。

重新创建

它会在创建新的 Pods 之前移除所有现有的 Pods。Kubernetes 首先终止当前版本的所有容器，然后在旧容器消失时同时启动所有新容器。这会为应用程序带来停机时间，但它确保没有多个版本同时运行。

在下面的示例中列出了在 Kubernetes 上驱动 Pods 部署的`Deployment`对象：

```java
apiVersion: apps/v1
kind: Deployment
metadata:
  name: inventory-quarkus-deploy ![1](img/1.png)
  labels:
    app: inventory-quarkus ![2](img/2.png)
spec:
  replicas: 1 ![3](img/3.png)
  selector:
    matchLabels:
      app: inventory-quarkus ![4](img/4.png)
  template: ![5](img/5.png)
    metadata:
      labels:
        app: inventory-quarkus
    spec:
      containers:
      - name: inventory-quarkus
        image: docker.io/modernizingjavaappsbook/inventory-quarkus:latest ![6](img/6.png)
        ports:
        - containerPort: 8080
        readinessProbe: ![7](img/7.png)
          httpGet:
            path: /
            port: 8080
            scheme: HTTP
          periodSeconds: 10
          successThreshold: 1
          failureThreshold: 3
        livenessProbe: ![8](img/8.png)
          httpGet:
            path: /
            port: 8080
            scheme: HTTP
          periodSeconds: 10
          successThreshold: 1
          failureThreshold: 3
```

![1](img/#co_a_kubernetes_based_software_development_platform_CO4-1)

部署对象的名称。

![2](img/#co_a_kubernetes_based_software_development_platform_CO4-2)

此对象的标签。

![3](img/#co_a_kubernetes_based_software_development_platform_CO4-3)

Pod 副本的期望数量。

![4](img/#co_a_kubernetes_based_software_development_platform_CO4-4)

使用标签找到要管理的 Pods 的选择器。

![5](img/#co_a_kubernetes_based_software_development_platform_CO4-5)

要使用的 Pod 模板，包括要继承的标签或要创建的容器。

![6](img/#co_a_kubernetes_based_software_development_platform_CO4-6)

要使用的容器镜像。

![7](img/#co_a_kubernetes_based_software_development_platform_CO4-7)

Kubernetes 使用就绪探针来知道何时容器已准备好开始接受流量，当其所有容器都准备好时，Pod 被视为已准备好。在这里，我们将根路径上的 HTTP 健康检查定义为就绪探针。

![8](img/#co_a_kubernetes_based_software_development_platform_CO4-8)

Kubernetes 使用存活探针来知道何时重新启动容器。在这里，我们将根路径上的 HTTP 健康检查定义为存活探针。

运行以下命令来创建您的 Deployment。您也可以在本书的 [GitHub 存储库](https://oreil.ly/PWucG) 中找到它：

```java
kubectl create -f deployment.yaml
```

运行以下命令来验证已创建 Deployment，并获取其状态：

```java
kubectl get deploy
```

您应该得到类似于以下输出：

```java
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
inventory-quarkus-deploy   1/1     1            1           10s
```

查看 `READY` 列，您的期望状态已正确匹配，请求在 Kubernetes 上运行的 Inventory 微服务的一个副本。您可以交叉检查是否已创建 Pod：

```java
kubectl get pods
```

您应该会得到类似的输出：

```java
NAME                                        READY   STATUS    RESTARTS   AGE
inventory-quarkus                           1/1     Running   0          1m
inventory-quarkus-deploy-5cb46f5d8d-fskpd   1/1     Running   0          30s
```

现在使用随机生成的名称创建了一个新的 Pod，从 `inventory-quarkus-deploy` 部署名称开始。如果应用程序崩溃或我们杀死由部署管理的 Pod，则 Kubernetes 将自动重新创建它。这对没有部署的 Pod 并不适用：

```java
kubectl delete pod inventory-quarkus inventory-quarkus-deploy-5cb46f5d8d-fskpd
```

您可以看到始终满足期望状态：

```java
kubectl get pods
```

您应该会得到类似于以下的输出：

```java
NAME                                        READY   STATUS    RESTARTS   AGE
inventory-quarkus-deploy-5cb46f5d8d-llp7n   1/1     Running   0          42s
```

# Kubernetes 和 Java

Kubernetes 具有管理应用程序生命周期的巨大潜力，有许多研究关于开发人员和架构师如何最好地适应其架构，例如模式。Kubernetes 模式是针对基于容器的应用程序和服务的可重用设计模式。

从 Java 开发者的角度来看，第一步是从单块应用程序方法迁移到基于微服务的方法。完成后，下一步是进入 Kubernetes 环境，并最大化该平台提供的优势：API 可扩展性，声明式模型以及 IT 行业正在收敛的标准化流程。

有一些 Java 框架可以帮助开发人员连接到 Kubernetes 并将其应用程序转换为容器。您已经使用 Dockerfile 对 `Inventory` Quarkus 微服务进行了容器化。现在让我们从 Java 驱动这种容器化过程，使用 Maven 和 Gradle 为 `Catalog` Spring Boot 微服务生成一个容器镜像。

## Jib

[Jib](https://oreil.ly/N2jRr) 是由 Google 开发的开源框架，用于构建符合开放容器倡议（OCI）镜像格式的容器镜像，无需 Docker 或任何容器运行时。您甚至可以从您的 Java 代码库中创建容器，因为它为此提供了 Maven 和 Gradle 插件。这意味着 Java 开发人员可以在不编写和/或维护任何 Dockerfile 的情况下容器化其应用程序，将这种复杂性委托给 Jib。

我们看到从这种方法中获得的[好处](https://oreil.ly/2y92D)如下：

纯 Java

无需 Docker 或 Dockerfile 知识；只需将 Jib 添加为插件，它将为您生成容器镜像。由此产生的镜像通常被称为“distroless”，因为它不继承任何基础镜像。

速度

应用程序被分成多个层，从类中分离依赖项。无需像 Dockerfile 那样重新构建容器镜像；Jib 负责部署更改的层。

可复现性

不会触发不必要的更新，因为相同的内容始终生成相同的镜像。

使用命令行添加插件，以现有 Maven 快速启动使用 Jib 构建容器镜像的最简单方法是：

```java
mvn compile com.google.cloud.tools:jib-maven-plugin:2.8.0:build
  -Dimage=<MY IMAGE>
```

或者，您可以通过将 Jib 添加为 *pom.xml* 中的插件来完成：

```java
<project>
  ...
  <build>
    <plugins>
      ...
      <plugin>
        <groupId>com.google.cloud.tools</groupId>
        <artifactId>jib-maven-plugin</artifactId>
        <version>2.8.0</version>
        <configuration>
          <to>
            <image>myimage</image>
          </to>
        </configuration>
      </plugin>
      ...
    </plugins>
  </build>
  ...
</project>
```

通过这种方式，你也可以管理其他设置，比如身份验证或构建的参数。如果你想要构建 Catalog 服务并直接推送到 Docker Hub，请运行下面的命令：

```java
mvn compile com.google.cloud.tools:jib-maven-plugin:2.8.0:build↳
-Dimage=docker.io/modernizingjavaappsbook/catalog-spring-boot:latest↳
-Djib.to.auth.username=<USERNAME>↳
-Djib.to.auth.password=<PASSWORD>
```

这里的身份验证是通过命令行选项管理的，但 Jib 能够使用 Docker CLI 管理现有的身份验证，或者从你的 *settings.xml* 读取凭据。

构建需要一些时间，结果是一个本地构建并直接推送到仓库的 distroless 容器镜像，在这种情况下是 Docker Hub：

```java
[INFO] Scanning for projects...
[INFO]
[INFO] -------------------< com.redhat.cloudnative:catalog >-------------------
[INFO] Building CoolStore Catalog Service 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ catalog ---
[INFO] Copying 4 resources
[INFO]
[INFO] --- maven-compiler-plugin:3.6.1:compile (default-compile) @ catalog ---
[INFO] Nothing to compile - all classes are up to date
[INFO]
[INFO] --- jib-maven-plugin:2.8.0:build (default-cli) @ catalog ---
[INFO]
[INFO] Containerizing application to modernizingjavaappsbook/catalog-spring-boot
  ...
[WARNING] Base image 'gcr.io/distroless/java:11' does not use a specific image
  digest↳ - build may not be reproducible
[INFO] Using credentials from <to><auth> for modernizingjavaappsbook/
  catalog-spring-boot
[INFO] Using base image with digest:↳
sha256:65aa73135827584754f1f1949c59c3e49f1fed6c35a918fadba8b4638ebc9c5d
[INFO]
[INFO] Container entrypoint set to [java, -cp, /app/resources:/app/classes:/app/
  libs/*, com.redhat.cloudnative.catalog.CatalogApplication]
[INFO]
[INFO] Built and pushed image as modernizingjavaappsbook/catalog-spring-boot
[INFO] Executing tasks:
[INFO] [==============================] 100,0% complete
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  27.817 s
[INFO] Finished at: 2021-03-19T11:48:16+01:00
[INFO] ------------------------------------------------------------------------
```

###### 注意

由于你不需要任何容器运行时来使用 Jib 构建镜像，因此你的容器镜像不会出现在本地缓存中。你不会在 `docker images` 命令中看到它，但之后你可以从 Docker Hub 拉取它并存储在你的缓存中。如果你也想从一开始就将其存储在本地，Jib 也可以连接到 Docker 主机并为你执行此操作。

## JKube

[Eclipse JKube](https://oreil.ly/Km2ci) 是由 Eclipse 基金会和红帽支持的社区项目，是另一个帮助 Java 开发人员与 Kubernetes 交互的开源 Java 框架。它支持使用 Docker/Podman、Jib 和 Source-to-Image (S2I) 构建容器镜像。Eclipse JKube 还提供了一组工具，可自动部署到 Kubernetes 并管理应用程序，其中包括调试和日志记录的助手。它来自 Fabric8 Maven 插件，经过重新品牌和增强，以目标 Kubernetes 的项目形式发布。

###### 提示

JKube 支持 Kubernetes 和 OpenShift。OpenShift 在 Kubernetes 之上提供 [Source-to-Image](https://oreil.ly/4z2Zn) ，这是一种从源代码自动编译容器镜像的机制。这样一来，构建就在 Kubernetes 上进行，因此开发人员可以直接在目标平台上测试和部署他们的应用。

与 Jib 一样，JKube 提供了零配置模式，用于快速上手，其中会预先选择有见地的默认值。它提供内联配置，在插件配置中使用 XML 语法。此外，它提供了真实部署描述符的外部配置模板，这些模板会由插件增强。

JKube 提供三种形式：

Kubernetes 插件

它可以在任何 Kubernetes 集群中使用，提供 distroless 或 Dockerfile 驱动的构建。

OpenShift 插件

它可以在任何 Kubernetes 或 OpenShift 集群中使用，提供 distroless、Dockerfile 驱动的构建，或者 Source-to-Image (S2I) 构建。

JKube 工具包

一个与 JKube Core 交互的工具包和命令行工具，它也作为一个 Kubernetes 客户端，并提供了一个扩展 Kubernetes manifests 的 Enricher API。

JKube 提供了比 Jib 更多的功能；事实上，它可以被视为一个超集。你可以进行 distroless Jib 构建，但也可以使用 Dockerfile 并从 Java 部署 Kubernetes manifests。在这种情况下，我们不需要编写 Deployment 或 Service；JKube 将负责构建容器并将其部署到 Kubernetes。

让我们在我们的目录 POM 文件中包含 JKube，并配置它进行 Jib 构建和部署到 Kubernetes。这样做将使插件持久化。你也可以在这本[书的 GitHub 存储库](https://oreil.ly/Ba4Ro)中找到源代码。

首先，我们需要将 JKube 添加为插件：

```java
<project>
  ...
  <build>
    <plugins>
      ...
    <plugin>
       <groupId>org.eclipse.jkube</groupId>
       <artifactId>kubernetes-maven-plugin</artifactId>
       <version>1.1.1</version>
    </plugin>
      ...
    </plugins>
  </build>
  ...
</project>
```

之后，你可以使用属性驱动容器镜像构建。在这种情况下，你可能想要使用 Jib 来构建镜像并将其推送到 Docker Hub。然后，你将部署到 Kubernetes：

```java
...
<properties>
...
    <jkube.build.strategy>jib</jkube.build.strategy>
    <jkube.generator.name>docker.io/modernizingjavaappsbook/catalog-spring-boot:
      ${project.version}</jkube.generator.name>
</properties>
...
```

让我们构建镜像：

```java
mvn k8s:build
```

你应该会看到类似以下的输出：

```java
JIB>... modernizingjavaappsbook/catalog-spring-boot/1.0-SNAPSHOT/build/
  deployments/catalog-1.0-SNAPSHOT.jar
JIB>    :
JIB>... modernizingjavaappsbook/catalog-spring-boot/1.0-SNAPSHOT/build/Dockerfile
...
JIB> [========================      ] 80,0% complete > building image to tar file
JIB> Building image to tar file...
JIB> [========================      ] 80,0% complete > writing to tar file
JIB> [==============================] 100,0% complete
[INFO] k8s: ... modernizingjavaappsbook/catalog-spring-boot/1.0-SNAPSHOT/tmp/↳
docker-build.tar successfully built
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  36.229 s
[INFO] Finished at: 2021-03-19T13:03:19+01:00
[INFO] ------------------------------------------------------------------------
```

JKube 使用 Jib 在本地创建了容器镜像，现在已准备好推送到 Docker Hub。你可以通过以下三种方式之一指定凭据：

Docker 登录

你可以登录到你的注册表，这里是 Docker Hub，JKube 将读取 *~/.docker/config.json* 文件以获取认证详细信息。

在 POM 中提供凭据

将注册表凭据作为 XML 配置的一部分提供。

在 Maven 设置中提供凭据

你可以在你的 *~/.m2/settings.xml* 文件中提供注册表凭据，插件将从中读取。

在这种情况下，你使用第三个选项，并将凭据设置到 Maven 设置中，这样你就可以使用你的凭据复制这个文件。你也可以在这本[书的 GitHub 存储库](https://oreil.ly/uxAxW)中找到源代码：

```java
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0↳
 http://maven.apache.org/xsd/settings-1.0.0.xsd">

  <servers>
    <server>
      <id>https://index.docker.io/v1</id>
      <username>USERNAME</username>
      <password>PASSWORD</password>
    </server>
  </servers>
</settings>
```

要将其推送到 Docker Hub，只需运行此 Maven 目标：

```java
mvn k8s:push
```

你应该看到类似以下的输出：

```java
JIB> [=========================] 81,8% complete > scheduling pushing manifests
JIB> [=========================] 81,8% complete > launching manifest pushers
JIB> [=========================] 81,8% complete > pushing manifest for latest
JIB> Pushing manifest for latest...
JIB> [=========================] 90,9% complete > building images to registry
JIB> [=========================] 90,9% complete > launching manifest list pushers
JIB> [=========================] 100,0% complete
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  01:08 min
[INFO] Finished at: 2021-03-19T13:21:28+01:00
```

现在是时候在 Kubernetes 上部署目录了。JKube 将连接到你的 Kubernetes 集群，读取你工作站上的 `~/.kube/config` 文件：

```java
mvn k8s:resource k8s:apply
```

你应该会看到类似以下的输出：

```java
[INFO] Scanning for projects...
[INFO]
[INFO] -------------------< com.redhat.cloudnative:catalog >-------------------
[INFO] Building CoolStore Catalog Service 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- kubernetes-maven-plugin:1.1.1:resource (default-cli) @ catalog ---
[INFO] k8s: Running generator spring-boot
  ...
[INFO] k8s: Creating a Service from kubernetes.yml namespace default name catalog
[INFO] k8s: Created Service: target/jkube/applyJson/default/service-catalog.json
[INFO] k8s: Creating a Deployment from kubernetes.yml namespace default name
  catalog
[INFO] k8s: Created Deployment: target/jkube/applyJson/default/deployment-
  catalog.json
[INFO] k8s: HINT: Use the command `kubectl get pods -w` to watch your pods start
  up
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  7.464 s
[INFO] Finished at: 2021-03-19T13:38:27+01:00
[INFO] ------------------------------------------------------------------------
```

应用程序已成功部署到 Kubernetes，使用生成的清单：

```java
kubectl get pods
```

```java
NAME                       READY   STATUS    RESTARTS   AGE
catalog-64869588f6-fpjj8   1/1     Running   0          2m2s
```

```java
kubectl get deploy
```

```java
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
catalog   1/1     1            1           3m54s
```

为了测试它，让我们看一下 Service：

```java
kubectl get svc
```

```java
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
catalog      ClusterIP   10.99.26.127   <none>        8080/TCP   4m44s
```

###### 提示

默认情况下，Kubernetes 仅在集群内部公开应用程序，使用 `ClusterIP` 服务类型。你可以使用 `Service` 类型 `NodePort` 或使用 Ingress 在外部公开它。在这个例子中，你将使用 `kubectl port-forward` 将 Kubernetes 公开的端口映射到我们工作站的端口。

让我们使用 `kubectl port-forward` 命令尝试我们的应用程序：

```java
 kubectl port-forward deployment/catalog 8080:8080
```

如果你现在在浏览器中打开 [*http://localhost:8080/api/catalog*](http://localhost:8080/api/catalog)，你将看到 Coolstore 的目录 JSON 输出。

# 摘要

在本章中，我们讨论了 Java 开发人员如何从 Kubernetes 的能力中受益，以使其应用程序现代化并增强其功能，展示了开发人员在 Kubernetes 环境中的内部循环。我们演示了如何创建容器镜像以及如何将其部署到 Kubernetes。我们还介绍了如何通过 Maven 直接从 Java 驱动容器创建和部署，感谢 Jib 和 JKube 的支持。

现代化对于开发人员非常重要，以使应用程序云原生且可移植，为提供高可用性的生产和服务做好准备。在下一章中，我们将更深入地研究现有 Java 应用程序的现代化以及实现它所需的步骤。
