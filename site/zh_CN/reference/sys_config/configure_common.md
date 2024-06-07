---
id: configure_common.md
related_key: configure
group: system_configuration.md
summary: 学习如何配置 Milvus 的常见参数。
title: 常见配置
---

# 常见配置

本主题介绍了 Milvus 的常见配置。

在本节中，您可以配置 Milvus 的默认分区和索引名称，以及时间旅行（数据保留）跨度。

## `common.defaultPartitionName`

<table id="common.defaultPartitionName">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>在创建集合时的默认分区名称。</td>
      <td>"_default"</td>
    </tr>
  </tbody>
</table>

## `common.defaultIndexName`

<table id="common.defaultPartitionName">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>当未指定名称创建索引时的索引名称。</td>
      <td>"_default_idx"</td>
    </tr>
  </tbody>
</table>

## `common.retentionDuration`

<table id="common.retentionDuration">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <li>允许时间旅行的已删除数据的保留时长。</li>
        <li>单位：秒</li>
      </td>
      <td>432000</td>
    </tr>
  </tbody>
</table>

## `common.ttMsgEnabled`

<table id="common.ttMsgEnabled">
  <thead>
    <tr>
      <th class="width80">描述</th>
      <th class="width20">默认值</th> 
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>是否禁用系统的内部时间消息机制。如果禁用（设置为 `false`），系统将不允许 DML 操作，包括插入、删除、查询和搜索。这有助于 Milvus-CDC 同步增量数据。</td>
      <td>false</td>
    </tr>
  </tbody>
</table>