---
type: Reference
resource: https://support.google.com/analytics/answer/9037342
title: 购买者受众指标
description: 计算已完成购买或应用内购买的用户的计数或列表。
tags:
- metric
- audience
- ga4
- purchasers
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T21:16:12+00:00'
sources:
- id: sample_queries
  title: Sample queries for audiences based on BigQuery data - Analytics Help
  resource: https://support.google.com/analytics/answer/9037342
---

计算购买者（purchasers）受众，定义为记录了 `in_app_purchase` 或 `purchase` 的用户。

# 结构
本参考描述了一种查询模式，并不映射到单一的数据库 schema。

# 常见查询模式

```sql
/**
 * 计算购买者受众。
 *
 * 购买者 = 记录了 in_app_purchase 或 purchase 的用户。
 */
 
SELECT
  COUNT(DISTINCT user_id) AS purchasers_count
FROM
  `YOUR_TABLE.events_*`
WHERE
  event_name IN ('in_app_purchase', 'purchase')
  AND _TABLE_SUFFIX BETWEEN '20180501' AND '20240131';
```
[^sample_queries]

[^sample_queries]: [Google Analytics 帮助：基于 BigQuery 数据的受众示例查询](https://support.google.com/analytics/answer/9037342)
