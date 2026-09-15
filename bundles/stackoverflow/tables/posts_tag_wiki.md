---
type: BigQuery Table
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/posts_tag_wiki
title: 帖子标签 Wiki
description: 与 Stack Overflow 上所用标签关联的详细 wiki 条目，提供超越基础标签描述的综合信息。
tags:
- stackoverflow
- posts
- tags
- wiki
generated:
  by: reference_agent/gemini-2.5-flash
  at: '2026-07-10T22:49:37+00:00'
sources:
- resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/posts_tag_wiki
  title: 'BigQuery Table: posts_tag_wiki'
  id: bq-table
---

[stackoverflow](../datasets/stackoverflow.md) 数据集中的 `posts_tag_wiki` 表包含 Stack Overflow 平台上所用标签的综合 wiki 条目。与 [tags](tags.md) 表或 [posts_tag_wiki_excerpt](posts_tag_wiki_excerpt.md) 中的简短描述不同，该表为每个标签的 wiki 提供完整长度的内容，通常包含详尽的解释、使用指南和示例。每一行代表一个唯一的标签 wiki 条目，由其唯一的 `id` 标识。

# 架构

- `id`: 标签 wiki 条目的唯一标识符。
- `title`: wiki 条目的标题（在此表中通常为 NULL，意味着标签本身即为标题）。
- `body`: 标签 wiki 的正文内容，通常为 HTML 格式。
- `accepted_answer_id`: 被采纳回答的 ID（如适用）。
- `answer_count`: 回答数量。
- `comment_count`: 评论数量。
- `community_owned_date`: 帖子转为社区所有的日期。
- `creation_date`: 标签 wiki 条目创建的timestamp。
- `favorite_count`: 帖子被收藏的次数。
- `last_activity_date`: 帖子上最近活动的timestamp。
- `last_edit_date`: 最近一次编辑的timestamp。
- `last_editor_display_name`: 最近编辑者的显示名称。
- `last_editor_user_id`: 最近编辑者的用户 ID。
- `owner_display_name`: 所有者的显示名称。
- `owner_user_id`: 所有者的用户 ID。
- `parent_id`: 父帖子的 ID（如适用）。
- `post_type_id`: 帖子类型（如 5 为 Wiki 条目）。
- `score`: 帖子的评分。
- `tags`: 与条目关联的标签（在此表中通常为 NULL，因为其本身*就是*标签 wiki）。
- `view_count`: 浏览次数。

# 常见查询模式

```sql
SELECT
    id,
    creation_date,
    body
  FROM
    `bigquery-public-data.stackoverflow.posts_tag_wiki`
  WHERE
    id = 5046395;
```

```sql
SELECT
    id,
    creation_date,
    last_edit_date,
    LENGTH(body) AS body_length
  FROM
    `bigquery-public-data.stackoverflow.posts_tag_wiki`
  WHERE
    creation_date > '2020-01-01 00:00:00 UTC'
  ORDER BY
    creation_date DESC
  LIMIT 10;
```

```sql
SELECT
    id,
    SUBSTR(body, 1, 100) AS body_preview
  FROM
    `bigquery-public-data.stackoverflow.posts_tag_wiki`
  WHERE
    LOWER(body) LIKE '%example code%';
```
