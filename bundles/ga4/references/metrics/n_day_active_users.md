---
type: Reference
resource: https://support.google.com/analytics/answer/9037342
title: N 日活跃用户指标
description: 基于 engagement_time_msec 构建最近 N 天内活跃的用户的受众。
tags:
- metric
- audience
- ga4
- active-users
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T21:16:16+00:00'
sources:
- title: Sample queries for audiences based on BigQuery data - Analytics Help
  resource: https://support.google.com/analytics/answer/9037342
  id: sample_queries
---

构建一个 N 日活跃用户（N-Day Active Users）受众，定义为在最近 N 天内记录了至少一个带有事件参数 `engagement_time_msec > 0` 的事件的用户。

# 结构
本参考描述了一种查询模式，并不映射到单一的数据库 schema。

# 常见查询模式

```sql
/**
 * 构建 N 日活跃用户受众。
 *
 * N 日活跃用户 = 在最近 N 天内记录了至少一个
 * 带有事件参数 engagement_time_msec > 0 的事件的用户。
*/

SELECT
  COUNT(DISTINCT user_id) AS n_day_active_users_count
FROM
  `YOUR_TABLE.events_*` AS T
    CROSS JOIN
      T.event_params
WHERE
  event_params.key = 'engagement_time_msec' AND event_params.value.int_value > 0
  -- 选择最近 N = 20 天内的事件。
  AND event_timestamp >
      UNIX_MICROS(TIMESTAMP_SUB(CURRENT_TIMESTAMP, INTERVAL 20 DAY))
  AND _TABLE_SUFFIX BETWEEN '20180521' AND '20240131';
```
[^sample_queries]

[^sample_queries]: [Google Analytics 帮助：基于 BigQuery 数据的受众示例查询](https://support.google.com/analytics/answer/9037342)
