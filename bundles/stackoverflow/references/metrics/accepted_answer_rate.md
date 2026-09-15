---
type: Reference
resource: https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede
title: 采纳回答率
description: 拥有被采纳回答的问题所占的比例。
tags:
- metric
- posts
- community
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T23:02:48+00:00'
sources:
- resource: https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede
  id: meta_schema_doc
  title: Database schema documentation for the public data dump and SEDE
---

# accepted_answer_rate

采纳回答率衡量拥有已解决且被采纳回答的问题所占的比例。它是衡量问题解决效率的核心社区健康 KPI。

## 公式

```sql
SAFE_DIVIDE(
  COUNT(AcceptedAnswerId),
  COUNT(Id)
)
```

[^1]: 公式来源于并基于 `posts_questions` 与 `stackoverflow_posts` 的架构定义，详见 [Database Schema Documentation](https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede)。
