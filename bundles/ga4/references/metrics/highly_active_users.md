---
type: Reference
resource: https://support.google.com/analytics/answer/9037342
title: 高度活跃用户指标
description: 构建一个在最近 M 天中活跃/互动超过 N 分钟的用户的受众。
tags:
- metric
- audience
- ga4
- high-actives
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T21:16:29+00:00'
sources:
- title: Sample queries for audiences based on BigQuery data - Analytics Help
  resource: https://support.google.com/analytics/answer/9037342
  id: sample_queries
---

构建一个高度活跃用户（Highly Active Users）受众，定义为在最近 M 天中活跃/互动超过 N 分钟的用户，其中 M > N（例如：在最近 10 天中超过 0.1 分钟）。

# 结构
本参考描述了一种查询模式，并不映射到单一的数据库 schema。

# 常见查询模式

```sql
/**
 * 构建高度活跃用户受众。
 *
 * 高度活跃用户 = 在最近 M 天（M > N）中活跃超过 N 分钟的用户。
*/

SELECT
  COUNT(DISTINCT user_id) AS high_active_users_count
FROM
  (
    SELECT
      user_id,
      event_params.key,
      SUM(event_params.value.int_value)
    FROM
      `YOUR_TABLE.events_*` AS T
    CROSS JOIN
      T.event_params
    WHERE
      -- 最近 M = 10 天的用户互动。
      event_timestamp >
          UNIX_MICROS(TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 10 DAY))
      AND event_params.key = 'engagement_time_msec'
      AND _TABLE_SUFFIX BETWEEN '20180521' AND '20240131'
    GROUP BY 1, 2
    HAVING
      -- 互动时长超过 N = 0.1 分钟。
      SUM(event_params.value.int_value) > 0.1 * 60 * 1000000
  );
```
[^sample_queries]

[^sample_queries]: [Google Analytics 帮助：基于 BigQuery 数据的受众示例查询](https://support.google.com/analytics/answer/9037342)
