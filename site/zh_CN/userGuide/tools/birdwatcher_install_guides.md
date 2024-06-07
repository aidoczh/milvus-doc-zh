---
id: birdwatcher_install_guides.md
summary: 学习如何安装 Birdwatch 以调试 Milvus。
title: 安装 Birdwatcher
---

# 安装 Birdwatcher

本页面演示了如何安装 Birdwatcher。

## 本地安装

如果您已经使用 Docker 安装了 Milvus Standalone [请参考此处](install_standalone-docker.md)，您最好下载并安装构建好的二进制文件，将 Birdwatcher 安装为一个普通的 Go 模块，或者从源代码构建 Birdwatcher。

- 将其安装为一个普通的 Go 模块。

    ```shell
    git clone https://github.com/milvus-io/birdwatcher.git
    cd birdwatcher
    go install github.com/milvus-io/birdwatcher
    ```

    然后您可以按照以下方式运行 Birdwatcher：

    ```shell
    go run main.go
    ```

- 从源代码构建。

    ```shell
    git clone https://github.com/milvus-io/birdwatcher.git
    cd birdwatcher
    go build -o birdwatcher main.go
    ```

    然后您可以按照以下方式运行 Birdwatcher：

    ```shell
    ./birdwatcher
    ```

- 下载已构建好的二进制文件

    首先，打开[最新发布页面](https://github.com/milvus-io/birdwatcher/releases/latest)，找到准备好的二进制文件。

    ```shell
    wget -O birdwatcher.tar.gz \
    https://github.com/milvus-io/birdwatcher/releases/download/latest/birdwatcher_<os>_<arch>.tar.gz
    ```

    然后您可以解压缩 tar 文件并按以下方式使用 Birdwatcher：

    ```shell
    tar -xvzf birdwatcher.tar.gz
    ./birdwatcher
    ```

## 作为 Kubernetes Pod 安装

如果您已经安装了 Milvus Standalone [使用 Helm charts](install_standalone-helm.md) 或 [Milvus Operator](install_standalone-operator.md)，或者 Milvus Cluster [使用 Helm charts](install_cluster-helm.md) 或 [Milvus Operator](install_cluster-milvusoperator.md)，建议您将 Birdwatcher 安装为 Kubernetes Pod。

### 准备 deployment.yml

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: birdwatcher
spec:
  selector:
    matchLabels:
      app: birdwatcher
  template:
    metadata:
      labels:
        app: birdwatcher
    spec:
      containers:
      - name: birdwatcher
        image: milvusdb/birdwatcher
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
```

<div class="alert note">

如果 DockerHub 上的镜像不是最新的，您可以使用提供的 Dockerfile 构建 Birdwatcher 的镜像，方法如下：

```shell
git clone https://github.com/milvus-io/birdwatcher.git
cd birdwatcher
docker build -t milvusdb/birdwatcher .
```

要部署本地构建的镜像，您需要在上述规范中添加 `imagePullPolicy` 并将其设置为 `Never`。

```yaml
...
      - name: birdwatcher
        image: milvusdb/birdwatcher
        imagePullPolicy: Never
...
```

</div>

### 应用 deployment.yml

将上述 YAML 保存到一个文件中并命名为 `deployment.yml`，然后运行以下命令

```shell
kubectl apply -f deployment.yml
```