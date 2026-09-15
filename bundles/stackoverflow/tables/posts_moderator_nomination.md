---
type: BigQuery Table
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/posts_moderator_nomination
title: 帖子版主提名
description: 包含 Stack Overflow 平台上与版主提名相关的帖子。
tags: stackoverflow, posts, moderator, nomination
generated:
  by: reference_agent/gemini-2.5-flash
  at: '2026-07-10T22:48:22+00:00'
sources:
- title: 'BigQuery Table: posts_moderator_nomination'
  id: posts-moderator-nomination-table
  resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/posts_moderator_nomination
---

该表存储了 Stack Overflow 社区内代表版主提名的帖子。每一行对应一条提名帖子，通常详述某人应被考虑担任版主角色的理由。这些帖子的特征是 `post_type_id = 6`。该表包含帖子的内容（`body`）、创建日期以及帖子所有者等用户信息。

# 架构

- `id` (INTEGER)：帖子的唯一标识符。
- `title` (STRING)：帖子的标题。
- `body` (STRING)：版主提名帖子的正文内容。
- `accepted_answer_id` (STRING)
- `answer_count` (STRING)
- `comment_count` (INTEGER)：帖子上的评论数量。
- `community_owned_date` (TIMESTAMP)
- `creation_date` (TIMESTAMP)：帖子的创建日期和时间。
- `favorite_count` (STRING)
- `last_activity_date` (TIMESTAMP)
- `last_edit_date` (TIMESTAMP)
- `last_editor_display_name` (STRING)
- `last_editor_user_id` (INTEGER)
- `owner_display_name` (STRING)
- `owner_user_id` (INTEGER)：帖子所有者的用户 ID。
- `parent_id` (STRING)
- `post_type_id` (INTEGER)：帖子类型；版主提名为 `6`。
- `score` (INTEGER)：帖子的评分。
- `tags` (STRING)：与帖子关联的标签。
- `view_count` (STRING)

# 常见查询模式

```sql
SELECT
  id,
  creation_date,
  owner_user_id,
  body
FROM
  `bigquery-public-data.stackoverflow.posts_moderator_nomination`
WHERE
  creation_date >= '2020-01-01'
LIMIT 100;
```
