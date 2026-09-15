---
type: BigQuery Table
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/tags
title: 标签
description: 包含 Stack Overflow 上所用标签的信息，包括其名称和使用次数。
tags:
- stackoverflow
- tags
- metadata
generated:
  by: reference_agent/gemini-2.5-flash
  at: '2026-07-10T22:50:47+00:00'
sources:
- resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/tags
  title: Stack Overflow Tags BigQuery Table
  id: stackoverflow-tags-table
- id: stackoverflow-website
  resource: https://stackoverflow.com/
  title: Stack Overflow Website
---

[stackoverflow](../datasets/stackoverflow.md) 数据集中的 `tags` 表提供了 Stack Overflow 上所使用全部标签的综合列表及其关联元数据。每一行代表一个唯一标签，详述其标识符、名称、被使用次数，以及对摘要和 wiki 帖子的引用。该表对于理解 Stack Overflow 社区内问题与回答的分类至关重要。

# 架构

- `id`: 标签的唯一标识符。(INTEGER)
- `tag_name`: 标签的名称（如 'python'、'java'、'c#'）。(STRING)
- `count`: 该标签被使用的总次数。(INTEGER)
- `excerpt_post_id`: 包含该标签摘要描述的帖子 ID。可与 [posts_tag_wiki_excerpt](posts_tag_wiki_excerpt.md) 或 [posts_tag_wiki](posts_tag_wiki.md) 表连接。(INTEGER)
- `wiki_post_id`: 包含该标签完整 wiki 描述的帖子 ID。可与 [posts_tag_wiki](posts_tag_wiki.md) 表连接。(INTEGER)

# 常见查询模式

```sql
-- 检索使用最多的前 10 个标签
SELECT
    tag_name,
    count
  FROM
    `bigquery-public-data.stackoverflow.tags`
  ORDER BY
    count DESC
  LIMIT 10;
```

```sql
-- 查找特定标签的详情
SELECT
    id,
    tag_name,
    count,
    excerpt_post_id,
    wiki_post_id
  FROM
    `bigquery-public-data.stackoverflow.tags`
  WHERE
    tag_name = 'python';
```

```sql
-- 统计唯一标签的总数
SELECT
    COUNT(DISTINCT tag_name) AS total_unique_tags
  FROM
    `bigquery-public-data.stackoverflow.tags`;
```
