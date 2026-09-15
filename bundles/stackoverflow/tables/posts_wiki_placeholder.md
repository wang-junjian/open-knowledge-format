---
type: BigQuery Table
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/posts_wiki_placeholder
title: Stack Overflow 帖子 Wiki 占位
description: Stack Overflow 数据集中各类 wiki 风格帖子的占位表。
tags: stackoverflow, posts, wiki, placeholder, community
generated:
  by: reference_agent/gemini-2.5-flash
  at: '2026-07-10T22:59:18+00:00'
sources:
- resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/posts_wiki_placeholder
  title: 'BigQuery Public Data: Stack Overflow posts_wiki_placeholder table'
  id: stackoverflow-table
---

`posts_wiki_placeholder` 表充当更广泛的 [Stack Overflow 数据集](../datasets/stackoverflow.md) 中各类 wiki 风格帖子的存储库。这些帖子通常包含社区贡献的信息、指南或元讨论，而非直接的问题和回答。内容通常具有说明性质，涵盖站点选举、帮助中心文章，以及对什么构成有效编程问题的定义等主题。每一行代表一个唯一的 Wiki 帖子，由其唯一的 `id` 标识。

# 架构

该表包含以下字段：

- `id`: 帖子的唯一标识符。
- `title`: 帖子的标题。
- `body`: Wiki 帖子的正文内容，通常包含富文本和 markdown。
- `accepted_answer_id`: (NULLABLE)
- `answer_count`: (NULLABLE)
- `comment_count`: 帖子上的评论数量。
- `community_owned_date`: (NULLABLE)
- `creation_date`: 帖子创建的timestamp。
- `favorite_count`: (NULLABLE)
- `last_activity_date`: 帖子上最近活动的timestamp（如编辑、评论）。
- `last_edit_date`: 帖子最近一次编辑的timestamp。
- `last_editor_display_name`: 最近编辑该帖子的用户显示名称。
- `last_editor_user_id`: 最近编辑该帖子的用户 ID。
- `owner_display_name`: 帖子所有者的显示名称。
- `owner_user_id`: 帖子所有者的用户 ID。对社区所有的帖子通常为 -1。
- `parent_id`: (NULLABLE)
- `post_type_id`: 帖子的类型。对于此表，它始终为 `7`，表示 Wiki 帖子。
- `score`: 帖子的评分，反映社区赞同票/反对票。
- `tags`: (NULLABLE) 与帖子关联的标签。
- `view_count`: (NULLABLE) 帖子的浏览次数。

# 常见查询模式

要检索某个特定 Wiki 帖子的正文内容：
```sql
SELECT
    id,
    title,
    body
  FROM
    `bigquery-public-data.stackoverflow.posts_wiki_placeholder`
  WHERE id = 8041931
```

要按创建日期查找最近的 Wiki 帖子：
```sql
SELECT
    id,
    title,
    creation_date
  FROM
    `bigquery-public-data.stackoverflow.posts_wiki_placeholder`
  ORDER BY
    creation_date DESC
  LIMIT 5
```

要统计 Wiki 帖子的总数：
```sql
SELECT
    COUNT(DISTINCT id) AS total_wiki_posts
  FROM
    `bigquery-public-data.stackoverflow.posts_wiki_placeholder`
```
