---
type: Reference
resource: https://support.google.com/analytics/answer/9037342
title: N 日非活跃用户指标
description: 构建一个在最近 M 天中活跃、但在最近 N 天内未活跃的用户的受众。
tags:
- metric
- audience
- ga4
- inactive-users
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T21:16:21+00:00'
sources:
- title: Sample queries for audiences based on BigQuery data - Analytics Help
  id: sample_queries
  resource: https://support.google.com/analytics/answer/9037342
---

构建一个 N 日非活跃用户（N-Day Inactive Users）受众。非活跃用户定义为在最近 M 天（例如 7 天）内活跃、但在最近 N 天（例如 2 天）内未记录任何带有事件参数 `engagement_time_msec > 0` 的事件的用户，其中 M > N。

# 结构
本参考描述了一种查询模式，并不映射到单一的数据库 schema。

# 常见查询模式

```sql
/**
 * 构建 N 日非活跃用户受众。
 *
 * N 日非活跃用户 = 在最近 M 天内活跃、但在最近 N 天内
 * 未记录任何带有事件参数 engagement_time_msec > 0 的事件的用户，
 * 其中 M > N。
 */
 
SELECT
  COUNT(DISTINCT MDaysUsers.user_id) AS n_day_inactive_users_count
FROM
  (
    SELECT
      user_id
    FROM
      `YOUR_TABLE.events_*` AS T
    CROSS JOIN
      T.event_params
    WHERE
      event_params.key = 'engagement_time_msec' AND event_params.value.int_value > 0
      /* 在最近 M = 7 天内有过互动 */
      AND event_timestamp >
          UNIX_MICROS(TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY))
      AND _TABLE_SUFFIX BETWEEN '20180521' AND '20240131'
  ) AS MDaysUsers
LEFT JOIN
  (
    SELECT
      user_id
    FROM
      `YOUR_TABLE.events_*` AS T
    CROSS JOIN
      T.event_params
    WHERE
      event_params.key = 'engagement_time_msec' AND event_params.value.int_value > 0
      /* 在最近 N = 2 天内有过互动 */
      AND event_timestamp >
          UNIX_MICROS(TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 2 DAY))
      AND _TABLE_SUFFIX BETWEEN '20180521' AND '20240131'
  ) AS NDaysUsers
  ON MDaysUsers.user_id = NDaysUsers.user_id
WHERE
  NDaysUsers.user_id IS NULL;
```
[^sample_queries]

[^sample_queries]: [Google Analytics 帮助：基于 BigQuery 数据的受众示例查询](https://support.google.com/analytics/answer/9037342)
