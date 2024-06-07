---
id: visualize.md
title: 可视化指标
related_key: 监控, 警报
summary: 学习如何在 Grafana 中可视化 Milvus 的指标。
---

# 在 Grafana 中可视化 Milvus 的指标

本主题介绍如何使用 Grafana 可视化 Milvus 的指标。

如 [监控指南](monitor.md) 中所述，指标包含诸如特定 Milvus 组件使用了多少内存等有用信息。监控指标可以帮助您更好地了解 Milvus 的性能和运行状态，以便您及时调整资源分配。

可视化是指展示资源使用情况随时间变化的图表，这样您可以更容易地快速查看和注意资源使用情况的变化，尤其是在发生事件时。

本教程使用 Grafana，这是一个用于时序分析的开源平台，来可视化部署在 Kubernetes（K8s）上的 Milvus 集群的各种性能指标。

## 先决条件
- 您已经在 K8s 上[安装了 Milvus 集群](install_cluster-helm.md)。
- 您需要在使用 Grafana 可视化指标之前[配置 Prometheus](monitor.md) 来监控和收集指标。如果设置成功，您可以通过 `http://localhost:3000` 访问 Grafana。或者您也可以使用默认的 Grafana `user:password` 为 `admin:admin` 来访问 Grafana。

## 使用 Grafana 可视化指标

### 1. 下载并导入仪表板

从 JSON 文件中下载并导入 Milvus 仪表板。

```
wget https://raw.githubusercontent.com/milvus-io/milvus/2.2.0/deployments/monitor/grafana/milvus-dashboard.json
```

![下载并导入](../../../../assets/import_dashboard.png "下载并导入仪表板.")

### 2. 查看指标

选择要监视的 Milvus 实例。然后您可以看到 Milvus 组件面板。


![选择实例](../../../../assets/grafana_select.png "选择一个实例.")

![Grafana 面板](../../../../assets/grafana_panel.png "Milvus 组件面板.")


## 接下来做什么
- 如果您已经设置 Grafana 来可视化 Milvus 的指标，您可能还想要：
  - 学习如何[为 Milvus 服务创建警报](alert.md)
  - 调整您的[资源分配](allocate.md)
  - [扩展或缩减 Milvus 集群](scaleout.md)
- 如果您对升级 Milvus 版本感兴趣，
  - 阅读[升级 Milvus 集群的指南](upgrade_milvus_cluster-operator.md) 和[升级 Milvus 独立部署的指南](upgrade_milvus_standalone-operator.md)。