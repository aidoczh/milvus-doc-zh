---
id: configure_pulsar.md
related_key: configure
group: system_configuration.md
summary: 学习如何为 Milvus 集群配置 Pulsar。
title: 与 Pulsar 相关的配置
---

# 与 Pulsar 相关的配置

本主题介绍了 Milvus 中与 Pulsar 相关的配置。

Pulsar 是支持 Milvus 集群可靠存储和消息流发布/订阅的底层引擎。

在本节中，您可以配置 Pulsar 地址、消息大小等。

<div class="alert note">
<li>要在多个 Milvus 实例之间共享启用了多租户的 Pulsar 实例，您需要为每个 Milvus 实例更改 <code>pulsar.tenant</code> 或 <code>pulsar.namespace</code> 为唯一值。</li>
<li>要在多个 Milvus 实例之间共享禁用了多租户的 Pulsar 实例，您需要为每个 Milvus 实例更改 <code>msgChannel.chanNamePrefix.cluster</code> 为唯一值。</li>
有关详细信息，请参考 <a href="operational_faq.md#Can-I-share-a-Pulsar-instance-among-multiple-Milvus-instances">操作常见问题解答</a>。
</div>

## `pulsar.address`

<table id="pulsar.address">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>Pulsar 服务的 IP 地址。</li>
        <li>环境变量：<code>PULSAR_ADDRESS</code></li>
        <li><code>pulsar.address</code> 和 <code>pulsar.port</code> 共同生成对 Pulsar 的有效访问。</li>
        <li>Milvus 启动时，Pulsar 优先从环境变量 <code>PULSAR_ADDRESS</code> 中获取有效的 IP 地址。</li>
        <li>当 Pulsar 在与 Milvus 相同网络上运行时，使用默认值。</li>
      </td>
      <td>localhost</td>
    </tr>
  </tbody>
</table>

## `pulsar.port`

<table id="pulsar.port">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>Pulsar 服务的端口。</li>
        <li>环境变量：<code>PULSAR_ADDRESS</code></li>
        <li><code>pulsar.address</code> 和 <code>pulsar.port</code> 共同生成对 Pulsar 的有效访问。</li>
        <li>Milvus 启动时，Pulsar 优先从环境变量 <code>PULSAR_ADDRESS</code> 中获取有效的端口。</li>
      </td>
      <td>6650</td>
    </tr>
  </tbody>
</table>

## `pulsar.webport`

<table id="pulsar.webport">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>Pulsar 服务的 Web 端口。</li>
        <li>如果直接连接而无需代理，则应使用 8080。</li>
      </td>
      <td>80</td>
    </tr>
  </tbody>
</table>

## `pulsar.maxMessageSize`

<table id="pulsar.maxMessageSize">
  <thead>
    <tr>
## `pulsar.maxMessageSize`

<table id="pulsar.maxMessageSize">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>Pulsar 中每条消息的最大大小。</li>
        <li>单位：字节</li>
        <li>默认情况下，Pulsar 可以在单个消息中传输最多 5 MB 的数据。当插入数据的大小大于此值时，代理将数据分段为多个消息，以确保它们可以正确传输。</li>
        <li>如果在 Pulsar 中对应的参数保持不变，增加此配置将导致 Milvus 失败，减少它则没有任何优势。</li>
      </td>
      <td>5242880</td>
    </tr>
  </tbody>
</table>

## `pulsar.tenant`

<table id="pulsar.tenant">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>可以为特定租户配置适当的容量来为其提供 Pulsar。</li>
        <li>要在多个 Milvus 实例之间共享一个 Pulsar 实例，您可以在启动它们之前将其更改为 Pulsar 租户，而不是每个 Milvus 实例的默认租户。但是，如果您不想使用 Pulsar 多租户功能，建议您将 <code>msgChannel.chanNamePrefix.cluster</code> 更改为不同的值。详情请参见 <a href="operational_faq.md#Can-I-share-a-Pulsar-instance-among-multiple-Milvus-instances">操作常见问题解答</a>。</li>
      </td>
      <td>public</td>
    </tr>
  </tbody>
</table>

## `pulsar.namespace`

<table id="pulsar.namespace">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>Pulsar 命名空间是租户内的管理单元名称。</li>
        <li>要在多个 Milvus 实例之间共享一个 Pulsar 实例，您可以在启动它们之前将其更改为 Pulsar 租户，而不是每个 Milvus 实例的默认租户。但是，如果您不想使用 Pulsar 多租户功能，建议您将 <code>msgChannel.chanNamePrefix.cluster</code> 更改为不同的值。详情请参见 <a href="operational_faq.md#Can-I-share-a-Pulsar-instance-among-multiple-Milvus-instances">操作常见问题解答</a>。</li>
      </td>
      <td>default</td>
    </tr>
  </tbody>
</table>