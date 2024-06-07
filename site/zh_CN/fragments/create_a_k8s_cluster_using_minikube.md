### 使用 minikube 创建 K8s 集群

我们建议使用 [minikube](https://minikube.sigs.k8s.io/docs/) 在 K8s 上安装 Milvus，这是一个允许您在本地运行 K8s 的工具。

<div class="alert note">
minikube 仅适用于测试环境。不建议在生产环境中以这种方式部署 Milvus 分布式集群。
</div>

#### 1. 安装 minikube

查看 [安装 minikube](https://minikube.sigs.k8s.io/docs/start/) 获取更多信息。

#### 2. 使用 minikube 启动 K8s 集群

安装 minikube 后，运行以下命令启动 K8s 集群。

```
$ minikube start
```

#### 3. 检查 K8s 集群状态

运行 `$ kubectl cluster-info` 来检查您刚刚创建的 K8s 集群的状态。确保可以通过 `kubectl` 访问 K8s 集群。如果您尚未在本地安装 `kubectl`，请参阅 [在 minikube 中使用 kubectl](https://minikube.sigs.k8s.io/docs/handbook/kubectl/)。