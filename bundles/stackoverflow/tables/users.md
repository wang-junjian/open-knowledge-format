---
type: BigQuery Table
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/users
title: Stack Overflow 用户
description: 包含 Stack Overflow 平台上注册用户的信息。
tags: stackoverflow, users, community, reputation
generated:
  by: reference_agent/gemini-2.5-flash
  at: '2026-07-10T22:51:02+00:00'
sources:
- resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/users
  title: 'BigQuery Table: users'
  id: bq-table-users
---

[stackoverflow](../datasets/stackoverflow.md) 数据集中的 `users` 表为 Stack Overflow 平台上的每位注册用户提供综合资料。该表中的每一行代表一个唯一用户，记录了其显示名称、声望评分、活动日期和传记信息等详细信息。该表对于分析用户行为、社区参与度和整体平台动态至关重要。

# 架构

- `id`: 用户的唯一标识符。
- `display_name`: 用户选择的公开显示名称。
- `about_me`: 用户提供的简短个人简介。
- `age`: 用户的年龄（以字符串形式，如提供）。
- `creation_date`: 用户账户创建的timestamp。
- `last_access_date`: 用户最近活动或登录的timestamp。
- `location`: 用户提供的地理位置。
- `reputation`: 用户的声望评分。
- `up_votes`: 用户收到的赞同票总数。
- `down_votes`: 用户收到的反对票总数。
- `views`: 用户资料的浏览次数。
- `profile_image_url`: 用户头像图片的 URL。
- `website_url`: 用户个人网站的 URL。

# 常见查询模式

```sql
-- 按声望获取前 10 名用户
SELECT
    display_name,
    reputation,
    location
FROM
    `bigquery-public-data.stackoverflow.users`
ORDER BY
    reputation DESC
LIMIT 10;
```

```sql
-- 查找 2020 年加入且拥有大量赞同票的用户
SELECT
    id,
    display_name,
    creation_date,
    up_votes
FROM
    `bigquery-public-data.stackoverflow.users`
WHERE
    EXTRACT(YEAR FROM creation_date) = 2020
    AND up_votes > 1000
ORDER BY
    up_votes DESC
LIMIT 5;
```

```sql
-- 按地点统计用户数（前 5 个地点）
SELECT
    location,
    COUNT(id) AS user_count
FROM
    `bigquery-public-data.stackoverflow.users`
WHERE
    location IS NOT NULL AND location != ''
GROUP BY
    location
ORDER BY
    user_count DESC
LIMIT 5;
```
