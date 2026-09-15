---
type: Reference
resource: https://support.google.com/analytics/answer/9037342
title: 获客用户指标
description: 构建一个通过特定 Source、Medium 和 Campaign 名称获取的用户的受众。
tags:
- metric
- audience
- ga4
- acquired-users
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T21:16:35+00:00'
sources:
- title: Sample queries for audiences based on BigQuery data - Analytics Help
  resource: https://support.google.com/analytics/answer/9037342
  id: sample_queries
---

构建一个获客用户（Acquired Users）受众，定义为通过特定营销广告系列的 source、medium 和 name 获取的用户。

# 结构
本参考描述了一种查询模式，并不映射到单一的数据库 schema。

# 常见查询模式

```sql
/**
 * 构建获客用户受众。
 *
 * 获客用户 = 通过某个 Source/Medium/Campaign 获取的用户。
 */
 
SELECT
  COUNT(DISTINCT user_id) AS acquired_users_count
FROM
  `YOUR_TABLE.events_*`
WHERE
  traffic_source.source = 'google'
  AND traffic_source.medium = 'cpc'
  AND traffic_source.name = 'VTA-Test-Android'
  AND _TABLE_SUFFIX BETWEEN '20180521' AND '20240131';
```
[^sample_queries]

[^sample_queries]: [Google Analytics 帮助：基于 BigQuery 数据的受众示例查询](https://support.google.com/analytics/answer/9037342)
