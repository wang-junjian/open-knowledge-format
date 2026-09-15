---
type: Reference
resource: https://support.google.com/analytics/answer/9037342
title: 高频活跃用户指标
description: 构建一个在最近 M 天中至少 N 天活跃的用户的受众。
tags:
- metric
- audience
- ga4
- frequent-actives
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T21:16:25+00:00'
sources:
- title: Sample queries for audiences based on BigQuery data - Analytics Help
  id: sample_queries
  resource: https://support.google.com/analytics/answer/9037342
---

构建一个高频活跃用户（Frequently Active Users）受众，定义为在最近 M 天中至少 N 天记录了至少一个带有事件参数 `engagement_time_msec > 0` 的事件的用户，其中 M > N（例如：在最近 10 天中至少 4 天）。

# 结构
本参考描述了一种查询模式，并不映射到单一的数据库 schema。

# 常见查询模式

```sql
/**
 * 构建高频活跃用户受众。
 *
 * 高频活跃用户 = 在最近 M 天中至少 N 天（M > N）记录了至少一个
 * 带有事件参数 engagement_time_msec > 0 的事件的用户。
 */
 
SELECT
  COUNT(DISTINCT user_id) AS frequent_active_users_count
FROM
  (
    SELECT
      user_id,
      COUNT(DISTINCT event_date)
    FROM
      `YOUR_TABLE.events_*` AS T
    CROSS JOIN
      T.event_params
    WHERE
      event_params.key = 'engagement_time_msec' AND event_params.value.int_value > 0
      -- 最近 M = 10 天的用户互动。
      AND event_timestamp >
          UNIX_MICROS(TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 10 DAY))
      AND _TABLE_SUFFIX BETWEEN '20180521' AND '20240131'
    GROUP BY 1
    -- 至少互动了 N = 4 天。
    HAVING COUNT(event_date) >= 4
  );
```
[^sample_queries]

[^sample_queries]: [Google Analytics 帮助：基于 BigQuery 数据的受众示例查询](https://support.google.com/analytics/answer/9037342)
