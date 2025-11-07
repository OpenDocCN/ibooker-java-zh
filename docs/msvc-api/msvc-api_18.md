# 14 使用 Kubernetes 部署微服务 API

本章涵盖了

+   使用 AWS 的 Elastic Kubernetes Service (EKS) 创建集群

+   使用 AWS Load Balancer Controller 暴露服务

+   将服务部署到 Kubernetes 集群

+   在 Kubernetes 中安全地管理机密

+   部署 Aurora Serverless 数据库

Kubernetes 是一个开源的容器编排框架，它正在迅速成为跨平台部署和管理应用程序的标准方式。您可以自己将 Kubernetes 部署到自己的服务器上，或者您可以使用托管 Kubernetes 服务。在两种情况下，您都将获得对服务的统一接口，这意味着跨云提供商迁移对您的运营影响较小。您还可以在自己的机器上部署 Kubernetes 集群，并以与云中相同的方式在本地运行测试。

使用 minikube 在本地运行 kubernetes 您可以使用 minikube 在本地运行 Kubernetes 集群。尽管我们不会在本章中介绍它，但 minikube 是一个很好的工具，可以帮助您更熟悉 Kubernetes。请查看 minikube 的官方文档（[`minikube.sigs.k8s.io/docs/start/`](https://minikube.sigs.k8s.io/docs/start/)）。

自己部署 Kubernetes 是熟悉这项技术的良好练习，但在实践中，大多数公司都使用托管服务。在本章中，我们将使用 Kubernetes 托管服务来部署我们的集群。许多供应商提供 Kubernetes 托管服务。主要玩家包括 Google Cloud 的 Google Kubernetes Engine (GKE)、Azure 的 Kubernetes 服务 (AKS) 和 AWS 的 Elastic Kubernetes Service (EKS)。这三个服务都非常稳健，并提供类似的功能。¹ 在本章中，我们将使用 EKS，它是目前最受欢迎的托管 Kubernetes 服务。²

为了说明如何将应用程序部署到 Kubernetes 集群，我们将以订单服务为例。我们还将创建一个 Aurora Serverless 数据库，并展示如何使用 Kubernetes secrets 安全地将数据库连接凭证传递给服务。

本章不假设您对 AWS 或 Kubernetes 有先前的知识。我已经努力详细解释了 Kubernetes 和 AWS 的每个概念，以便您即使没有这两种技术的先前经验，也能跟随示例。关于这些主题已经写出了整本书，所以本章只是一个概述，并提供了一些其他资源的引用，您可以使用这些资源深入了解这些话题。

在继续之前，请记住，本章中使用的 EKS 和其他 AWS 服务是付费服务，所以这是本书中唯一一个如果你跟随示例可能会让你花费一些金钱的章节。在 AWS EKS 中的 Kubernetes 集群的基础费用是每小时 $0.10，相当于每天 $2.40，大约每月 $72。如果预算是个问题，我的建议是先阅读本章，了解我们在做什么，然后尝试 EKS 示例。如果你是第一次使用 EKS 和 Kubernetes，可能需要一两天的时间来处理示例，所以尽量安排时间来处理这些示例。第 14.9 节描述了如何删除 EKS 集群以及本章创建的所有其他资源，以确保你不产生额外的费用。

不再拖延，让我们开始吧！我们将从设置环境开始。

## 14.1 设置本章的环境

在本节中，我们设置环境，以便你可以跟随本章其余部分的示例。即使你并不打算尝试这些示例，我也建议你至少快速浏览这一节，了解我们将要使用的工具。本章工具较多，因此在这里我们安装最重要的依赖项，在接下来的章节中，你将找到其他工具的额外说明。

首先，通过运行以下命令将第十三章的代码复制到一个名为 ch14 的新文件夹中：

```
$ cp -r ch13 ch14
```

进入 ch14 目录，安装依赖项，并通过运行以下命令激活虚拟环境：

```
$ cd ch14 && pipenv install --dev && pipenv shell
```

由于我们将部署到 AWS，我们需要能够以编程方式访问 AWS 服务。在第十三章中，我们安装并配置了 AWS CLI。如果你还没有这样做，请回到 13.1 节，按照步骤安装和配置 AWS CLI。

你将学习如何将服务部署到 Kubernetes，因此你还需要安装 Kubernetes CLI，即 kubectl。根据你使用的平台，安装 kubectl 有不同的方法，所以请参考官方文档以查看哪个选项最适合你 ([`kubernetes.io/docs/tasks/tools/`](https://kubernetes.io/docs/tasks/tools/))。

最后，在本章中我们将大量使用 `jq`——一个帮助我们解析和查询 JSON 文档的 CLI 工具。`jq` 并非严格必要，以跟随本章中的示例，但它确实让一切变得更容易，如果你之前没有使用过这个工具，我强烈建议你了解它。我们将主要使用 `jq` 来过滤 JSON 有效负载并从中检索特定属性。与 Kubernetes 一样，根据你的平台，有不同的安装选项，所以请参考官方文档以了解哪种策略最适合你 ([`stedolan.github.io/jq/download/`](https://stedolan.github.io/jq/download/))。

现在我们已经准备好了环境，是时候部署了！在我们创建集群之前，下一节将解释一些与 Kubernetes 相关的主要概念，以确保你能理解接下来的章节。如果你有 Kubernetes 的先前经验，可以跳过第 14.2 节。

## 14.2 Kubernetes 的工作原理：简化版

那么，Kubernetes 是什么？如果你没有 Kubernetes 的先前经验或者对其工作原理仍然感到困惑，本节提供了一个对 Kubernetes 主要组件的超压缩介绍。

Kubernetes 是一个开源的容器编排工具。*容器编排*是指运行容器化应用的过程。除了容器编排，Kubernetes 还帮助我们自动化部署，并处理优雅的滚动发布和回滚、应用扩展等更多功能。

图 14.1 提供了 Kubernetes 集群主要组件的高级概述。Kubernetes 集群的核心是控制平面，这是一个运行我们集群的 Kubernetes API、控制其状态并管理可用资源的过程，以及其他许多任务。在控制平面上也可以安装附加组件，包括特定的 DNS 服务器，如 CoreDNS。

![](img/14-01.png)

图 14.1 Kubernetes 集群的高级架构图，展示了集群所有组件如何协同工作。

**定义** Kubernetes 控制平面是一个运行 Kubernetes API 并控制集群状态以及管理可用资源、调度和其他许多任务的过程。有关控制平面的更多信息，请参阅 Jay Vyas 和 Chris Love 所著的《Core Kubernetes》的第十一章([`mng.bz/yayE`](http://mng.bz/yayE))和第十二章([`mng.bz/M0dm`](http://mng.bz/M0dm))（Manning, 2022）。

Kubernetes 中最小的计算单元是*pod*：围绕一个或多个容器的一个包装器。最常见的做法是每个*pod*运行一个容器，在本章中，我们将订单服务作为每个*pod*的单个容器进行部署。

要将*pods*部署到集群中，我们使用*工作负载*。Kubernetes 有四种类型的工作负载：`Deployment`、`StatefulSet`、`DaemonSet`和`Job/CronJob`。`Deployment`是 Kubernetes 中最常见的工作负载类型，适用于运行无状态分布式应用。`StatefulSet`用于运行需要同步状态的状态化分布式应用。使用`DaemonSet`来定义应在集群的所有或大多数节点上运行的进程，例如日志收集器。`Job`和`CronJob`帮助我们定义一次性进程或应用，这些进程或应用需要按计划运行，例如每天或每周一次。

要部署微服务，我们使用 `Deployment` 或 `StatefulSet`。由于我们的服务都是无状态的，在本章中我们将订单服务作为 `Deployment` 进行部署。为了管理 pod 的数量，部署使用 `ReplicaSet` 的概念，这是一个维护集群中所需 pod 数量的进程。

工作负载通常在 *命名空间* 内进行范围划分。在 Kubernetes 中，命名空间是资源的逻辑分组，允许我们隔离和范围我们的部署。例如，我们可以在平台中的每个服务上创建一个命名空间。命名空间使得管理我们的部署和避免名称冲突变得更容易：我们的资源名称必须在命名空间内是唯一的，但不需要在命名空间之间是唯一的。

要将我们的应用程序作为网络服务运行，Kubernetes 提供了 *服务* 的概念——这些是管理我们 pod 接口并使它们之间能够通信的进程。为了通过互联网公开我们的服务，我们使用一个 *负载均衡器*，它位于 Kubernetes 集群之前，并根据入口规则将流量转发到服务。

Kubernetes 系统的最后一部分是 *节点*，它代表我们的服务运行的实际计算资源。我们将节点定义为计算资源，因为它们可以是物理服务器到虚拟机中的任何东西。例如，当在 AWS 上运行 Kubernetes 集群时，我们的节点将由 EC2 机器表示。

现在我们已经了解了 Kubernetes 的主要组成部分，让我们创建一个集群吧！

## 14.3 使用 EKS 创建 Kubernetes 集群

在本节中，您将学习如何使用 AWS EKS 创建 Kubernetes 集群。我们使用 eksctl 启动 Kubernetes 集群，这是在 AWS 中管理 Kubernetes 的推荐工具。

eksctl 是由 Weaveworks 创建和维护的一个开源工具。它使用 CloudFormation 在幕后创建和管理 Kubernetes 集群的变化。这是一个好消息，因为它意味着我们可以重用 CloudFormation 模板来在不同环境中复制相同的架构。这也使得我们对集群的所有更改都通过 CloudFormation 可见。

定义 *CloudFormation* 是 AWS 的基础设施即代码服务。使用 CloudFormation，我们可以在称为 *模板* 的 YAML 或 JSON 文件中声明我们的资源。当我们提交模板到 CloudFormation 时，AWS 创建一个 CloudFormation *堆栈*，即模板中定义的资源集合。CloudFormation 模板不应包含敏感信息，并且可以提交到我们的代码仓库中，这使得我们对基础设施的更改非常可见，并且可以在不同的环境中复制。

根据您使用的平台，安装 eksctl 有多种方法，因此请参阅官方文档以了解哪种策略最适合您（[`github.com/weaveworks/eksctl`](https://github.com/weaveworks/eksctl)）。

要在 Kubernetes 集群中运行容器，我们使用 AWS Fargate。如图 14.2 所示，Fargate 是 AWS 的无服务器容器服务，允许我们在云中运行容器而无需配置服务器。使用 AWS Fargate，你无需担心服务器扩展的问题，因为 Fargate 会处理这些。

![图片](img/14-02.png)

图 14.2 AWS Fargate 自动配置运行我们的 Kubernetes 集群所需的服务器。

要使用 eksctl 创建 Kubernetes 集群，请运行以下命令：

```
$ eksctl create cluster --name coffeemesh --region <aws_region> --fargate \
--alb-ingress-access
```

创建过程大约需要 30 分钟来完成。让我们看看这个命令中的每个标志：

+   `--name`—集群的名称。我们将集群命名为`coffeemesh`。

+   `--region`—你想要部署集群的 AWS 区域。这个区域应该与你在第 13.4 节中创建 ECR 仓库时使用的区域相同。

+   `--fargate`—创建一个 Fargate 配置文件，用于在`default`和`kube-system`命名空间中调度 Pod。Fargate 配置文件是确定哪些 Pod 必须由 Fargate 启动的策略。

+   `--alb-ingress-access`—通过应用程序负载均衡器启用对集群的访问。

图 14.3 说明了 eksctl 在启动 Kubernetes 集群时创建的堆栈架构。默认情况下，eksctl 为集群创建一个专用的虚拟私有云（VPC）。

![图片](img/14-03.png)

图 14.3 eksctl 创建了一个包含三个公共网络、三个私有网络、两个 CIDR 预留和两个 VPC 安全组的 VPC。它还在 VPC 内部部署了 Kubernetes 集群。

Kubernetes 网络要高级使用 Kubernetes，你需要了解 Kubernetes 中的网络是如何工作的。要了解更多关于 Kubernetes 网络的信息，请查看 James Strong 和 Vallery Lancey 所著的《Networking and Kubernetes: A Layered Approach》（O’Reilly，2021 年）。

还可以在现有的 VPC 内部启动集群，通过指定你想要在其中运行部署的子网来实现。如果在现有的 VPC 内部启动，你必须确保 VPC 和提供的子网已正确配置以运行 Kubernetes 集群。有关 Kubernetes 集群的网络要求，请参阅 eksctl 文档（[`eksctl.io/usage/vpc-networking/`](https://eksctl.io/usage/vpc-networking/)）和 AWS 关于 Kubernetes 集群 VPC 网络要求的官方文档（[`mng.bz/aPRY`](http://mng.bz/aPRY)）。

如图 14.3 所示，eksctl 默认创建六个子网：三个公共和三个私有，以及它们对应的 NAT 网关和路由表。*子网*是 VPC 中可用 IP 地址的子集。公共子网可以通过互联网访问，而私有子网则不行。eksctl 还创建了两个用于 Kubernetes 内部使用的子网 CIDR 预留，以及两个安全组；其中一个允许集群中所有节点之间的通信，另一个允许控制平面与节点之间的通信。

*CIDR*定义的无类域间路由，是一种用于表示 IP 地址范围的表示法。CIDR 表示法包括一个 IP 地址后跟一个斜杠和一个十进制数，其中十进制数表示地址范围。例如，255.255.255.255/32 表示一个地址的范围。要了解更多关于 CIDR 表示法的信息，请参阅维基百科的文章：[`en.wikipedia.org/wiki/Classless_Inter-Domain_Routing`](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing)。

一旦我们创建了集群，我们就可以配置 kubectl 指向它，这将允许我们通过命令行管理集群。使用以下命令将 kubectl 指向集群：

```
$ aws eks update-kubeconfig --name coffeemesh --region <aws_region>
```

现在我们已经连接到集群，我们可以检查其属性。例如，我们可以使用以下命令获取正在运行的节点列表：

```
$ kubectl get nodes
# output truncated:
NAME                                       STATUS  ROLES  AGE    VERSION
fargate-ip-192-168-157-75.<aws_region>...  Ready  <none>  4d16h  v1.20.7...
fargate-ip-192-168-170-234.<aws_region>... Ready  <none>  4d16h  v1.20.7...
fargate-ip-192-168-173-63.<aws_region>...  Ready  <none>  4d16h  v1.20.7...
```

要获取集群中运行的 Pod 列表，请运行以下命令：

```
$ kubectl get pods -A
NAMESPACE    NAME                      READY  STATUS   RESTARTS  AGE
kube-system  coredns-647df9f975-2ns5m  1/1    Running  0         2d15h
kube-system  coredns-647df9f975-hcgjq  1/1    Running  0         2d15h
```

您可以运行许多其他有用的命令来了解更多关于您的集群的信息。查看 Kubernetes CLI 的官方文档以获取更多命令和选项（[`kubernetes.io/docs/reference/kubectl/`](https://kubernetes.io/docs/reference/kubectl/)）。一个好的起点是 kubectl 速查表（[`kubernetes.io/docs/reference/kubectl/cheatsheet/`](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)）。现在我们的集群已经启动并运行，在下一节中，我们将为我们的 Kubernetes 服务账户创建一个 IAM 角色。

## 14.4 为 Kubernetes 服务账户使用 IAM 角色

在您的 Kubernetes 集群中运行的每个进程都有一个身份，这个身份由一个*服务账户*提供。服务账户决定了进程在集群内的访问权限。有时，我们的服务需要使用 AWS API 与 AWS 资源进行交互。为了给 AWS API 提供访问权限，我们需要为我们的服务创建*IAM 角色*——这些实体为应用程序提供访问 AWS API 的权限。如图 14.4 所示，要将 Kubernetes 服务账户链接到 IAM 角色，我们使用 OpenID Connect (OIDC)。通过使用 OIDC，我们的 Pod 可以获取临时凭证以访问 AWS API。

![图片](img/14-04.png)

图 14.4 Pod 可以通过 OIDC 提供者进行身份验证以假定 IAM 角色，这使它们能够访问 AWS API，因此能够访问 AWS 服务。

要检查您的集群是否有 OIDC 提供者，请运行以下命令，将`<cluster_name>`替换为您的集群名称：

```
$ aws eks describe-cluster --name coffeemesh \
--query "cluster.identity.oidc.issuer" --output text
```

您将得到以下类似的输出：

```
https://oidc.eks.<aws_region>.amazonaws.com/id/BE4E5EE7DCDF9FB198D06FC9883F
➥ F1BE
```

在这种情况下，集群 OIDC 提供者的 ID 是`BE4E5EE7DCDF9FB198D06FC9883FF1BE`。获取 OIDC 提供者的 ID 并运行以下命令：

```
$ aws iam list-open-id-connect-providers | \
grep BE4E5EE7DCDF9FB198D06FC9883FF1BE
```

此命令列出您 AWS 账户中的所有 OIDC 提供者，并使用`grep`根据您集群 OIDC 提供者的 ID 进行过滤。如果您得到结果，这意味着您已经为您的集群配置了 OIDC 提供者。如果您没有输出任何内容，这意味着您没有 OIDC 提供者，因此让我们创建一个！要为您的集群创建 OIDC 提供者，请运行以下命令，将`<cluster_name>`替换为您的集群名称：

```
$ eksctl utils associate-iam-oidc-provider --cluster <cluster_name> \
--approve
```

就这么简单。现在我们可以将 IAM 角色链接到我们的服务账户！在下一节中，我们将部署一个 Kubernetes 负载均衡器以使集群能够接收外部流量。

## 14.5 部署 Kubernetes 负载均衡器

目前，我们的集群无法从 VPC 外部访问。如果我们部署应用程序，它们将只获得内部 IP，因此无法对外部世界访问。为了使集群能够从外部访问，我们需要一个*入口控制器*。如图 14.5 所示，入口控制器接受来自 Kubernetes 集群外部的流量，并在我们的 Pod 之间进行负载均衡。为了将流量重定向到特定的 Pod，我们为每个服务创建*入口资源*。入口控制器负责管理入口资源。

![](img/14-05.png)

图 14.5 入口控制器接受来自 Kubernetes 集群外部的流量，并根据入口资源定义的规则将其转发到 Pod。

在本节中，我们将部署一个 Kubernetes 入口控制器作为 AWS 负载均衡器控制器。³如图 14.5 所示，AWS 负载均衡器控制器部署了一个 AWS 应用程序负载均衡器（ALB），它位于我们的集群前面，捕获传入的流量并将其转发到我们的服务。为了将流量转发到我们的服务，ALB 使用*目标组*的概念——这是从 ALB 到特定资源转发流量的规则。例如，我们可以根据 IP、服务 ID 和其他因素设置目标组。负载均衡器监控其注册目标的状态，确保流量只被重定向到健康的目标。

要安装 AWS 负载均衡器控制器，我们需要在集群中有一个 OIDC 提供者，因此请确保您在继续之前已经阅读了第 14.4 节。部署 AWS 负载均衡器控制器的第一步是创建一个 IAM 策略，该策略允许控制器访问相关的 AWS API。维护 AWS 负载均衡器控制器项目的开源社区提供了一个我们需要策略的样本，因此我们只需获取它：

```
$ curl -o alb_controller_policy.json \
https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-
➥ controller/main/docs/install/iam_policy.json
```

运行此命令后，你会在你的目录中看到一个名为 alb_controller_policy.json 的文件。现在我们可以使用此文件创建 IAM 策略：

```
$ aws iam create-policy \
--policy-name ALBControllerPolicy \
--policy-document file://alb_controller_policy.json
```

下一步是创建一个 IAM 角色与 Kubernetes 服务账户关联，用于负载均衡器，命令如下：

```
$ eksctl create iamserviceaccount \
  --cluster=coffeemesh \
  --namespace=kube-system \
  --name=alb-controller \
  --attach-policy-arn=arn:aws:iam::<aws_account_id>:policy/ALBControllerPolicy \
  --override-existing-serviceaccounts \
  --approve
```

此命令创建一个 CloudFormation 堆栈，其中包括与之前创建的策略关联的 IAM 角色，以及名为 `alb-controller` 的服务账户，位于为 Kubernetes 集群的系统组件保留的 `kube-system` 命名空间内。

现在我们可以安装负载均衡器控制器。我们将使用 Helm 来安装控制器，它是 Kubernetes 的包管理器。如果你在机器上没有安装 Helm，你需要安装它。根据你的平台，安装 Helm 有不同的策略，所以请确保查看文档以了解哪种选项最适合你（[`helm.sh/docs/intro/install/`](https://helm.sh/docs/intro/install/))）。

一旦 Helm 在你的机器上可用，你需要通过添加 EKS 图表存储库到你的本地 `helm`（在 Helm 中，包被称为 *charts*）来更新它。要添加 EKS 图表，请运行以下命令：

```
$ helm repo add eks https://aws.github.io/eks-charts
```

现在让我们更新 `helm`，以确保我们获取到最新的图表更新：

```
$ helm repo update
```

现在 `helm` 已更新，我们可以安装 AWS 负载均衡器控制器。要安装控制器，我们需要获取在启动集群时 eksctl 创建的 VPC 的 ID。要找到 VPC ID，请运行以下命令：

```
$ eksctl get cluster --name coffeemesh -o json | \
jq '.[0].ResourcesVpcConfig.VpcId'
# output: "vpc-07d35ccc982a082c9"
```

要成功运行前面的命令，你需要安装 `jq`。请参阅第 14.1 节了解如何安装它。现在我们可以通过运行以下命令来安装控制器：

```
$ helm install aws-load-balancer-controller eks/aws-load-balancer-
➥ controller \
  -n kube-system \
  --set clusterName=coffeemesh \
  --set serviceAccount.create=false \
  --set serviceAccount.name=alb-controller \
  --set vpcId=<vpc_id>
```

由于控制器是 Kubernetes 的内部组件，我们在 `kube-system` 命名空间内安装它。我们确保控制器为 `coffeemesh` 集群安装。我们还指示 Helm 不要为控制器创建新的服务账户，而是使用我们之前创建的 `alb-controller` 服务账户。

所有资源创建需要几分钟时间。要验证部署是否成功，请运行以下命令：

```
$ kubectl get deployment -n kube-system aws-load-balancer-controller
NAME              READY    UP-TO-DATE    AVAILABLE    AGE
alb-controller    2/2      2             2            84s
```

当 `READY` 列显示 2/2 时，你就知道控制器正在运行，这意味着所需的资源数量已经启动。我们的集群现在已准备就绪，是时候部署订单服务了！

## 14.6 在 Kubernetes 集群中部署微服务

现在我们的 Kubernetes 集群已准备就绪，是时候开始部署我们的服务了！在本节中，我们将介绍部署订单服务所需的步骤。你可以遵循相同的步骤来部署 CoffeeMesh 平台的其他服务。

![](img/14-06.png)

图 14.6 要部署一个微服务，我们创建一个新的命名空间，并在该命名空间内部署所有运行微服务所需的组件，例如 `Deployment` 对象和 `Service` 对象。

如图 14.6 所示，我们将订单服务部署到名为`orders-service`的新命名空间。这允许我们逻辑上分组和隔离操作订单服务所需的所有资源。要创建新的命名空间，请运行以下命令：

```
$ kubectl create namespace orders-service
```

由于我们将在新的命名空间中运行订单服务，因此我们还需要创建一个新的 Fargate 配置文件，该配置文件配置为在`orders-service`命名空间内调度作业。要创建新的 Fargate 配置文件，请运行以下命令：

```
$ eksctl create fargateprofile --namespace orders-service --cluster \
coffeemesh --region <aws_region>
```

随着`orders-service`命名空间和 Fargate 配置文件的准备就绪，我们可以部署订单服务。要执行部署，我们采取以下步骤：

1.  为订单服务创建一个部署对象。

1.  创建一个服务对象。

1.  创建一个 ingress 资源以公开服务。

以下各节将详细说明如何在每个步骤中进行操作。

### 14.6.1 创建部署对象

让我们从创建一个用于订单服务的部署开始，使用服务清单文件。如图 14.7 所示，部署是 Kubernetes 对象，它们操作我们的 Pod，并为它们提供运行所需的一切，包括 Docker 镜像和端口配置。创建一个名为 orders-service-deployment.yaml 的文件，并将列表 14.1 的内容复制到其中。

![](img/14-07.png)

图 14.7 `Deployment`对象为 Pod 提供必要的配置，例如它们的 Docker 镜像和端口配置，并确保我们运行所需的 Pod 数量。

我们使用 Kubernetes 的 API 版本 apps/v1，并声明此对象为`Deployment`。在元数据中，我们命名部署为`orders-service`，指定其命名空间，并添加标签`app:` `orders-service`。*标签*是 Kubernetes 对象的自定义标识符，可用于监控、跟踪或调度任务等多种用途。⁴

在`spec`部分，我们定义了一个选择器规则，该规则匹配带有标签`app:` `orders-service`的 Pod，这意味着此部署将仅操作具有此标签的 Pod。我们还声明我们只想运行一个 Pod 副本。

在`spec.template`部分，我们定义由此部署操作的 Pod。我们使用与部署选择器规则一致的`app:` `orders-service`键值对标记 Pod。在 Pod 的`spec`部分，我们声明属于 Pod 的容器。在这种情况下，我们只想运行一个容器，即订单服务应用程序。在订单服务容器的定义中，我们指定运行应用程序必须使用的镜像以及应用程序运行的端口。

列表 14.1 声明部署清单

```
# file: orders-service-deployment.yaml

apiVersion: apps/v1                   ①
kind: Deployment                      ②
metadata:
  name: orders-service                ③
  namespace: orders-service           ④
  labels:                             ⑤
    app: orders-service
spec:                                 ⑥
  replicas: 1                         ⑦
  selector:
    matchLabels:
      app: orders-service             ⑧
  template:                           ⑨
    metadata:
      labels:
        app: orders-service           ⑩
    spec:                             ⑪
      containers:
      - name: orders-service
        image: <aws_account_id>.dkr.ecr.<aws_region>.amazonaws.com/coffeemesh-orders:1.0            ⑫
        ports:
          - containerPort: 8000       ⑬
        imagePullPolicy: Always
```

① 此清单中使用的 Kubernetes API 版本

② 此清单定义了一个 Deployment 对象。

③ 部署的名称

④ 部署必须位于的命名空间

⑤ 部署的标签

⑥ 部署的规范

⑦ 需要部署多少个 Pod

⑧ Pod 的标签选择器

⑨ Pod 的模板

⑩ Pod 的标签

⑪ Pod 的规范

⑫ Pod 的镜像

⑬ API 运行的端口

要创建部署，我们运行以下命令：

```
$ kubectl apply -f orders-service-deployment.yaml
```

此命令创建部署并启动清单文件中定义的 Pod。Pod 变为可用可能需要几秒钟。您可以使用以下命令检查它们的状态：

```
$ kubectl get pods -n orders-service
```

Pod 的初始状态将是`Pending`，一旦它们启动并运行，其状态将变为`Running`。

什么是 Kubernetes 清单文件？

在 Kubernetes 中，我们可以使用清单文件创建对象。对象是资源，如命名空间、部署、服务等。清单文件是一个 YAML 文件，描述了对象的属性及其期望状态。使用清单文件很方便，因为它们可以在源控制中跟踪，这有助于我们跟踪对基础设施的更改。

每个清单文件至少包含以下属性：

+   `apiVersion`—我们想要使用的 Kubernetes API 版本。每个 Kubernetes 对象都有自己的稳定版本。您可以通过运行以下命令来检查您 Kubernetes 集群中每个对象的最新稳定版本：`kubectl api-resources`。

+   `kind`—我们正在创建的对象类型。可能的值包括`Service`、`Ingress`和`Deployment`等。

+   `metadata`—一组属性，提供有关对象标识信息的集合，例如其名称、其命名空间和附加标签。

+   `spec`—对象的规范。例如，如果我们正在创建服务，我们使用此部分来指定我们正在创建的服务类型（例如，`NodePort`）和选择规则。

要从清单文件创建对象，我们使用`kubectl apply`命令。例如，如果我们有一个名为 deployment.yaml 的清单文件，我们使用以下命令应用它：

```
$ kubectl apply -f deployment.yaml
```

### 14.6.2 创建服务对象

现在我们的部署已经就绪，我们将为订单服务创建一个服务对象。正如我们在第 14.2 节中学到的，服务是 Kubernetes 对象，允许我们将 Pod 作为网络服务暴露。如图 14.8 所示，服务对象将我们的应用程序作为 Web 服务暴露，并将流量从集群重定向到指定端口上的我们的 Pod。创建一个名为 orders-service.yaml 的文件，并将列表 14.2 的内容复制进去，该列表显示了如何配置简单的服务清单。

![](img/14-08.png)

图 14.8 一个服务对象将集群中的流量重定向到指定端口上的 Pod。在这个例子中，端口 80 的集群入站流量被重定向到 Pod 的端口 8000。

我们使用 Kubernetes API 的 v1 版本来声明我们的服务。在元数据中，我们指定服务的名称为`orders-service`，并且要在`orders-service`命名空间内启动。我们还添加了一个标签：`app:` `orders-service`。在服务的`spec`部分，我们配置了`ClusterIP`类型，这意味着 pod 只能在集群内部访问。Kubernetes 中还有其他类型的服务，例如`NodePort`和`LoadBalancer`。（要了解更多关于服务类型及其使用情况，请参阅侧边栏，“我应该使用哪种类型的 Kubernetes 服务？”）

我们还创建了一个转发规则，将来自 80 端口的流量重定向到 8000 端口，这是我们容器运行的端口。最后，我们指定了一个`app:` `orders-service`标签的选择器，这意味着此服务将仅操作带有该标签的 pods。

列表 14.2 声明服务清单

```
# file: orders-service.yaml

apiVersion: v1
kind: Service                  ①
metadata:
  name: orders-service
  namespace: orders-service
  labels:
    app: orders-service
spec:
  selector:
    app: orders-service
  type: ClusterIP              ②
  ports:
    - protocol: http           ③
      port: 80                 ④
      targetPort: 8000         ⑤
```

① 此清单定义了一个 Service 对象。

② 这是一个 ClusterIP 类型的 Service。

③ 服务通过 HTTP 进行通信。

④ 服务必须映射到端口 80。

⑤ 服务在内部运行在端口 8000 上。

要部署此服务，请运行以下命令：

```
$ kubectl apply -f orders-service.yaml
```

我应该使用哪种类型的 Kubernetes 服务？

Kubernetes 有四种类型的服务。在这里，我们讨论每种服务类型的特性和用例：

+   `ClusterIP`—在集群的内部 IP 上暴露服务，因此只能在集群内部访问

+   `NodePort`—在节点的外部 IP 上暴露服务，因此使它们在集群网络上可用

+   `LoadBalancer`—通过专用云负载均衡器直接暴露服务

+   `ExternalName`—通过集群内部的内部 DNS 记录暴露服务

应该使用哪种类型？这取决于您的需求。`NodePort`如果您想能够从运行它们的节点的 IP 地址外部访问服务，那么它很有用。缺点是服务使用节点的静态端口，因此每个节点只能运行一个服务。`ClusterIP`如果您更愿意通过集群的 IP 访问服务，那么它很有用。`ClusterIP`服务不能从集群外部直接访问，但您可以通过创建将流量转发到它们的 ingress 规则来暴露它们。`LoadBalancer`如果您想为每个服务使用一个云负载均衡器，那么它很有用。使用每个服务的负载均衡器可以使配置稍微简单一些，因为您不需要配置多个 ingress 规则。然而，负载均衡器通常是集群中最昂贵的组件，所以如果预算是一个因素，您可能不想使用这个选项。最后，`ExternalName`如果您想能够使用自定义域名从集群内部访问服务，那么它很有用。

### 14.6.3 使用 ingress 对象暴露服务

最后一步是通过互联网公开服务。为了公开服务，我们需要创建一个路由到服务的入口资源。如图 14.9 所示，入口资源是一个服务，它将 HTTP 流量重定向到我们在指定端口和 URL 路径上运行的 Kubernetes 集群中的 pod。创建一个名为 orders-service-ingress.yaml 的文件，并将列表 14.3 的内容复制到其中。

![](img/14-09.png)

图 14.9 入口对象允许我们将特定端口和 URL 路径上的 HTTP 流量重定向到服务对象。

在入口清单中，我们使用 Kubernetes API 的 networking.k8s.io/v1 版本，并声明对象为 `Ingress` 类型。在 `metadata` 中，我们命名入口对象为 `orders-service-ingress`，并指定它应在 `orders-service` 命名空间内部署。我们使用注解将入口对象绑定到我们在 14.5 节中部署的 AWS 负载均衡器。在 `spec` 部分中，我们定义入口资源的转发规则。我们声明了一个 HTTP 规则，将所有 `/orders` 路径下的流量转发到订单服务，以及访问服务 API 文档的附加规则。

列表 14.3 声明入口清单

```
# file: orders-service-ingress.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress                                              ①
metadata:
  name: orders-service-ingress
  namespace: orders-service
  annotations:                                             ②
    kubernetes.io/ingress.class: alb                       ③
    alb.ingress.kubernetes.io/target-type: ip              ④
    alb.ingress.kubernetes.io/scheme: internet-facing      ⑤
spec:
  rules:                                                   ⑥
  - http:
      paths:
      - path: /orders                                      ⑦
        pathType: Prefix                                   ⑧
        backend:                                           ⑨
          service:
            name: orders-service                           ⑩
            port:
              number: 80                                   ⑪
      - path: /docs/orders
        pathType: Prefix
        backend:
          service:
            name: orders-service
            port:
              number: 80
      - path: /openapi/orders.json
        pathType: Prefix
        backend:
          service:
            name: orders-service
            port:
              number: 80
```

① 清单定义了一个入口对象。

② 入口的 AWS 配置

③ 入口暴露了一个应用程序负载均衡器。

④ 根据 IP 地址将流量路由到 pod。

⑤ 入口可用于外部连接。

⑥ 流量转发规则

⑦ 针对 /orders URL 路径的规则

⑧ 该规则适用于以 /orders 前缀开始的请求。

⑨ 处理此流量的后端服务

⑩ 流量必须路由到 orders-service 服务。

⑪ orders-service 服务在端口 80 上可用。

要创建此入口资源，我们运行以下命令：

```
$ kubectl apply -f orders-service-ingress.yaml
```

订单 API 现在可访问。要调用 API，我们首先需要找到我们刚刚创建的入口规则的端点。运行以下命令以获取入口资源的详细信息：

```
$ kubectl get ingress/orders-service-ingress -n orders-service
# output truncated:
NAME                     CLASS    HOSTS   ADDRESS...
orders-service-ingress   <none>   *       k8s-ordersse-ordersse-3c391193...
```

`ADDRESS` 字段下的值是负载均衡器的 URL。您也可以通过运行以下命令来获取此值：

```
$ kubectl get ingress/orders-service-ingress -n orders-service -o json | \
jq '.status.loadBalancer.ingress[0].hostname'
"k8s-ordersse-ordersse-3c39119336-236890178.<aws_region>.elb.amazonaws.com"
```

我们可以使用此 URL 调用订单服务 API。由于数据库尚未就绪，API 本身将无法工作，但我们可以访问 API 文档：

```
$ curl http://k8s-ordersse-ordersse-3c39119336-
➥ 236890178.<aws_region>.elb.amazonaws.com/openapi/orders.json
```

负载均衡器变为可用可能需要一些时间，在此期间 `curl` 将无法解析主机。如果发生这种情况，请等待几分钟再试。为了能够与 API 交互，我们必须设置一个数据库，这将是我们下一节的目标！

## 14.7 使用 AWS Aurora 设置无服务器数据库

订单服务几乎准备好了：应用程序正在运行，我们可以通过互联网访问它。唯一缺少的部分是数据库。我们有很多选择来设置数据库。我们可以在我们的 Kubernetes 集群中设置数据库作为部署，使用挂载的卷，或者我们可以选择云提供商提供的众多托管数据库服务之一。

为了保持简单且成本效益，在本节中，我们将在 AWS 中设置一个 Aurora Serverless 数据库——这是一个强大的数据库引擎，由于你只需为使用的部分付费，因此具有成本效益，并且非常方便，因为你无需担心管理或扩展数据库。

### 14.7.1 创建 Aurora Serverless 数据库

我们将在 Kubernetes 集群的 VPC 内启动我们的 Aurora 数据库。为了能够在现有的 VPC 内启动数据库，我们需要创建一个*数据库子网组*：VPC 内的一组子网。正如我们在第 14.3 节中学到的，eksctl 将 Kubernetes 集群的 VPC 划分为六个子网：三个公共和三个私有。这六个子网分布在三个*可用区*（AWS 区域内的数据中心）中，每个可用区有一个公共子网和一个私有子网。

在选择数据库子网组的子网时，我们需要考虑以下约束条件：

+   Aurora Serverless 在每个可用区只支持一个子网。

+   在创建数据库子网组时，子网必须全部为私有或公共。⁵

为了安全起见，在数据库子网组中使用私有子网是最佳实践，因为它确保数据库服务器无法从 VPC 外部访问，这意味着外部和未经授权的用户无法直接连接到它。为了找到 VPC 中私有子网的列表，我们首先需要使用以下命令获取 Kubernetes 集群 VPC 的 ID：

```
$ eksctl get cluster --name coffeemesh -o json | \
jq '.[0].ResourcesVpcConfig.VpcId'
```

然后使用以下命令获取 VPC 中私有子网的 ID：

```
$ aws ec2 describe-subnets --filters Name=vpc-id,Values=<vpc_id> \
--output json | jq '.Subnets[] | select(.MapPublicIpOnLaunch == false) | \
.SubnetId'
```

之前的命令列出了 Kubernetes 集群 VPC 中的所有子网，并使用`jq`过滤出公共子网。有了所有这些信息，我们现在可以使用以下命令创建数据库子网组：

```
$ aws rds create-db-subnet-group --db-subnet-group-name \
coffeemesh-db-subnet-group --db-subnet-group-description "Private subnets" \
--subnet-ids "<subnet_id>" "<subnet_id>" "<subnet_id>"
```

如图 14.10 所示，此命令创建了一个名为`coffeemesh-db-subnet-group`的数据库子网组。在运行命令时，请确保将`<subnet_id>`占位符替换为你的私有子网的 ID。我们将在该数据库子网组内部署我们的 Aurora 数据库。

![图 14-10](img/14-10.png)

图 14.10 我们在名为`coffeemesh-db-subnet-group`的数据库子网组中部署了一个 Aurora 数据库。该数据库子网组是在我们的 VPC 的三个私有子网之上创建的，以防止未经授权的访问。

接下来，我们需要创建一个*VPC 安全组*——一组规则，定义了从 VPC 进入和出去的允许流量，以便允许流量访问数据库，使我们的应用程序能够连接到它。以下命令创建了一个名为`db-access`的安全组：

```
$ aws ec2 create-security-group --group-name db-access --vpc-id <vpc-id> \
--description "Security group for db access"
# output:
{
    "GroupId": "sg-00b47703a4299924d"
}
```

在之前的命令中，将`<vpc-id>`替换为你的 Kubernetes 集群 VPC 的 ID。之前命令的输出是我们刚刚创建的安全组的 ID。我们将允许所有 IP 地址在 PostgreSQL 的默认端口（5432）上的流量。由于我们将数据库部署到私有子网中，监听所有 IP 是可行的，但为了额外的安全性，你可能想限制地址范围到你的 Pods 的地址。我们使用以下命令为我们的数据库访问安全组创建一个入站流量规则：

```
$ aws ec2 authorize-security-group-ingress --group-id \
<db-security-group-id> --ip-permissions \ 
'FromPort=5432,IpProtocol=TCP,IpRanges=0.0.0.0/0'
```

在此命令中，将`<db-security-group-id>`替换为你的数据库访问安全组的 ID。

现在我们已经有一个数据库子网组和允许我们的 Pods 连接到它的安全组，我们可以使用子网组在我们的 VPC 内启动一个 Aurora 无服务器集群！运行以下命令来启动 Aurora 无服务器集群：

```
$ aws rds create-db-cluster --db-cluster-identifier coffeemesh-orders-db \
--engine aurora-postgresql --engine-version 10.14 \
--engine-mode serverless \
--scaling-configuration MinCapacity=8,MaxCapacity=64,
➥ SecondsUntilAutoPause=1000,AutoPause=true \
--master-username <username> \
--master-user-password <password> \
--vpc-security-group-ids <security_group_id> \
--db-subnet-group <db_subnet_group_name>
```

让我们仔细看看命令的参数：

+   `--db-cluster-identifier`—数据库集群的名称。我们将集群命名为`coffeemesh-orders-db`。

+   `--engine`—你想要使用的数据库引擎。我们使用兼容 PostgreSQL 的引擎，但如果你更喜欢，也可以选择兼容 MySQL 的引擎。

+   `--engine-version`—你想要使用的 Aurora 引擎版本。我们选择版本 10.14，这是目前 Aurora PostgreSQL 无服务器可用的唯一版本。参见 AWS 文档以了解新版本信息([`mng.bz/gRyn`](http://mng.bz/gRyn))。

+   `--engine-mode`—数据库引擎模式。我们选择无服务器模式以保持示例简单且成本效益高。

+   `--scaling-configuration`—Aurora 集群的自动扩展配置。我们配置集群以最小 Aurora 容量单元（ACU）为 8，最大为 64。每个 ACU 提供大约 2 GB 的内存。我们还配置集群在 1,000 秒无活动后自动缩小到 0 ACU。⁶

+   `--master-username`—数据库主用户的用户名。

+   `--master-user-password`—数据库主用户的密码。

+   `--vpc-security-group-ids`—我们在上一步中创建的数据库访问安全组的 ID。

+   `--db-subnet-group`—我们之前创建的数据库安全组的名称。

运行此命令后，你将获得一个包含数据库详细信息的 JSON 有效负载。要连接到数据库，我们需要有效负载中`DBCluster.Endpoint`属性的值，它表示数据库的主机名。我们将在下一节中使用此值来连接到数据库。

### 14.7.2 Kubernetes 中的秘密管理

为了将我们的服务连接到数据库，我们需要一种安全的方式来传递连接凭据。在 Kubernetes 中管理敏感信息的原生方式是使用 Kubernetes 机密。这样，我们就避免了通过代码或通过我们的镜像构建暴露敏感信息。在本节中，你将学习如何安全地管理 Kubernetes 机密。

AWS EKS 提供两种安全的方式来管理 Kubernetes 机密：我们可以使用 AWS Secrets & Configuration Provider for Kubernetes，⁷，或者我们可以使用 AWS 密钥管理服务（KMS）通过封装加密来保护我们的机密。在本节中，我们将使用封装加密来保护我们的机密。⁸

正如你在图 14.11 中看到的，封装加密是指使用数据加密密钥（DEK）加密你的数据，然后使用密钥加密密钥（KEK）加密 DEK 的做法。⁹ 这听起来很复杂，但由于 AWS 为我们承担了繁重的工作，所以使用起来很简单。

![图 14.11](img/14-11.png)

图 14.11 封装加密是指使用数据加密密钥（DEK）加密数据，然后使用密钥加密密钥（KEK）加密 DEK 的做法。

要使用封装加密，首先我们需要生成一个 AWS KMS 密钥。你可以使用以下命令来创建密钥：

```
$ aws kms create-key
```

此命令的输出是一个包含新创建密钥元数据的有效负载。从这个有效负载中，我们想要使用 `KeyMetadata.Arn` 属性，它表示密钥的 ARN，或 Amazon 资源名称。下一步是在我们的 Kubernetes 集群中使用 eksctl 启用机密加密：`

```
$ eksctl utils enable-secrets-encryption --cluster coffeemesh \
--key-arn=<key_arn> --region <aws_region>
```

确保将 `<key_arn>` 替换为你的 KMS 密钥的 ARN，并将 `<aws_region>` 替换为你部署 Kubernetes 集群的区域。由前面的命令触发的操作可能需要最多 45 分钟才能完成。该命令会一直运行，直到集群创建完成，所以只需等待它完成。一旦完成，我们就可以创建 Kubernetes 机密。让我们创建一个表示数据库连接字符串的机密。数据库连接字符串具有以下结构：

```
<engine>://<username>:<password>@<hostname>:<port>/<database_name>
```

让我们看看连接字符串的每个组成部分：

+   `engine`—数据库引擎，例如，`postgresql`。

+   `username`—我们在创建数据库时选择的用户名。

+   `password`—我们在创建数据库时选择的密码。

+   `hostname`—数据库的主机名，这是我们上一节中从 `aws rds create-db-cluster` 命令返回的有效负载的 `DBCluster.Endpoint` 属性中获得的。

+   `port`—数据库运行在上的端口。每个数据库都有自己的默认端口，例如 PostgreSQL 的 5432 和 MySQL 的 3306。

+   `database_name`—我们要连接到的数据库的名称。在 PostgreSQL 中，默认数据库的名称为 `postgres`。

例如，对于 PostgreSQL 数据库，一个典型的连接字符串看起来像这样：

```
postgresql://username:password@localhost:5432/postgres
```

要将数据库连接字符串作为 Kubernetes 机密存储，我们运行以下命令：

```
$ kubectl create secret generic -n orders-service db-credentials \
--from-literal=DB_URL=<connection_string>
```

之前的命令在 `orders-service` 命名空间内创建了一个名为 `db-credentials` 的密钥对象。要获取此密钥对象的详细信息，您可以运行以下命令：

```
$ kubectl get secret db-credentials -n orders-service -o json
# output:
{
    "apiVersion": "v1",
    "data": {
        "DB_URL": "cG9zdGdyZXNxbDovL3VzZXJuYW1lOnBhc3N3b3JkQGNvZmZlZW1lc2gtZGIuY2x1c3Rlci1jYn
➥ Y0YWhnc2JjZWcuZXUtd2VzdC0xLnJkcy5hbWF6b25hd3MuY29tOjU0MzIvcG9zdGdyZXM="
    },
    "kind": "Secret",
    "metadata": {
        "creationTimestamp": "2021-11-19T15:21:42Z",
        "name": "db-credentials",
        "namespace": "orders-service",
        "resourceVersion": "599258",
        "uid": "d2c210e7-c61c-46b7-9f43-9407766e147c"
    },
    "type": "Opaque"
}
```

密钥列在有效载荷的 `data` 属性下，并且它们是 Base64 编码的。要获取它们的值，您可以运行以下命令：

```
$ echo <DB_URL> | base64 --decode
```

其中 `<DB_URL>` 是 `DB_URL` 键的 Base64 编码值。

为了使密钥对订单服务可用，我们需要更新订单服务部署以消费密钥并将其作为环境变量公开。

列表 14.4 在部署中作为环境变量消费密钥

```
# file: orders-service-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: orders-service
  namespace: orders-service
  labels:
    app: orders-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: orders-service
  template:
    metadata:
      labels:
        app: orders-service
    spec:
      containers:
      - name: orders-service
        image: 
➥ <aws_account_id>.dkr.ecr.<aws_region>.amazonaws.com/coffeemesh-orders:1.0
        ports:
          - containerPort: 8000
        imagePullPolicy: Always
        envFrom:                       ①
          - secretRef:                 ②
              name: db-credentials ③
```

① Pod 的环境配置

② 配置用于识别密钥

③ 环境必须从名为 db-credentials 的密钥中加载。

让我们通过运行以下命令来应用这些更改：

```
$ kubectl apply -f orders-service-deployment.yaml
```

我们的服务现在可以连接到数据库了！我们几乎完成了。最后一步是应用数据库迁移，我们将在下一节中完成。

### 14.7.3 运行数据库迁移并将我们的服务连接到数据库

我们的数据库正在运行，现在我们可以将其与订单服务连接起来。然而，在我们能够创建记录和运行查询之前，我们必须确保数据库具有预期的模式。正如我们在第七章中看到的，创建数据库模式的过程被称为迁移。我们的应用程序的迁移可以在 migrations 文件夹下找到。在本节中，我们将对 Aurora Serverless 数据库运行迁移。

在上一节中，我们将 Aurora 数据库部署到了我们的私有子网中，这意味着我们无法直接访问数据库来运行迁移。我们有两个主要选项来连接到数据库：通过堡垒机服务器连接或创建一个应用迁移的 Kubernetes Job。由于我们正在使用 Kubernetes，并且我们的集群已经启动并运行，因此使用 Kubernetes Job 对我们来说是一个合适的选项。

定义 A *堡垒机服务器* 是一个允许您与私有网络建立安全连接的服务器。通过连接到堡垒机服务器，您能够访问私有网络中的其他服务器。

要创建 Kubernetes 作业，我们首先需要为运行数据库迁移创建一个 Docker 镜像。创建一个名为 migrations.dockerfile 的文件，并将列表 14.5 的内容复制到其中。此 Dockerfile 安装了生产环境和开发依赖项，并将迁移和 Alembic 配置复制到容器中。正如我们在第七章中看到的，我们使用 Alembic 来管理我们的数据库迁移。此容器的命令是单次执行的 `alembic upgrade`。

列表 14.5 数据库迁移作业的 Dockerfile

```
# file: migrations.dockerfile

FROM python:3.9-slim

RUN mkdir -p /orders/orders

WORKDIR /orders

RUN pip install -U pip && pip install pipenv

COPY Pipfile Pipfile.lock /orders/

RUN pipenv install --dev --system --deploy

COPY orders/repository /orders/orders/repository/
COPY migrations /orders/migrations
COPY alembic.ini /orders/alembic.ini

ENV PYTHONPATH=/orders                 ①

CMD ["alembic", "upgrade", "heads"]
```

① 我们设置了 PYTHONPATH 环境变量。

构建 Docker 镜像，请运行以下命令：

```
$ docker build -t 
➥ <aws_account_number>.dkr.ecr.<aws_region>.amazonaws.com/coffeemesh-
➥ orders-migrations:1.0 -f migrations.dockerfile .
```

我们将镜像命名为`coffeemesh-orders-migrations`并标记为版本 1.0。确保你用你的 AWS 账户 ID 替换`<aws_account_id>`，并用你想要存储 Docker 构建的区域的名称替换`<aws_region>`。在我们将镜像推送到容器注册库之前，我们需要创建一个仓库：

```
$ aws ecr create-repository --repository-name coffeemesh-orders-migrations
```

现在让我们将镜像推送到容器注册库：

```
$ docker push 
➥ <aws_account_id>.dkr.ecr.<aws_region>.amazonaws.com/coffeemesh-orders-
➥ migrations:1.0
```

如果你的 ECR 凭证已过期，你可以通过再次运行以下命令来刷新它们：

```
$ aws ecr get-login-password --region <aws_region> | docker login \
--username AWS --password-stdin \
<aws_account_id>.dkr.ecr.<aws_region>.amazonaws.com
```

现在我们已经准备好了镜像，我们需要创建一个 Kubernetes Job 对象。我们使用清单文件来创建 Job。创建一个名为 orders-migrations-job.yaml 的文件，并将列表 14.6 的内容复制进去。列表 14.6 使用 batch/v1 API 定义了一个类型为`Job`的 Kubernetes 对象。就像我们在上一节为订单服务所做的那样，我们通过在容器的定义中使用`envFrom`属性加载`db-credentials`机密来在环境中公开数据库连接字符串。我们还设置了`ttlSecondsAfterFinished`参数为 30 秒，这控制了 Pod 在完成作业后将在`orders-service`命名空间中持续多长时间。

列表 14.6 创建数据库迁移 Job

```
# file: orders-migrations-job.yaml

apiVersion: batch/v1
kind: Job
metadata:
  name: orders-service-migrations
  namespace: orders-service
  labels:
    app: orders-service
spec:
  ttlSecondsAfterFinished: 30      ①
  template:
    spec:
      containers:
      - name: orders-service-migrations
        image: 
➥ <aws_account_id>.dkr.ecr.<aws_region>.amazonaws.com/coffeemesh-orders-
➥ migrations:1.0
        imagePullPolicy: Always
        envFrom:
          - secretRef:
              name: db-credentials
      restartPolicy: Never
```

① Pod 必须在完成 30 秒后删除。

让我们通过运行以下命令来创建 Job：

```
$ kubectl apply -f orders-migrations-job.yaml
```

直到`job`的 Pod 启动并运行，可能需要几秒钟。你可以通过运行以下命令来检查其状态：

```
$ kubectl get pods -n orders-service
```

一旦 Pod 的状态为`Running`或`Completed`，你可以通过运行以下命令来检查`job`的日志：

```
$ kubectl logs -f jobs/orders-service-migrations -n orders-service
```

以这种方式查看 Pod 的日志对于检查进程的进展和发现执行中出现的任何问题很有用。由于迁移作业是短暂的，完成后会删除，确保你在进程运行时检查日志。一旦迁移作业完成，数据库最终就绪，可以使用了！我们终于可以与订单服务交互了——我们一直等待的时刻！我们的服务现在已准备好使用。下一节将解释我们需要进行的另一个更改以完成部署。

## 14.8 使用 ALB 的主机名更新 OpenAPI 规范

现在我们的服务已经就绪，数据库已部署并配置，是时候玩转应用程序了！在第二章和第六章，我们学习了如何使用 Swagger UI 与我们的 API 交互。为了在我们的部署中使用 Swagger UI，我们需要更新 API 规范，以包含我们 Kubernetes 集群 ALB 的主机名。在本节中，我们更新了订单的 API 规范，创建了一个新的部署，并对其进行了测试。

列表 14.7 将 ALB 的主机名添加为服务器

```
# file: oas.yaml

openapi: 3.0.0

info:
  title: Orders API
  description: API that allows you to manage orders for CoffeeMesh
  version: 1.0.0

servers:
  - url: <alb-hostname>
    description: ALB's hostname
  - url: https://coffeemesh.com
    description: main production server
  - url: https://coffeemesh-staging.com
    description: staging server for testing purposes only
  - url: http://localhost:8000
    description: URL for local testing

...
```

在列表 14.8 中，将`<alb-hostname>`替换为你自己的 ALB 的主机名。正如我们在 14.6 节中学到的，你可以通过运行以下命令来获取 ALB 的主机名：

```
$ kubectl get ingress/orders-service-ingress -n orders-service -o json | \
jq '.status.loadBalancer.ingress[0].hostname'
# output:
# "k8s-ordersse-ordersse-8cf837ce7a-1036161040.<aws_region>.elb.amazonaws.com"
```

现在我们需要重新构建我们的 Docker 镜像：

```
$ docker build -t 
<aws_account_number>.dkr.ecr.<aws_region>.amazonaws.com/coffeemesh-
➥ orders:1.1 .
```

然后，我们将新的构建发布到 AWS ECR：

```
$ docker push 
<aws_account_number>.dkr.ecr.<aws_region>.amazonaws.com/coffeemesh-
➥ orders:1.1
```

接下来，我们需要更新订单服务部署清单。

列表 14.8 声明部署清单

```
# file: orders-service-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: orders-service
  namespace: orders-service
  labels:
    app: orders-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: orders-service
  template:
    metadata:
      labels:
        app: orders-service
    spec:
      containers:
      - name: orders-service
        image: 
➥ <aws_account_id>.dkr.ecr.<aws_region>.amazonaws.com/coffeemesh-orders:1.1
        ports:
          - containerPort: 8000
        imagePullPolicy: Always
```

最后，通过运行以下命令应用新的部署配置：

```
$ kubectl apply -f orders-service-deployment.yaml
```

通过运行以下命令来监控部署：

```
kubectl get pods -n orders-service
```

一旦旧节点终止，新节点启动并运行，通过在浏览器中粘贴 ALB 的主机名并访问 `/docs/orders` 页面来加载订单服务的 Swagger UI。你可以使用在 2 章和 6 章中学到的方法来玩转 API：创建订单、修改它们，并从服务器获取它们的详细信息。

旅程终于完成了！如果你能够跟到这里，并且成功地将你的 Kubernetes 集群启动并运行，请接受我最真诚的祝贺！你已经成功了！图 14.12 展示了你本章部署的架构的高级概述。

![图片](img/14-12.png)

图 14.12 本章部署的架构高级概述

本章对 Kubernetes 的概述是简要的，但足以了解 Kubernetes 的工作原理，并且足以在你的生产环境中启动并运行一个集群。如果你正在使用或打算使用 Kubernetes，我强烈建议你继续阅读有关这项技术的信息。你可以查看我在本章中引用的所有参考文献，我想要补充的是 Marko Lukša 的基础书籍 *Kubernetes in Action*（第 2 版，Manning，预计 2023 年出版）。

在下一节中，我们将删除本章中创建的所有资源。如果你不想支付不必要的费用，请不要错过！

## 14.9 删除 Kubernetes 集群

本节解释了如何删除本章中创建的所有资源。这一步至关重要，以确保你在完成示例工作后不会为 Kubernetes 集群付费。如图 14.13 所示，我们的一些资源之间存在依赖关系。为了成功删除所有资源，我们必须按照它们的依赖关系反向删除。例如，数据库集群依赖于数据库子网组，数据库子网组依赖于 VPC 子网，VPC 子网依赖于 VPC。在这种情况下，我们将首先删除数据库集群，最后一步将删除 VPC。

![图片](img/14-13.png)

图 14.13 我们堆栈中的资源之间存在依赖关系。依赖关系的方向由箭头的方向表示。为了删除资源，我们首先删除那些没有箭头指向的资源。

使用以下命令删除数据库集群：

```
$ aws rds delete-db-cluster --db-cluster-identifier coffeemesh-db \
--skip-final-snapshot
```

`--skip-final-snapshot` 标志指示命令在删除前不要创建数据库快照。删除数据库需要几分钟时间。一旦删除完成，我们可以使用以下命令删除数据库子网组：

```
$ aws rds delete-db-subnet-group --db-subnet-group-name \
coffeemesh-db-subnet-group
```

接下来，让我们删除 AWS 负载均衡器控制器。删除 AWS 负载均衡器控制器是一个两步过程：首先我们使用`helm`卸载控制器，然后我们删除在安装控制器时创建的 ALB。要删除 ALB，我们需要它的 URL，所以让我们首先获取这个值（确保在用`helm`卸载之前运行此步骤）：

```
$ kubectl get ingress/orders-service-ingress -n orders-service -o json | \
jq '.status.loadBalancer.ingress[0].hostname'
# output: "k8s-ordersse-ordersse-8cf837ce7a-
➥ 1036161040.<aws_region>.elb.amazonaws.com"
```

现在，让我们使用以下命令卸载控制器：

```
$ helm uninstall aws-load-balancer-controller -n kube-system
```

运行此命令后，我们需要删除 ALB。要删除 ALB，我们需要找到它的 ARN。我们将使用 AWS CLI 列出我们账户中的负载均衡器，并通过它们的 DNS 名称过滤它们。以下命令获取与 ALB 的 URL 匹配的负载均衡器的 ARN，这是我们之前获得的：

```
$ aws elbv2 describe-load-balancers | jq '.LoadBalancers[] | \
select(.DNSName == "<load_balancer_url>") | .LoadBalancerArn'
# output: "arn:aws:elasticloadbalancing:<aws_region>:<aws_account_id>:
➥ loadbalancer/app/k8s-ordersse-ordersse-8cf837ce7a/cf708f97c2485719"
```

确保将`<load_balancer_url>`替换为你的负载均衡器的 URL，这是我们之前步骤中获得的。此命令为我们提供了负载均衡器的 ARN，我们可以用它来删除它：

```
$ aws elbv2 delete-load-balancer --load-balancer-arn "<load_balancer_arn>"
```

现在，我们可以使用以下命令删除 Kubernetes 集群：

```
$ eksctl delete cluster coffeemesh
```

最后，让我们删除我们之前创建的用于加密 Kubernetes 机密的 KMS 密钥。要删除密钥，我们运行以下命令：

```
$ aws kms schedule-key-deletion --key-id <key_id>
```

其中`<key_id>`是我们之前创建的密钥的 ID。

## 摘要

+   Kubernetes 是一个容器编排工具，它正在成为大规模部署微服务的标准。使用 Kubernetes 可以帮助我们在云提供商之间迁移，同时保持对服务的一致接口。

+   三个主要的托管 Kubernetes 服务是 Google 的 Kubernetes Engine (GKE)、Azure 的 Kubernetes Service (AKS)和 AWS 的 Elastic Kubernetes Service (EKS)。在本章中，我们学习了如何使用 EKS 部署 Kubernetes 集群，EKS 是最广泛采用的 Kubernetes 托管服务。

+   我们可以使用控制台、CloudFormation 或 eksctl 命令行工具在 AWS 中部署一个 Kubernetes 集群。在本章中，我们使用了 eksctl CLI，因为它是 AWS 推荐的方式来管理 Kubernetes 集群。

+   为了使我们的 Kubernetes 集群可以从互联网访问，我们使用一个 ingress 控制器，例如 AWS 负载均衡器控制器。

+   要将微服务部署到 Kubernetes 集群，我们创建以下资源：

    +   一个`Deployment`，它管理 pods 的期望状态，运行 Docker 构建的过程

    +   一个允许我们将应用程序作为 Web 服务公开的`Service`

    +   一个绑定到 ingress 控制器（AWS 负载均衡器控制器）的`Ingress`对象，它将流量转发到服务

+   Aurora Serverless 是一个强大的数据库引擎，是微服务的便捷选择。使用 Aurora Serverless，你只需为使用的部分付费，你不需要担心数据库的扩展，从而降低你的成本和管理时间。

+   为了在 Kubernetes 中安全地将敏感配置细节提供给应用程序，我们使用 Kubernetes 机密。使用 EKS，我们有两种策略来安全地管理 Kubernetes 机密：

    +   使用 AWS Secrets & Configuration Provider for Kubernetes

    +   在 Kubernetes 中使用密钥管理与 AWS 密钥管理服务结合使用

* * *

¹ 关于 GKE、AKS 和 EKS 的快速比较，请参阅 Alexander Postasnick 的文章“AWS vs EKS vs GKE: Managed Kubernetes Services Compared”，发布于 2021 年 6 月 9 日，[`acloudguru.com/blog/engineering/aks-vs-eks-vs-gke-managed-kubernetes-services-compared`](https://acloudguru.com/blog/engineering/aks-vs-eks-vs-gke-managed-kubernetes-services-compared)。

² Flexera，“2022 云状态报告”（第 52-53 页），[`info.flexera.com/CM-REPORT-State-of-the-Cloud`](https://info.flexera.com/CM-REPORT-State-of-the-Cloud)。

³ AWS 负载均衡器控制器是一个托管在 GitHub 上的开源项目 ([`github.com/kubernetes-sigs/aws-load-balancer-controller/`](https://github.com/kubernetes-sigs/aws-load-balancer-controller/))。该项目最初由 Ticketmaster 和 CoreOS 创建。

⁴ 若想了解更多关于标签及其使用方法的信息，请参阅官方文档，[`kubernetes.io/docs/concepts/overview/working-with-objects/labels/`](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)，以及 Zane Hitchcox 的文章“matchLabels, Labels, and Selectors Explained in Detail, for Beginners”，发表于 *Medium*（2018 年 7 月 15 日），[`medium.com/@zwhitchcox/matchlabels-labels-and-selectors-explained-in-detail-for-beginners-d421bdd05362`](https://medium.com/@zwhitchcox/matchlabels-labels-and-selectors-explained-in-detail-for-beginners-d421bdd05362)。

⁵ 关于此点的更多信息，请参阅官方 AWS 文档：[`docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.WorkingWithRDSInstanceinaVPC.html`](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.WorkingWithRDSInstanceinaVPC.html)。

⁶ 若想了解更多关于 Aurora Serverless 的工作原理和自动扩展配置参数的信息，请参阅官方文档：[`docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.html`](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.html)。

⁷ 您可以通过 Tracy Pierce 的文章“如何使用 AWS Secrets & Configuration Provider 与您的 Kubernetes Secrets Store CSI 驱动程序一起使用”了解更多关于此选项的信息，[`aws.amazon.com/blogs/security/how-to-use-aws-secrets-configuration-provider-with-kubernetes-secrets-store-csi-driver/`](https://aws.amazon.com/blogs/security/how-to-use-aws-secrets-configuration-provider-with-kubernetes-secrets-store-csi-driver/)。

⁸ 安全管理 Kubernetes 是一个庞大且重要的主题，若想了解更多，可以查看 Alex Soto Bueno 和 Andrew Block 的著作 *Securing Kubernetes Secrets*（Manning，2022），[`livebook.manning.com/book/securing-kubernetes-secrets/chapter-4/v-3/point-13495-119-134-1`](https://livebook.manning.com/book/securing-kubernetes-secrets/chapter-4/v-3/point-13495-119-134-1)。

⁹ 同上。
