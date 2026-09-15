---
type: Reference
resource: https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede
title: 帖子回答 ↔ 帖子问题连接
description: posts_answers 表与 posts_questions 表之间的连接路径。
tags:
- join
- posts
- answers
- questions
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T23:03:04+00:00'
sources:
- id: meta_schema_doc
  title: Database schema documentation for the public data dump and SEDE
  resource: https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede
---

# posts_answers ↔ posts_questions

Stack Overflow 问题与它们的回答之间的连接关系。

```sql
ON posts_answers.parent_id = posts_questions.id
```

## 用法

使用此连接路径将回答直接关联回其父问题，以汇总回答数量、验证采纳回答率等指标，或比较问题/回答的评分。

[^1]: 经 Meta Stack Exchange 上的 [Database Schema Documentation](https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede) 核对。
