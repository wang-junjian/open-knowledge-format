---
type: BigQuery Table
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/post_links
title: 帖子链接
description: 包含 Stack Overflow 上帖子之间链接的信息。
tags:
- stackoverflow
- posts
- links
- cross-references
generated:
  by: reference_agent/gemini-2.5-flash
  at: '2026-07-10T22:47:44+00:00'
sources:
- id: post_links_table
  resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/post_links
---

`post_links` 表存储了 Stack Overflow 上帖子彼此之间如何链接的信息。每一行代表两个帖子之间的一条链接，表示某种关系，例如重复问题、相关问题或 wiki 链接。该表对于理解 Stack Overflow 平台内内容的相互关联至关重要。

该表的粒度是每两个帖子之间的一条链接一行。

# 架构

- `id`: 帖子链接的唯一标识符。
- `creation_date`: 链接创建的日期和时间。
- `link_type_id`: 链接类型的标识符（如 duplicate、related）。
- `post_id`: 链接中主帖子的 ID。通常指 [posts_questions](posts_questions.md) 或 [posts_answers](posts_answers.md) 表中的帖子。
- `related_post_id`: 链接中相关帖子的 ID，同样指向 [posts_questions](posts_questions.md) 或 [posts_answers](posts_answers.md) 表中的帖子。

# 常见查询模式

```sql
SELECT
  pl.id,
  pl.creation_date,
  pl.link_type_id,
  p1.title AS post_title,
  p2.title AS related_post_title
FROM
  `bigquery-public-data.stackoverflow.post_links` AS pl
JOIN
  `bigquery-public-data.stackoverflow.posts_questions` AS p1
  ON pl.post_id = p1.id
JOIN
  `bigquery-public-data.stackoverflow.posts_questions` AS p2
  ON pl.related_post_id = p2.id
WHERE
  pl.link_type_id = 3 -- 示例：LinkType = "Related"
LIMIT 100;
```

```sql
SELECT
  link_type_id,
  COUNT(*)
FROM
  `bigquery-public-data.stackoverflow.post_links`
GROUP BY
  link_type_id
ORDER BY
  COUNT(*) DESC;
```
