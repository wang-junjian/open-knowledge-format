---
type: BigQuery Table
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/posts_tag_wiki_excerpt
title: 帖子标签 Wiki 摘要
description: 该表包含 Stack Overflow 标签 wiki 的摘要帖子。
tags:
- stackoverflow
- tag wiki
- posts
- excerpt
generated:
  by: reference_agent/gemini-2.5-flash
  at: '2026-07-10T22:49:51+00:00'
sources:
- id: posts-tag-wiki-excerpt-resource
  resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/posts_tag_wiki_excerpt
  title: 'BigQuery Table: posts_tag_wiki_excerpt'
---

该表包含 Stack Overflow 标签 wiki 的摘要帖子。每一行代表一个标签 wiki 的摘要或节选，提供特定标签的简要描述。这有助于在不阅读完整标签 wiki 的情况下理解 Stack Overflow 平台上各种标签的用途和背景。

# 架构

- `id`: INTEGER，标签 wiki 摘要帖子的唯一标识符。
- `title`: STRING，标签 wiki 摘要的标题。
- `body`: STRING，标签 wiki 摘要的正文或内容。
- `accepted_answer_id`: STRING，被采纳回答的 ID（如适用，但在标签 wiki 摘要中不太可能出现）。
- `answer_count`: STRING，回答数量（如适用）。
- `comment_count`: INTEGER，帖子上的评论数量。
- `community_owned_date`: TIMESTAMP，帖子转为社区所有的日期。
- `creation_date`: TIMESTAMP，帖子创建的日期。
- `favorite_count`: STRING，帖子被收藏的次数。
- `last_activity_date`: TIMESTAMP，帖子上最近活动的日期。
- `last_edit_date`: TIMESTAMP，帖子最近一次编辑的日期。
- `last_editor_display_name`: STRING，最近编辑者的显示名称。
- `last_editor_user_id`: INTEGER，最近编辑者的用户 ID。
- `owner_display_name`: STRING，帖子所有者的显示名称。
- `owner_user_id`: INTEGER，帖子所有者的用户 ID。
- `parent_id`: STRING，父帖子的 ID（如适用）。
- `post_type_id`: INTEGER，帖子类型（如 5 为标签 Wiki 摘要）。
- `score`: INTEGER，帖子的评分。
- `tags`: STRING，与帖子关联的标签（如 `<python><sql>`）。
- `view_count`: STRING，帖子的浏览次数。

# 常见查询模式

```sql
SELECT
    id,
    title,
    body
  FROM
    `bigquery-public-data.stackoverflow.posts_tag_wiki_excerpt`
  WHERE
    creation_date BETWEEN '2020-01-01' AND '2020-12-31'
  LIMIT 100;
```
```sql
SELECT
    t.tag_name,
    p.title AS excerpt_title,
    p.body AS excerpt_body
  FROM
    `bigquery-public-data.stackoverflow.posts_tag_wiki_excerpt` AS p
    JOIN `bigquery-public-data.stackoverflow.tags` AS t ON CONCAT('<', t.tag_name, '>') = p.tags
  WHERE
    t.tag_name = 'python'
  LIMIT 1;
```
