---
id: configure_etcd.md
related_key: configure
group: system_configuration.md
summary: 学习如何为 Milvus 配置 etcd。
title: 与 etcd 相关的配置
---

# 与 etcd 相关的配置

本主题介绍了 Milvus 的与 etcd 相关的配置。etcd 是支持 Milvus 元数据存储和访问的元数据引擎。

在本节中，您可以配置 etcd 的端点、相关键前缀等。

<div class="alert note">
要在多个 Milvus 实例之间共享一个 etcd 实例，您需要为每个 Milvus 实例更改 <code>etcd.rootPath</code> 为一个唯一值。详情请参考 <a href="operational_faq.md#Can-I-share-an-etcd-instance-among-multiple-Milvus-instances">操作常见问题</a>。
</div>

## `etcd.endpoints`

<table id="etcd.endpoints">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>用于访问 etcd 服务的端点。您可以将此参数更改为您自己 etcd 集群的端点。</li>
        <li>环境变量：<code>ETCD_ENDPOINTS</code></li>
        <li>当启动 Milvus 时，etcd 优先从环境变量 <code>ETCD_ENDPOINTS</code> 中获取有效地址。</li>
      </td>
      <td>localhost:2379</td>
    </tr>
  </tbody>
</table>


## `etcd.rootPath`

<table id="etcd.rootPath">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>Milvus 在 etcd 中存储数据的键的根前缀。</li>
        <li>建议在首次启动 Milvus 之前更改此参数。</li>
        <li>要在多个 Milvus 实例之间共享一个 etcd 实例，请考虑在启动之前为每个 Milvus 实例更改此值。详情请参见 <a href="operational_faq.md#Can-I-share-an-etcd-instance-among-multiple-Milvus-instances">操作常见问题</a>。</li>
        <li>如果 etcd 服务已存在，请为 Milvus 设置一个易于识别的根路径。</li>
        <li>在运行中的 Milvus 实例更改此值可能导致无法读取旧数据。</li>
      </td>
      <td>by-dev</td>
    </tr>
  </tbody>
</table>

## `etcd.metaSubPath`

<table id="etcd.metaSubPath">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>Milvus 在 etcd 中存储与元数据相关信息的键的子前缀。</li>
        <li>注意：在使用 Milvus 一段时间后更改此参数将影响您访问旧数据。</li>
        <li>建议在首次启动 Milvus 之前更改此参数。</li>
      </td>
      <td>meta</td>
    </tr>
  </tbody>
</table>


## `etcd.kvSubPath`

<table id="etcd.kvSubPath">
  <thead>
    <tr>
<table>
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认数值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>在 etcd 中存储 Milvus 时间戳的键的子前缀。</li>
        <li>注意：在使用 Milvus 一段时间后更改此参数将影响您访问旧数据。</li>
        <li>建议没有特殊原因时不更改此参数。</li>
      </td>
      <td>kv</td>
    </tr>
  </tbody>
</table>