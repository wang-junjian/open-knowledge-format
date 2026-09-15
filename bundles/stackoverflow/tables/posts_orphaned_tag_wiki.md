---
type: BigQuery Table
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/posts_orphaned_tag_wiki
title: 孤立的标签 Wiki 帖子
description: 充当已不存在或孤立标签的 wiki 条目的帖子。
tags:
- stackoverflow
- posts
- wiki
- tags
- orphaned
generated:
  by: reference_agent/gemini-2.5-flash
  at: '2026-07-10T22:48:39+00:00'
sources:
- id: posts-orphaned-tag-wiki-resource
  resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/posts_orphaned_tag_wiki
---

该表包含被视为已孤立或不再存在于 Stack Overflow 上的标签的"标签 wiki"条目帖子。这些帖子通常为特定标签提供解释或定义。表 `posts_orphaned_tag_wiki` 是更大的 [stackoverflow 数据集](../datasets/stackoverflow.md) 的一部分。

# 架构

该架构包含与帖子本身相关的字段，例如内容、日期和所有者信息。

- `id`: 帖子的唯一标识符。
- `title`: 帖子的标题。
- `body`: 帖子的正文内容，通常为 Markdown 格式。
- `accepted_answer_id`: (STRING) 被采纳回答的 ID（如适用）。
- `answer_count`: (STRING) 帖子的回答数量。
- `comment_count`: 帖子上的评论数量。
- `community_owned_date`: 帖子转为社区所有的timestamp。
- `creation_date`: 帖子创建的timestamp。
- `favorite_count`: (STRING) 帖子被收藏的次数。
- `last_activity_date`: 帖子上最近活动的timestamp。
- `last_edit_date`: 帖子最近一次编辑的timestamp。
- `last_editor_display_name`: 最近编辑者的显示名称。
- `last_editor_user_id`: 最近编辑者的用户 ID。
- `owner_display_name`: 帖子所有者的显示名称。
- `owner_user_id`: 帖子所有者的用户 ID。
- `parent_id`: (STRING) 父帖子的 ID（针对回答或评论）。
- `post_type_id`: 帖子类型（如 1 为问题，2 为回答，3 为标签 wiki 条目）。
- `score`: 帖子的评分。
- `tags`: (STRING) 与帖子关联的标签（对标签 wiki 自身通常为 NULL，因为其描述的是标签）。
- `view_count`: (STRING) 帖子的浏览次数。

# 常见查询模式

```sql
-- 选择所有孤立的标签 wiki 帖子
SELECT
    id,
    title,
    creation_date
FROM
    `bigquery-public-data.stackoverflow.posts_orphaned_tag_wiki`
LIMIT 100;
```

```sql
-- 查找某个特定孤立标签 wiki 帖子的正文内容
SELECT
    body
FROM
    `bigquery-public-data.stackoverflow.posts_orphaned_tag_wiki`
WHERE
    id = 4164933;
```
