---
type: Reference
resource: https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede
title: 帖子 ↔ 投票连接
description: votes 表与 posts 表之间的连接路径。
tags:
- join
- posts
- votes
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T23:02:55+00:00'
sources:
- resource: https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede
  id: meta_schema_doc
  title: Database schema documentation for the public data dump and SEDE
---

# posts ↔ votes

votes 表与 posts 表之间的连接关系。

```sql
ON votes.post_id = posts.id
```

## 用法

使用此连接路径将各项投票、标记和收藏关联到其目标帖子（问题、回答或版主提名）。

[^1]: 经 Meta Stack Exchange 上的 [Database Schema Documentation](https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede) 核对。
