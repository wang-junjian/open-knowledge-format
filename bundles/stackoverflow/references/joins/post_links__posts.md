---
type: Reference
resource: https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede
title: 帖子链接 ↔ 帖子连接
description: post_links 表与 posts 表之间的连接路径。
tags:
- join
- posts
- links
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T23:03:01+00:00'
sources:
- title: Database schema documentation for the public data dump and SEDE
  id: meta_schema_doc
  resource: https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede
---

# post_links ↔ posts

post_links 表与目标/源帖子之间的连接关系。

```sql
ON post_links.post_id = posts.id
```

## 用法

使用此连接路径来解析链接/重复关系中的源帖子（`post_id`）或目标（`related_post_id`）的元数据。

[^1]: 经 Meta Stack Exchange 上的 [Database Schema Documentation](https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede) 核对。
