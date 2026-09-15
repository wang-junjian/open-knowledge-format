---
type: BigQuery Table
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/post_history
title: 帖子历史
description: 记录 Stack Overflow 上与帖子相关的所有变更与事件的历史。
tags:
- stackoverflow
- posts
- history
- changes
- events
generated:
  by: reference_agent/gemini-2.5-flash
  at: '2026-07-10T22:47:32+00:00'
sources:
- id: post-history-resource
  resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/post_history
  title: Post History BigQuery Table
---

[stackoverflow](../datasets/stackoverflow.md) 数据集中的 `post_history` 表记录了 Stack Overflow 上与帖子相关的所有变更与事件的详细历史。每一行代表对某个帖子执行的一项特定历史事件或修订，包括初始创建、编辑、回滚以及状态变更。该表对于审计帖子的演变过程以及理解内容如何随时间变化至关重要。记录可追溯至 2016 年 10 月，总条目超过 1.5 亿条。

# 架构

- `id`: 每条帖子历史条目的唯一标识符。
- `creation_date`: 该历史条目的创建timestamp。
- `post_id`: 该历史条目所属帖子的 ID。链接到 [posts_questions](../tables/posts_questions.md) 和 [posts_answers](../tables/posts_answers.md) 表中的 `id` 字段。
- `post_history_type_id`: 表示历史事件类型的整数 ID（如初始标题、正文编辑、标签编辑）。这些 ID 的具体含义通常在单独的查找表中定义。
- `revision_guid`: 用于归组构成单次修订的相关历史条目的唯一 GUID。
- `user_id`: 执行此操作的用户 ID。链接到 [users](../tables/users.md) 表中的 `id` 字段。
- `text`: 与该历史条目关联的内容。其含义取决于 `post_history_type_id`（如新标题、新正文内容、旧标签）。
- `comment`: 用户针对此次变更提供的可选评论。

# 常见查询模式

**检索某个帖子的所有历史条目：**
```sql
SELECT
  id,
  creation_date,
  post_id,
  post_history_type_id,
  revision_guid,
  user_id,
  text,
  comment
FROM
  `bigquery-public-data.stackoverflow.post_history`
WHERE
  post_id = 12345 -- 替换为具体的帖子 ID
ORDER BY
  creation_date DESC;
```

**查找特定用户最近执行的编辑：**
```sql
SELECT
  ph.creation_date,
  ph.post_id,
  ph.text,
  ph.comment,
  pq.title AS post_title
FROM
  `bigquery-public-data.stackoverflow.post_history` AS ph
JOIN
  `bigquery-public-data.stackoverflow.posts_questions` AS pq ON ph.post_id = pq.id
WHERE
  ph.user_id = 67890 -- 替换为实际的用户 ID
  AND ph.post_history_type_id IN (2, 5) -- 示例：假设 2 和 5 分别代表正文/标题编辑
ORDER BY
  ph.creation_date DESC
LIMIT 10;
```

**统计某个帖子不同类型的事件数量：**
```sql
SELECT
  post_history_type_id,
  COUNT(*) AS event_count
FROM
  `bigquery-public-data.stackoverflow.post_history`
WHERE
  post_id = 12345 -- 替换为具体的帖子 ID
GROUP BY
  post_history_type_id
ORDER BY
  event_count DESC;
```
