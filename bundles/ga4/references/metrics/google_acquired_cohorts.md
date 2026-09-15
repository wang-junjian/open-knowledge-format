---
type: Reference
resource: https://support.google.com/analytics/answer/9037342
title: Google 获客同期群指标
description: 构建一个由 Google 广告系列来源过滤的特定时间窗口同期群中获取的用户的受众。
tags:
- metric
- audience
- ga4
- cohorts
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T21:16:43+00:00'
sources:
- title: Sample queries for audiences based on BigQuery data - Analytics Help
  resource: https://support.google.com/analytics/answer/9037342
  id: sample_queries
---

构建一个由上周通过 Google 广告系列获取的、由特定广告系列过滤的同期群（cohorts）用户组成的受众。

# 结构
本参考描述了一种查询模式，并不映射到单一的数据库 schema。

# 常见查询模式

```sql
/**
 * 构建一个由上周通过 Google 广告系列获取、
 * 即带有过滤条件的同期群用户组成的受众。
 *
 * 同期群定义为上周（即 7 - 14 天前）获取的用户。
 * 同期群过滤条件针对通过直接广告系列获取的用户。
 */
 
SELECT
  COUNT(DISTINCT user_id) AS users_acquired_through_google_count
FROM
  `YOUR_TABLE.events_*`
WHERE
  event_name = 'first_open'
  -- 同期群：1-2 周前打开应用。一周的同期群，即每周。
  AND event_timestamp >
      UNIX_MICROS(TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 14 DAY))
  AND event_timestamp <
      UNIX_MICROS(TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY))
  -- 同期群过滤：通过 'google' 来源获取的用户。
  AND traffic_source.source = 'google'
  AND _TABLE_SUFFIX BETWEEN '20180501' AND '20240131';
```
[^sample_queries]

[^sample_queries]: [Google Analytics 帮助：基于 BigQuery 数据的受众示例查询](https://support.google.com/analytics/answer/9037342)
