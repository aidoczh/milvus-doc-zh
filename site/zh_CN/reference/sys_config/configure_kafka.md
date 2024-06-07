---
id: configure_kafka.md
related_key: configure
group: system_configuration.md
summary: 学习如何为 Milvus 集群配置 Kafka。
title: 与 Kafka 相关的配置
---

# 与 Kafka 相关的配置

本主题介绍了 Milvus 的与 Kafka 相关的配置。

Kafka 是支持 Milvus 集群可靠存储和消息流发布/订阅的基础引擎。

在本节中，您可以配置 Kafka 的生产者、消费者、sasl 信息等。

## `kafka.producer.client.id`

<table id="kafka.producer.client.id">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>Kafka 服务的生产者客户端 ID。</li>
      </td>
    </tr>
  </tbody>
</table>

## `kafka.consumer.client.id`

<table id="kafka.consumer.client.id">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>Kafka 服务的消费者客户端 ID。</li>
      </td>
    </tr>
  </tbody>
</table>

## `kafka.brokerList`

<table id="kafka.brokerList">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>Kafka 服务的 broker 地址列表。</li>
        <li>每个值之间用逗号分隔。</li>
        <li>例如：localhost1:9092,localhost2:9092,localhost3:9092</li>
      </td>
    </tr>
  </tbody>
</table>

## `kafka.saslUsername`

<table id="kafka.saslUsername">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>如果启用了 <code>简单身份验证和安全层</code>，则为 Kafka 服务的用户名。</li>
      </td>
    </tr>
  </tbody>
</table>

## `kafka.saslPassword`

<table id="kafka.saslPassword">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>如果启用了 <code>简单身份验证和安全层</code>，则为 Kafka 服务的密码。</li>
      </td>
    </tr>
  </tbody>
</table>

## `kafka.saslMechanisms`

<table id="kafka.saslMechanisms">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>如果启用了 <code>简单身份验证和安全层</code>，则为 Kafka 服务的机制。</li>
        <li>有效值可能为：GSSAPI, PLAIN, SCRAM, OAUTHBEARER。</li>
      </td>
    </tr>
  </tbody>
</table>

## `kafka.securityProtocol`

<table id="kafka.securityProtocol">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
```markdown
<tr>
  <td>
    <li>Kafka 服务的安全协议。</li>
    <li>有效值可能包括：PLAINTEXT，SSL，SASL_PLAINTEXT，SASL_SSL。</li>
  </td>
</tr>
</tbody>
</table>
```