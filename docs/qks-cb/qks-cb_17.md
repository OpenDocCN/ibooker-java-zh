# 附录 A. Minikube

本书中涉及 Kubernetes 集群的所有示例都在 minikube 中进行了测试；然而，它们也应该在任何其他 Kubernetes 集群中正常工作。

Minikube 是一个工具，使得在本地而非远程 Kubernetes 集群中运行 Kubernetes 变得非常简单。

在本书中，使用了 minikube 1.7.3 和 Kubernetes 1.17.3；但是，由于没有使用高级技术，所以其他版本也应该可以。Minikube 需要安装一个虚拟化程序。我们建议您使用 VirtualBox 虚拟化程序。根据我们的经验，这是运行 minikube 最便携和稳定的方式。

安装 minikube、VirtualBox 和 `kubectl` 的方式可能取决于您正在运行的系统，因此我们提供了安装每个组件的说明链接：

+   [VirtualBox](https://oreil.ly/KU2vk)

+   [Minikube](https://oreil.ly/gth-J)

+   [kubectl](https://oreil.ly/FpZzN)

安装完所有软件后，您可以通过打开一个终端窗口并运行以下命令来启动 minikube：

```java
minikube start --vm-driver=virtualbox --memory='8192mb' \
 --kubernetes-version='v1.17.3'

ߙ䠠[serverless] minikube v1.7.3 on Darwin 10.15.3
✨  Using the virtualbox driver based on user configuration
⌛  Reconfiguring existing host ...
ߔ䠠Starting existing virtualbox VM for "default" ...
ߐ㠠Preparing Kubernetes v1.17.3 on Docker 19.03.6 ...
ߚࠠLaunching Kubernetes ...
ߌEnabling addons: dashboard, default-storageclass, storage-provisioner
ߏ䠠Done! kubectl is now configured to use "default"
```

然后，配置 `docker` CLI 使用 minikube 的 docker 主机：

```java
eval $(minikube docker-env)
```

然后，任何使用 `docker` 执行的操作，如 `docker build` 或 `docker run`，都将在 minikube 集群内进行。
