# 附录 C. Knative

在 第十章 中，你需要访问一个 Kubernetes 集群 —— 它可以是 minikube 安装或其他任何类型。但是，你还需要安装 Knative Serving 来运行 Knative 示例。对于本书而言，Kourier 用作 Knative 的 Ingress。

本书中使用的版本为 minikube 1.7.3、Kubernetes 1.17.3、Knative 0.13.0 和 Kourier 0.3.12。

要安装 Knative Serving，需要运行以下命令：

```java
kubectl apply -f \
 https://github.com/knative/serving/releases/download/v0.13.0/serving-core.yaml
kubectl apply -f \
 https://raw.githubusercontent.com/3scale/kourier/v0.3.12/deploy/\
 kourier-knative.yaml
```

配置 Knative Serving 使用正确的 `ingress.class`：

```java
kubectl patch configmap/config-network \
 -n knative-serving \
 --type merge \
 -p '{"data":{"clusteringress.class":"kourier.ingress.networking.knative.dev",
 "ingress.class":"kourier.ingress.networking.knative.dev"}}'
```

设置你所需的域名；在本例中，使用 `127.0.0.1` 是因为它在 minikube 中运行：

```java
kubectl patch configmap/config-domain \
 -n knative-serving \
 --type merge \
 -p '{"data":{"127.0.0.1.nip.io":""}}'
```

现在，你已经准备好开始部署 Knative 服务。
