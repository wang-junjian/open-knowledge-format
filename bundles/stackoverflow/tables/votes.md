---
type: BigQuery Table
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/votes
title: Stack Overflow 投票
description: 记录对 Stack Overflow 帖子投出的所有投票。
tags: Stack Overflow, votes, posts, community
generated:
  by: reference_agent/gemini-2.5-flash
  at: '2026-07-10T22:51:18+00:00'
sources:
- resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/votes
  title: 'BigQuery Table: votes'
  id: bq-table
- title: Database schema documentation for the public data dump and SEDE
  resource: https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede
  id: meta_schema_doc
---

该表包含 Stack Overflow 社区内对帖子投出的所有单项投票。每一行代表一次投票，将其关联到特定帖子，并记录投票类型和发生时间。`vote_type_id` 列可与维度表连接以解码投票含义（如赞同票、反对票、收藏）。

关于 `vote_type_id` 列取值的详细目录，请参见 [投票类型参考](../references/vote_types.md)。 [^1]

# 架构

*   `id`: 投票的唯一标识符。
*   `creation_date`: 投票投出的日期和时间。（注意：出于隐私考虑，时间信息被截断为 `00:00:00`。）[^1]
*   `post_id`: 收到该投票的帖子标识符。可与 [posts_questions](posts_questions.md) 或 [posts_answers](posts_answers.md) 等表中的 `id` 列连接，以获取被投票帖子的详情。
*   `vote_type_id`: 所投投票的类型（如赞同票、反对票、收藏）。参见 [投票类型参考](../references/vote_types.md)。 [^1]

# 常见查询模式

要统计某个帖子的赞同票总数：
```sql
SELECT
    count(*) AS upvotes
  FROM
    `bigquery-public-data.stackoverflow.votes`
  WHERE
    post_id = 12345 -- 替换为实际的帖子 ID
    AND vote_type_id = 2 -- 假设 '2' 代表赞同票
```

要按总投票数查找前 10 个被投票最多的帖子：
```sql
SELECT
    post_id,
    count(*) AS total_votes
  FROM
    `bigquery-public-data.stackoverflow.votes`
  GROUP BY
    post_id
  ORDER BY
    total_votes DESC
  LIMIT 10
```

要查看投票类型的分布：
```sql
SELECT
    vote_type_id,
    count(*) AS vote_count
  FROM
    `bigquery-public-data.stackoverflow.votes`
  GROUP BY
    vote_type_id
  ORDER BY
    vote_count DESC
```

# 指标

- [劣质问题标记率](../references/metrics/bad_question_flag_ratio.md) — 计算问题上的垃圾信息与攻击性标记所占的比例。 [^1]

# 连接

- [posts](../references/joins/posts__votes.md) — 通过 `post_id` ↔ `id` 连接，将投票操作关联到问题、回答及其他帖子类型。 [^1]

[^1]: 来源于 Meta Stack Exchange 上的 [Database Schema Documentation](https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede)。
