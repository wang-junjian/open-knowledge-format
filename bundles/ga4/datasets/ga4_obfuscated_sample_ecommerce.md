---
type: BigQuery Dataset
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/ga4_obfuscated_sample_ecommerce
title: GA4 模糊化示例电商数据集
description: 模拟 Google Merchandise Store 网站电商实现的模糊化 Google Analytics
  4 数据集。
tags:
- ga4
- ecommerce
- obfuscated
- analytics
- sample-data
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T21:14:56+00:00'
sources:
- title: BigQuery Dataset Metadata for ga4_obfuscated_sample_ecommerce
  id: ga4-metadata
  resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/ga4_obfuscated_sample_ecommerce
- resource: https://developers.google.com/analytics/bigquery/web-ecommerce-demo-dataset
  title: Google Analytics 4 eCommerce Demo Dataset Documentation
  id: ga4-demo-docs
---

`ga4_obfuscated_sample_ecommerce` 数据集是一份模糊化、可公开访问的 Google Analytics 4（GA4）事件数据导出，代表了真实世界中的网站电商实现（具体来源于 Google Merchandise Store）[^ga4-demo-docs]。它涵盖了从 2020 年 11 月 1 日至 2021 年 1 月 1 日共三个月的历史活动[^ga4-demo-docs]，旨在让开发者、分析师和学生能够在 BigQuery 中试验海量、细粒度的 GA4 事件数据，而无需自行配置专有数据集。

该数据集包含单一的表族 [events_](../tables/events_.md)，其中保存了每日导出表，记录独立的会话交互、用户属性和电商交易明细。分析师可以利用该数据集学习如何查询 GA4 嵌套 schema、构建用户获取模型、还原用户路径，并分析购买漏斗。

# Schema

作为 BigQuery 数据集，该资源充当一个包含表的命名空间，本身没有扁平的列 schema。它托管以下表：

*   [events_](../tables/events_.md)：一个分区、分片的表，包含每日 Google Analytics 4 事件导出记录。

# 常见查询模式

### 1. 统计整个数据集中的事件总数与去重用户数

本查询演示了如何使用通配符后缀模式查询数据集中所有分片表。

```sql
SELECT
  COUNT(*) AS total_events,
  COUNT(DISTINCT user_pseudo_id) AS total_users
FROM
  `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
```

### 2. 定位表并确认数据可用性

本查询检索数据集命名空间中所包含的各表的元数据。

```sql
SELECT
  table_id,
  creation_time,
  row_count,
  size_bytes
FROM
  `bigquery-public-data.ga4_obfuscated_sample_ecommerce.__TABLES__`
ORDER BY
  table_id DESC
```

[^ga4-demo-docs]: [Google Analytics 4 电商示例数据集文档](https://developers.google.com/analytics/bigquery/web-ecommerce-demo-dataset)
