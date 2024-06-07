---
id: gcs.md
title: 通过工作负载身份配置 GCS 访问
related_key: gcs, storage, workload identity, iam
summary: 学习如何通过工作负载身份配置 GCS。
---

# 通过工作负载身份配置 GCS 访问
本主题介绍了在使用 Helm 安装 Milvus 时如何通过工作负载身份配置 GCS 访问。
更多详情，请参考 [工作负载身份](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity)。

## 开始之前

请使用 Google Cloud CLI 或 Google Cloud 控制台在集群和节点池上启用工作负载身份。在启用节点池上的工作负载身份之前，必须在集群级别启用工作负载身份。

## 配置应用程序以使用工作负载身份

- 创建存储桶。
```bash
gcloud storage buckets create gs://milvus-testing-nonprod --project=milvus-testing-nonprod --default-storage-class=STANDARD --location=us-west1 --uniform-bucket-level-access
```

- 为您的应用程序创建一个 Kubernetes 服务账号。
```bash
kubectl create serviceaccount milvus-gcs-access-sa
```

- 为您的应用程序创建一个 IAM 服务账号，或者使用现有的 IAM 服务账号。您可以在组织中的任何项目中使用任何 IAM 服务账号。
```bash
gcloud iam service-accounts create milvus-gcs-access-sa \
    --project=milvus-testing-nonprod
```

- 确保您的 IAM 服务账号具有所需的角色。您可以使用以下命令授予额外的角色：
```bash
gcloud projects add-iam-policy-binding milvus-testing-nonprod \
    --member "serviceAccount:milvus-gcs-access-sa@milvus-testing-nonprod.iam.gserviceaccount.com" \
    --role "roles/storage.admin" \
    --condition='title=milvus-testing-nonprod,expression=resource.service == "storage.googleapis.com" && resource.name.startsWith("projects/_/buckets/milvus-testing-nonprod")'
```

- 允许 Kubernetes 服务账号冒充 IAM 服务账号，通过在两个服务账号之间添加 IAM 策略绑定来实现。此绑定允许 Kubernetes 服务账号充当 IAM 服务账号。
```bash
gcloud iam service-accounts add-iam-policy-binding milvus-gcs-access-sa@milvus-testing-nonprod.iam.gserviceaccount.com \
    --role "roles/iam.workloadIdentityUser" \
    --member "serviceAccount:milvus-testing-nonprod.svc.id.goog[default/milvus-gcs-access-sa]"
```

- 使用 IAM 服务账号的电子邮件地址为 Kubernetes 服务账号添加注释。
```bash
kubectl annotate serviceaccount milvus-gcs-access-sa \
    --namespace default \
    iam.gke.io/gcp-service-account=milvus-gcs-access-sa@milvus-testing-nonprod.iam.gserviceaccount.com
```

## 验证工作负载身份设置

请参考 [工作负载身份](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity)。在 Pod 内运行以下命令：
```bash
curl -H "Metadata-Flavor: Google" http://169.254.169.254/computeMetadata/v1/instance/service-accounts/default/email
```
如果结果是 `milvus-gcs-access-sa@milvus-testing-nonprod.iam.gserviceaccount.com`，那就没问题。

## 部署 Milvus
```bash
helm install -f values.yaml my-release milvus/milvus
``` 

values.yaml 内容如下：
```yaml
cluster:
    enabled: true

service:
    type: LoadBalancer

minio:
    enabled: false

serviceAccount:
    create: false
    name: milvus-gcs-access-sa

externalS3:
    enabled: true
    host: storage.googleapis.com
    port: 443
    rootPath: milvus/my-release
    bucketName: milvus-testing-nonprod
    cloudProvider: gcp
    useSSL: true
    useIAM: true
```