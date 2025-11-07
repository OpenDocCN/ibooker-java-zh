# 附录 A 设置你的开发环境

本附录涵盖了

+   设置 Java

+   设置 Docker

+   设置 Kubernetes

+   设置其他工具

在本附录中，你将找到设置你的开发环境以及安装我们将在整本书中用于构建、管理和部署云原生应用的工具的说明。

## A.1 Java

本书中的所有示例都是基于 Java 17，这是写作时的最新长期发布版本。你可以安装任何 OpenJDK 17 发行版。我将使用来自 Adoptium 项目 ([`adoptium.net`](https://adoptium.net)) 的 Eclipse Temurin，之前被称为 AdoptOpenJDK，但你可以自由选择另一个版本。

在你的机器上管理不同的 Java 版本和发行版可能会很痛苦。我建议使用像 sdkman ([`sdkman.io`](https://sdkman.io)) 这样的工具来轻松安装、更新和切换不同的 JDK。在 macOS 和 Linux 上，你可以按照以下方式安装 sdkman：

```
$ curl -s "https://get.sdkman.io" | bash
```

请参阅官方文档以获取 Windows 的安装说明。

安装完成后，通过运行以下命令检查所有可用的 OpenJDK 发行版和版本：

```
$ sdk list java
```

然后选择一个发行版并安装它。例如，我可以在写作时安装最新的 17 版本的 Eclipse Temurin，如下所示：

```
$ sdk install java 17.0.3-tem
```

到你阅读本节时，可能已有更新的版本可用，所以请检查列表命令返回的内容以确定最新版本。

在安装过程结束时，sdkman 将询问你是否想将该发行版设置为默认版本。我建议你回答“是”，以确保你可以从你构建的整个书中的所有项目中访问 Java 17。你可以使用以下命令随时更改默认版本：

```
$ sdk default java 17.0.3-tem
```

让我们现在验证 OpenJDK 的安装：

```
$ java --version
openjdk 17.0.3 2022-04-19
OpenJDK Runtime Environment Temurin-17.0.3+7 (build 17.0.3+7)
OpenJDK 64-Bit Server VM Temurin-17.0.3+7 (build 17.0.3+7, mixed mode)
```

你还可以选择仅在本 shell 的上下文中更改 Java 版本：

```
$ sdk use java 17.0.3-tem
```

最后，如果你想检查当前 shell 中配置的 Java 版本，你可以这样做：

```
$ sdk current java
Using java version 17.0.3-tem
```

## A.2 Docker

Open Container Initiative (OCI)，一个 Linux 基金会项目，为与容器一起工作定义了行业标准 ([`opencontainers.org`](https://opencontainers.org))。特别是，OCI 图像规范定义了如何构建容器镜像，OCI 运行时规范定义了如何运行这些容器镜像，而 OCI 发行版规范定义了如何分发它们。我们全书用于与容器一起工作的工具是 Docker，它符合 OCI 规范。

在 Docker 网站 ([www.docker.com](http://www.docker.com)) 上，你可以找到在本地环境中设置 Docker 的说明。我将使用写作时提供的最新版本：Docker 20.10 和 Docker Desktop 4.11。

+   在 Linux 上，你可以直接安装 Docker 开源平台。它也被称为 Docker 社区版（Docker CE）。

+   在 macOS 和 Windows 上，您可以选择使用 Docker Desktop，这是一个基于 Docker 构建的商业产品，它使得从这些操作系统运行 Linux 容器成为可能。在撰写本文时，Docker Desktop 对个人使用、教育、非商业开源项目和中小企业是免费的。在安装软件之前，请仔细阅读 Docker 订阅服务协议，并确保您符合其规定（[www.docker.com/legal](http://www.docker.com/legal)）。

Docker Desktop 支持 ARM64 和 AMD64 架构，这意味着您可以在配备苹果硅处理器的苹果新电脑上运行本书中的所有示例。

如果您在 Windows 上工作，Docker Desktop 提供两种类型的设置：Hyper-V 或 WSL2。我建议您选择后者，因为它提供更好的性能，并且更稳定。

Docker 默认配置为从 Docker Hub 下载 OCI 镜像，Docker Hub 是一个容器注册库，托管着许多流行开源项目的镜像，如 Ubuntu、PostgreSQL 和 Redis。使用它是免费的，但如果您匿名使用，它将受到严格的速率限制政策。因此，我建议您在 Docker 网站上创建一个免费账户（[www.docker.com](http://www.docker.com)）。

创建账户后，打开一个终端窗口，并使用 Docker Hub 进行身份验证（确保您的 Docker 引擎正在运行）。由于它是默认的容器注册库，您不需要指定其 URL：

```
$ docker login
```

当被要求时，请输入您的用户名和密码。

使用 Docker CLI，您现在可以与 Docker Hub 交互，下载镜像（*pull*）或上传您自己的（*push*）。例如，尝试从 Docker Hub 拉取官方 Ubuntu 镜像：

```
$ docker pull ubuntu:22.04
```

在本书中，您将学习更多关于使用 Docker 的知识。在此之前，如果您想尝试容器，我将为您留下一份控制容器生命周期的有用命令列表（表 A.1）。

表 A.1 管理镜像和容器的有用 Docker CLI 命令

| Docker CLI 命令 | 它的作用 |
| --- | --- |
| docker images | 显示所有镜像 |
| docker ps | 显示正在运行的容器 |
| docker ps -a | 显示所有创建、启动和停止的容器 |
| docker run <image> | 从给定镜像运行一个容器 |
| docker start <name> | 启动一个现有的容器 |
| docker stop <name> | 停止一个正在运行的容器 |
| docker logs <name> | 显示给定容器的日志 |
| docker rm <name> | 删除一个已停止的容器 |
| docker rmi <image> | 删除一个镜像 |

本书构建的所有容器都符合 OCI 标准，并将与任何其他 OCI 容器运行时一起工作，例如 Podman ([`podman.io`](https://podman.io))。如果您决定使用除 Docker 之外的平台，请注意，我们用于本地开发和集成测试的一些工具可能需要额外的配置才能正确工作。

## A.3 Kubernetes

在您的本地环境中安装 Kubernetes 有几种方法。以下是一些最常用的选项：

+   *minikube* ([`minikube.sigs.k8s.io`](https://minikube.sigs.k8s.io)) 允许您在任何操作系统上运行本地 Kubernetes 集群。它由 Kubernetes 社区维护。

+   *kind* ([`kind.sigs.k8s.io`](https://kind.sigs.k8s.io)) 允许您将本地 Kubernetes 集群作为 Docker 容器运行。它最初是为了测试 Kubernetes 本身而开发的，但您也可以用它来进行 Kubernetes 的本地开发。它由 Kubernetes 社区维护。

+   *k3d* ([`k3d.io`](https://k3d.io)) 允许您基于 Rancher Labs 实现的最小化 Kubernetes 发行版 k3s 运行本地 Kubernetes 集群。它由 Rancher 社区维护。

随意选择最适合您需求的工具。由于它的稳定性和与所有操作系统和架构的兼容性，包括新的苹果硅电脑，本书将使用 minikube。您至少需要两个 CPU 和 4 GB 的空闲内存来使用 minikube 运行本书中的所有示例。

您可以在项目的网站上找到安装指南 ([`minikube.sigs.k8s.io`](https://minikube.sigs.k8s.io))。本书将使用写作时提供的最新版本：Kubernetes 1.24 和 minikube 1.26。在 macOS 上，您可以使用 Homebrew 如下安装 minikube：

```
$ brew install minikube
```

使用 minikube 运行本地 Kubernetes 集群需要一个容器运行时或虚拟机管理器。由于我们已经在使用 Docker，所以我们将使用它。在底层，任何 minikube 集群都将作为一个 Docker 容器运行。

安装 minikube 后，您可以使用 Docker 驱动程序启动一个新的本地 Kubernetes 集群。您第一次运行此命令时，将需要几分钟来下载运行集群所需的所有组件：

```
$ minikube start --driver=docker
```

我建议通过运行以下命令将 Docker 设置为 minikube 的默认驱动程序：

```
$ minikube config set driver docker
```

要与新建的 Kubernetes 集群交互，您需要安装 kubectl，Kubernetes 的 CLI。安装说明可在官方网站 ([`kubernetes.io/docs/tasks/tools`](https://kubernetes.io/docs/tasks/tools)) 上找到。在 macOS 和 Linux 上，您可以使用 Homebrew 如下安装它：

```
$ brew install kubectl
```

然后，您可以验证 minikube 集群是否已正确启动，并检查您的本地集群中是否有一个节点正在运行：

```
$ kubectl get nodes
NAME       STATUS   ROLES                  AGE     VERSION
minikube   Ready    control-plane,master   2m20s   v1.24.3
```

我建议在不需要 minikube 时停止它，以释放您本地环境中的资源：

```
$ minikube stop
```

在本书中，您将学习更多关于使用 Kubernetes 和 minikube 的知识。在此之前，如果您想尝试 Kubernetes 资源，我将为您提供一些有用的命令（表 A.2）。

表 A.2 管理 Pods、Deployments 和 Services 的有用 Kubernetes CLI 命令

| Kubernetes CLI 命令 | 执行的操作 |
| --- | --- |
| `kubectl get deployment` 显示所有部署 |
| kubectl get pod | 显示所有 Pods |
| kubectl get svc | 显示所有服务 |
| kubectl logs <pod_id> | 显示指定 Pod 的日志 |
| kubectl delete deployment <name> | 删除指定的部署 |
| kubectl delete pod <name> | 删除指定的 Pod |
| kubectl delete svc <service> | 删除指定的服务 |
| kubectl port-forward svc <service> <host-port>:<cluster-port> | 将本地机器的流量转发到集群内部 |

## A.4 其他工具

本节将介绍本书中用于执行特定任务的一系列有用工具，例如安全漏洞扫描或 HTTP 交互。

### A.4.1 HTTPie

HTTPie 是一个方便的“命令行 HTTP 和 API 测试客户端”([`httpie.org`](https://httpie.org))。它专为人类设计，并提供卓越的用户体验。请参考官方文档获取安装说明和更多关于该工具的信息。

在 macOS 和 Linux 上，您可以使用 Homebrew 进行安装，如下所示：

```
$ brew install httpie
```

作为安装的一部分，您将获得两个可以从终端窗口使用的工具：http 和 https。例如，您可以发送以下 GET 请求：

```
$ http pie.dev/get
```

### A.4.2 Grype

在供应链安全背景下，我们使用 Grype 扫描 Java 代码库和容器镜像中的漏洞([`github.com/anchore/grype`](https://github.com/anchore/grype))。扫描是在您运行它的机器上本地进行的，这意味着您的任何文件或工件都不会发送到外部服务。这使得它在更受监管的环境或断网场景中非常适合。请参考官方文档获取更多信息。

在 macOS 和 Linux 上，您可以使用 Homebrew 进行安装，如下所示：

```
$ brew tap anchore/grype
$ brew install grype
```

该工具目前尚不支持 Windows。如果您是 Windows 用户，我建议您利用 Windows Subsystem for Linux 2 (WSL2)并在其中安装 Grype。有关 WSL2 的更多信息，您可以参考官方文档([`docs.microsoft.com/en-us/windows/wsl/`](https://docs.microsoft.com/en-us/windows/wsl/))。

### A.4.3 Tilt

Tilt([`tilt.dev`](https://tilt.dev))旨在在 Kubernetes 上工作时提供良好的开发者体验。它是一个开源工具，提供在本地环境中构建、部署和管理容器化工作负载的功能。请参考官方文档获取安装说明([`docs.tilt.dev/install.html`](https://docs.tilt.dev/install.html))。

在 macOS 和 Linux 上，您可以使用 Homebrew 进行安装，如下所示：

```
$ brew install tilt-dev/tap/tilt
```

### A.4.4 八分仪

八分仪([`octant.dev`](https://octant.dev))是一个“面向开发者的开源 Kubernetes Web 界面，允许您检查 Kubernetes 集群及其应用程序。”请参考官方文档获取安装说明([`reference.octant.dev`](https://reference.octant.dev))。

在 macOS 和 Linux 上，您可以使用 Homebrew 进行安装，如下所示：

```
$ brew install octant
```

### A.4.5 Kubeval

Kubeval ([www.kubeval.com](http://www.kubeval.com)) 是一个方便的工具，当你需要“验证一个或多个 Kubernetes 配置文件”时。我们将在部署管道中使用它，以确保所有我们的 Kubernetes 清单都格式正确且符合 Kubernetes API。请参阅官方文档以获取安装说明 ([www.kubeval.com/installation/](http://www.kubeval.com/installation/))。

在 macOS 和 Linux 上，你可以使用 Homebrew 如下安装：

```
$ brew tap instrumenta/instrumenta
$ brew install kubeval
```

### A.4.6 Knative CLI

Knative 是一个“基于 Kubernetes 的平台，用于部署和管理现代无服务器工作负载” ([`knative.dev`](https://knative.dev))。该项目提供了一个方便的 CLI 工具，你可以使用它来与 Kubernetes 集群中的 Knative 资源进行交互。请参阅官方文档以获取安装说明 ([`knative.dev/docs/install/quickstart-install`](https://knative.dev/docs/install/quickstart-install))。

在 macOS 和 Linux 上，你可以使用 Homebrew 如下安装：

```
$ brew install kn
```
