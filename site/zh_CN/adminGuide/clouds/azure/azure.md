---
id: azure.md
title: 在 Microsoft Azure 上使用 Kubernetes 部署 Milvus
related_key: cluster
summary: 学习如何在 Azure 上部署 Milvus 集群。
---

# 在 Azure 上使用 AKS 部署 Milvus

本主题介绍如何使用 [Azure Kubernetes 服务](https://azure.microsoft.com/en-us/services/kubernetes-service/#overview) (AKS) 和 [Azure 门户](https://portal.azure.com) 配置和创建集群。

## 先决条件

确保您的 Azure 项目已正确设置，并且您可以访问要使用的资源。如果您对自己的访问权限不确定，请联系管理员。

## 软件要求
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli#install)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Helm](https://helm.sh/docs/intro/install/)

或者，您可以使用预安装了 Azure CLI、kubectl 和 Helm 的 [Cloud Shell](https://learn.microsoft.com/en-us/azure/cloud-shell/overview)。

<div class="alert note">安装 Azure CLI 后，请确保已正确进行身份验证。</div>

## 配置 Kubernetes 集群

1. 登录 Azure 门户。
2. 在 Azure 门户菜单或**主页**中，选择**创建资源**。
3. 选择**容器** > **Kubernetes 服务**。
4. 在**基本信息**页面上，配置以下选项：

- **项目详细信息**：
  - **订阅**：联系您组织的 Azure 管理员确定应使用哪个订阅。

    - **资源组**：联系您组织的 Azure 管理员确定应使用哪个资源组。

- **集群详细信息**：
  - **Kubernetes 集群名称**：输入集群名称。

  - **区域**：选择一个区域。

  - **可用区**：根据需要选择[可用区](https://docs.microsoft.com/en-us/azure/aks/availability-zones#overview-of-availability-zones-for-aks-clusters)。对于生产集群，我们建议选择多个可用区。

- **主节点池**：

  - **节点大小**：我们建议选择至少 16 GB RAM 的虚拟机，但您可以根据需要选择虚拟机大小。

  - **缩放方法**：选择一个缩放方法。

  - **节点计数范围**：选择节点数量的范围。

- **节点池**：

  - **启用虚拟节点**：选中复选框以启用虚拟节点。

  - **启用虚拟机规模集**：我们建议选择 `enabled`。

- **网络**：

  - **网络配置**：我们建议选择 `Kubenet`。

  - **DNS 名称前缀**：输入 DNS 名称前缀。

  - **流量路由**：

    - **负载均衡器**：`标准`。

    - **HTTP 应用程序路由**：不需要。

5. 配置选项后，单击**查看 + 创建**，然后在验证完成后单击**创建**。创建集群需要几分钟时间。

## 连接到集群

1. 转到您在 Kubernetes 服务中创建的集群，并单击它。
2. 在左侧导航窗格中，单击 `概览`。
3. 在显示的 **概览** 页面上，单击 **连接** 查看资源组和订阅。

## 设置订阅和凭据

<div class="alert note">您可以使用 Azure Cloud Shell 执行以下步骤。</div>

1. 运行以下命令设置您的订阅。

```shell
az account set --subscription EXAMPLE-SUBSCRIPTION-ID
```
2. 运行以下命令下载凭据并配置 Kubernetes CLI 以使用它们。
   
```shell
az aks get-credentials --resource-group YOUR-RESOURCE-GROUP --name YOUR-CLUSTER-NAME
```

<div class="alert note">
在以下步骤中继续使用相同的 Shell。如果切换到另一个 Shell，请重新运行上述命令。
</div>


## 使用 Azure Blob 存储作为外部对象存储

Azure Blob 存储是 Azure 的 AWS Simple Storage Service (S3) 版本。

- 创建存储帐户和容器
```bash
az storage account create -n milvustesting1 -g MyResourceGroup -l eastus --sku Standard_LRS --min-tls-version TLS1_2
az storage container create -n testmilvus --account-name milvustesting1
```

- 获取密钥，使用第一个值
```bash
az storage account keys list --account-name milvustesting2
```

- 添加 values.yaml
```yaml
cluster:
  enabled: true

service:
  type: LoadBalancer

extraConfigFiles:
  user.yaml: |+
    common:
      storageType: remote

minio:
  enabled: false

externalS3:
  enabled: true
  host: core.windows.net
  port: 443
  rootPath: my-release
  bucketName: testmilvus # 存储帐户容器名称
  cloudProvider: azure
  useSSL: true
  accessKey: "milvustesting1" # 存储帐户名称
  secretKey: "<secret-key>" 
```

## 部署 Milvus

现在 Kubernetes 集群已准备就绪。让我们立即部署 Milvus。

```bash
helm repo add milvus https://zilliztech.github.io/milvus-helm/
helm repo update
helm install -f values.yaml my-release milvus/milvus
```

在上述命令中，我们将 Milvus Helm 图表的存储库添加到本地，并更新存储库以获取最新的图表。然后我们安装一个 Milvus 实例并命名为 **my-release**。

注意配置 `service.type` 的值，该值表示我们希望通过第4层负载均衡器公开 Milvus 实例。

## 验证部署

一旦所有的 Pod 都在运行，运行以下命令以获取外部 IP 地址。

```bash
kubectl get services|grep my-release-milvus|grep LoadBalancer|awk '{print $4}'
```


## 你好，Milvus

请参考 [Hello Milvus](https://milvus.io/docs/example_code.md)，将主机值更改为外部 IP 地址，然后运行代码。


## 接下来做什么

如果您想了解如何在其他云上部署 Milvus：
- [在 EC2 上部署 Milvus 集群](aws.md)
- [在 EKS 上部署 Milvus 集群](eks.md)
- [在 GCP 上部署 Milvus 集群](gcp.md)