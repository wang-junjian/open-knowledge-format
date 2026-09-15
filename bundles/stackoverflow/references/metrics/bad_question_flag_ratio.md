---
type: Reference
resource: https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede
title: 劣质问题标记率
description: 计算垃圾信息与攻击性标记（VoteTypeId 4 和 12）占全部投票/标记的比例。
tags:
- metric
- votes
- moderation
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T23:02:44+00:00'
sources:
- id: meta_schema_doc
  resource: https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede
  title: Database schema documentation for the public data dump and SEDE
---

# bad_question_flag_ratio

劣质问题标记率衡量在问题上投出的全部标记中，被归类为垃圾信息或攻击性标记所占的比例。该指标是追踪垃圾信息攻击浪潮或高度不当内容趋势的有用信号。

## 公式

```sql
SAFE_DIVIDE(
  COUNTIF(VoteTypeId IN (4, 12)),
  COUNT(Id)
)
```

[^1]: 公式来源于并基于 `VoteTypeId` 分类（4 = Offensive，12 = Spam）定义，详见 [Database Schema Documentation](https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede)。
