---
id: gcp.md
title: 在 GCP 上部署 Milvus 集群
related_key: cluster
summary: 学习如何在 GCP 上部署 Milvus 集群。
---

# 在 GCP 上部署 Milvus 集群

Milvus 是一个云原生向量数据库，可以部署在各种云环境中。本指南将指导您如何在 Google Cloud Platform（GCP）上设置 Milvus。

![在 GCP 上部署 Milvus 集群](../../../../../assets/gcp-networking.png)

## 开始之前

要在 GCP 上部署 Milvus，请确保：

- 您的 GCP 帐户中已经存在一个项目。

  要创建项目，请参阅 [创建和管理项目](https://cloud.google.com/resource-manager/docs/creating-managing-projects)。本指南中使用的项目名称为 **milvus-testing-nonprod**。

- 您已经在本地安装了 [gcloud CLI](https://cloud.google.com/sdk/docs/quickstart#installing_the_latest_version)、[kubectl](https://kubernetes.io/docs/tasks/tools/) 和 [Helm](https://helm.sh/docs/intro/install/)，或者决定改用基于浏览器的 [Cloud Shell](https://cloud.google.com/shell)。

- 您已经使用您的 GCP 帐户凭据 [初始化了 gcloud CLI](https://cloud.google.com/sdk/docs/install-sdk#initializing_the)。

## 设置网络

为了确保 Milvus 的安全性，您需要在 GCP 项目中创建一个逻辑隔离的虚拟网络。以下命令将创建一个 VPC。

```bash
gcloud compute networks create milvus-network \
    --project=milvus-testing-nonprod \
    --subnet-mode=auto \
    --mtu=1460 \
    --bgp-routing-mode=regional
```

为了方便您的工作，您还需要设置几个防火墙规则，以允许 ICMP、RDP 和 SSH 上的外部流量，以及 VPC 内部的流量。
```bash
gcloud compute firewall-rules create milvus-network-allow-icmp \
    --project=milvus-testing-nonprod \
    --network=projects/milvus-testing-nonprod/global/networks/milvus-network \
    --description="允许来自任何源的 ICMP 连接到网络上的任何实例。" \
    --direction=INGRESS \
    --priority=65534 \
    --source-ranges=0.0.0.0/0 \
    --action=ALLOW \
    --rules=icmp

gcloud compute firewall-rules create milvus-network-allow-internal \
    --project=milvus-testing-nonprod \
    --network=projects/milvus-testing-nonprod/global/networks/milvus-network \
    --description="允许来自网络 IP 范围内任何源的连接到网络上的任何实例，使用所有协议。" \
    --direction=INGRESS \
    --priority=65534 \
    --source-ranges=10.128.0.0/9 \
    --action=ALLOW --rules=all

gcloud compute firewall-rules create milvus-network-allow-rdp \
    --project=milvus-testing-nonprod \
    --network=projects/milvus-testing-nonprod/global/networks/milvus-network \
    --description="允许来自任何源的 RDP 连接到网络上的任何实例，使用端口 3389。" \ --direction=INGRESS \
    --priority=65534 \
    --source-ranges=0.0.0.0/0 \
    --action=ALLOW \
    --rules=tcp:3389

gcloud compute firewall-rules create milvus-network-allow-ssh \
    --project=milvus-testing-nonprod \
    --network=projects/milvus-testing-nonprod/global/networks/milvus-network \
    --description="允许来自任何源的 TCP 连接到网络上的任何实例，使用端口 22。" \ --direction=INGRESS \
    --priority=65534 \
    --source-ranges=0.0.0.0/0 \
    --action=ALLOW \
    --rules=tcp:22
```
最后，您需要允许将来创建的 Milvus 实例在端口 **19530** 上接收流量。

```bash
gcloud compute firewall-rules create allow-milvus-in \
    --project=milvus-testing-nonprod  \
    --description="允许在端口19530上为 Milvus 接收入站流量" \
    --direction=INGRESS \
    --priority=1000 \
    --network=projects/milvus-testing-nonprod/global/networks/milvus-network \
    --action=ALLOW \
    --rules=tcp:19530 \
    --source-ranges=0.0.0.0/0
```

## 部署 Kubernetes 集群

在本指南中，我们将使用 Google Kubernetes Engine (GKE) 服务在 **us-west1-a** 区域为 Kubernetes 集群提供两个节点。每个节点是运行 **COS_CONTAINERD** 镜像的 **e2-standard-4** Compute Engine 虚拟机。

<div class="alert note">

建议您使用提供至少 16 GB 内存的机器类型，以确保服务稳定性。

</div>

```bash
gcloud container clusters create "milvus-cluster-1" \
    --project "milvus-testing-nonprod" \
    --zone "us-west1-a" \
    --workload-pool "milvus-testing-nonprod.svc.id.goog" \
    --no-enable-basic-auth \
    --cluster-version "1.27.3-gke.100" \
    --release-channel "regular" \
    --machine-type "c2-standard-4" \
    --image-type "COS_CONTAINERD" \
    --disk-type "pd-standard" \
    --disk-size "100" \
    --max-pods-per-node "110" \
    --num-nodes "3" \
    --enable-ip-alias \
    --network "projects/milvus-testing-nonprod/global/networks/milvus-network" \
    --subnetwork "projects/milvus-testing-nonprod/regions/us-west1/subnetworks/milvus-network"
```

Kubernetes 集群需要一段时间才能启动。一旦集群准备就绪，使用以下命令获取其凭据，以便您可以在终端中运行 `kubectl` 命令远程与集群通信。

```bash
gcloud container clusters get-credentials milvus-cluster-1 --zone "us-west1-a"
```

## 使用 Google Cloud Storage (GCS) 作为外部对象存储

- 创建存储桶。
```bash
gcloud storage buckets create gs://milvus-testing-nonprod --project=milvus-testing-nonprod --default-storage-class=STANDARD --location=us-west1 --uniform-bucket-level-access
```
- 生成用户访问密钥和秘密密钥，您应该转到项目的存储页面。在仪表板的左侧边栏中，点击 Google Cloud Storage，然后点击设置。选择 INTEROPERABILITY 选项卡。如果尚未启用，点击互操作性访问。然后点击创建密钥按钮。

![GCP 用户账户的访问密钥](../../../../../assets/access_key.jpg)

- 添加 values.yaml
```yaml
cluster:
    enabled: true

service:
    type: LoadBalancer

minio:
    enabled: false

externalS3:
    enabled: true
    host: storage.googleapis.com
    port: 443
    rootPath: milvus/my-release
    bucketName: milvus-testing-nonprod
    cloudProvider: gcp
    useSSL: true
    accessKey: "<access-key>"
    secretKey: "<secret-key>"
```

## 部署 Milvus
现在 Kubernetes 集群已准备就绪。让我们立即部署 Milvus。

```bash
helm repo add milvus https://zilliztech.github.io/milvus-helm/
helm repo update
helm install -f values.yaml my-release milvus/milvus
```

在上述命令中，我们将 Milvus Helm charts 仓库添加到本地，并更新仓库以获取最新的 charts。然后我们安装了一个 Milvus 实例，并命名为 **my-release**。

请注意配置文件 `service.type` 的值，该值表示我们希望通过 Layer-4 负载均衡器公开 Milvus 实例。

如果您希望通过 Layer-7 负载均衡器公开 Milvus 实例，请参阅[此文档](gcp_layer7.md)。

## 验证部署

一旦所有的 Pod 都在运行中，运行以下命令以获取外部 IP 地址。

```bash
kubectl get services|grep my-release-milvus|grep LoadBalancer|awk '{print $4}'
```

## 你好，Milvus

请参考[Hello Milvus](https://milvus.io/docs/example_code.md)，将主机值更改为外部 IP 地址，然后运行代码。