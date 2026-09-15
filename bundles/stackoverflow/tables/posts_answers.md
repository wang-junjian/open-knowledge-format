---
type: BigQuery Table
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/posts_answers
title: 帖子回答
description: 包含 Stack Overflow 的回答，包括其内容、评分和关联的元数据。
tags: stackoverflow, answers, posts, Q&A
generated:
  by: reference_agent/gemini-2.5-flash
  at: '2026-07-10T22:48:04+00:00'
sources:
- title: Stack Overflow Posts Answers Table
  id: stackoverflow-posts_answers
  resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/posts_answers
---

`bigquery-public-data.stackoverflow` 数据集中的 `posts_answers` 表包含用户在 Stack Overflow 平台上提交的所有回答。该表中的每一行代表对某个问题的一条回答。关键信息包括回答的 `body`（内容）、`creation_date`、`score` 和 `owner_user_id`。回答通过 `parent_id` 字段链接到相应的问题，该字段引用 [posts_questions](posts_questions.md) 表中的 `id`。该表可用于分析回答质量、用户贡献以及回答随时间的变化趋势。

# 架构

- id: INTEGER（回答的唯一 ID）
- title: STRING（帖子的标题。对回答通常为 NULL，因为标题属于问题）
- body: STRING（回答的 HTML 内容）
- accepted_answer_id: STRING（父问题的被采纳回答 ID。对回答自身通常为 NULL）
- answer_count: STRING（父问题的回答数量。对回答通常为 NULL）
- comment_count: INTEGER（该回答上的评论数量）
- community_owned_date: TIMESTAMP（回答转为社区所有的日期）
- creation_date: TIMESTAMP（回答发布的 UTC 时间戳）
- favorite_count: STRING（父问题被收藏的次数。对回答通常为 NULL）
- last_activity_date: TIMESTAMP（该回答上最近活动的 UTC 时间戳）
- last_edit_date: TIMESTAMP（该回答最近一次编辑的 UTC 时间戳）
- last_editor_display_name: STRING（最近编辑该回答的用户显示名称）
- last_editor_user_id: INTEGER（最近编辑该回答的用户 ID）
- owner_display_name: STRING（发布该回答的用户显示名称）
- owner_user_id: INTEGER（发布该回答的用户 ID）
- parent_id: INTEGER（该回答所属问题的 ID。链接到 [posts_questions](posts_questions.md) 表中的 `id`。）
- post_type_id: INTEGER（帖子类型；回答为 `2`。）
- score: INTEGER（回答的当前评分，基于赞同票与反对票）
- tags: STRING（与父问题关联的标签。对回答通常为 NULL）
- view_count: STRING（父问题的浏览次数。对回答通常为 NULL）

# 常见查询模式

```sql
SELECT
  id,
  body,
  score,
  creation_date
FROM
  `bigquery-public-data.stackoverflow.posts_answers`
ORDER BY
  score DESC
LIMIT 10
```

```sql
SELECT
  owner_user_id,
  count(id) AS answer_count
FROM
  `bigquery-public-data.stackoverflow.posts_answers`
WHERE
  owner_user_id IS NOT NULL
GROUP BY
  owner_user_id
ORDER BY
  answer_count DESC
LIMIT 5
```

```sql
SELECT
  id,
  body,
  score,
  owner_display_name
FROM
  `bigquery-public-data.stackoverflow.posts_answers`
WHERE
  parent_id = 12345
ORDER BY
  score DESC
```
