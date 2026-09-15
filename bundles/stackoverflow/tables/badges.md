---
type: BigQuery Table
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/badges
title: 徽章
description: 该表包含 Stack Overflow 上授予用户的徽章信息。
tags:
- stackoverflow
- badges
- gamification
generated:
  by: reference_agent/gemini-2.5-flash
  at: '2026-07-10T22:59:00+00:00'
sources:
- resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/badges
  title: Stack Overflow Badges Table
  id: badges-table
---

`badges` 表跟踪 Stack Overflow 平台上授予用户的所有徽章。每一行代表在某个特定时间授予某个特定用户的单个徽章实例。该表包含徽章名称、获得徽章的用户，以及是否为基于标签的徽章等详细信息。这些数据可用于分析用户参与度、识别顶尖贡献者，并理解平台的游戏化机制。

# 架构

- `id`: INTEGER，徽章授予记录的唯一标识符。
- `name`: STRING，所授予徽章的名称（如 "Great Answer"、"Electorate"）。
- `date`: TIMESTAMP，授予徽章的日期和时间。
- `user_id`: INTEGER，获得该徽章的用户 ID。链接到 [users](../tables/users.md) 表。
- `class`: INTEGER，徽章的等级或层级（如 1 为金牌，2 为银牌，3 为铜牌）。
- `tag_based`: BOOLEAN，指示该徽章是否与特定标签关联。

# 常见查询模式

1. **统计每位用户获得的徽章数量：**
   ```sql
   SELECT
     user_id,
     count(id) AS badge_count
   FROM
     `bigquery-public-data.stackoverflow.badges`
   GROUP BY
     user_id
   ORDER BY
     badge_count DESC
   LIMIT 10;
   ```
2. **查找授予最频繁的徽章：**
   ```sql
   SELECT
     name,
     count(id) AS award_count
   FROM
     `bigquery-public-data.stackoverflow.badges`
   GROUP BY
     name
   ORDER BY
     award_count DESC
   LIMIT 10;
   ```
3. **获取授予特定用户的所有金牌徽章：**
   ```sql
   SELECT
     t2.display_name,
     t1.name,
     t1.date
   FROM
     `bigquery-public-data.stackoverflow.badges` AS t1
   JOIN
     `bigquery-public-data.stackoverflow.users` AS t2
   ON
     t1.user_id = t2.id
   WHERE
     t1.class = 1 -- 假设 class 1 为金牌
     AND t2.display_name = 'Jon Skeet'
   ORDER BY
     t1.date DESC;
   ```
