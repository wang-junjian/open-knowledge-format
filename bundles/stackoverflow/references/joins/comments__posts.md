---
type: Reference
resource: https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede
title: 评论 ↔ 帖子连接
description: comments 表与 posts 表之间的连接路径。
tags:
- join
- comments
- posts
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T23:02:52+00:00'
sources:
- resource: https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede
  id: meta_schema_doc
  title: Database schema documentation for the public data dump and SEDE
---

# comments ↔ posts

comments 表与 posts（或回答/问题）表之间的连接关系。

```sql
ON comments.post_id = posts.id
```

## 用法

使用此连接路径将评论内容与评论评分直接关联到其父帖子、回答或问题。适用于计算每个帖子的评论互动量或查找评论串。

[^1]: 经 Meta Stack Exchange 上的 [Database Schema Documentation](https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede) 核对。
