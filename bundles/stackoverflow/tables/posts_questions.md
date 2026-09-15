---
type: BigQuery Table
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/posts_questions
title: Stack Overflow 帖子问题
description: 包含 Stack Overflow 的全部问题帖子。
tags: stackoverflow, posts, questions
generated:
  at: '2026-07-10T22:49:19+00:00'
  by: reference_agent/gemini-2.5-flash
sources:
- id: stackoverflow-posts-questions
  resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/posts_questions
  title: Stack Overflow Posts Questions Table
- id: meta_schema_doc
  resource: https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede
  title: Database schema documentation for the public data dump and SEDE
---

该表包含 Stack Overflow 的全部问题帖子，包括其内容、元数据，以及与活动和用户参与度相关的各种计数。每一行代表一个唯一的问题帖子。该表可与其他表连接，例如与 [posts_answers](posts_answers.md) 连接以将问题与对应回答关联，或与 [users](users.md) 连接以获取问题所有者的更多信息。

关于 `post_type_id` 列取值的详细目录，请参见 [帖子类型参考](../references/post_types.md)。关于帖子许可的详情，请参见 [知识共享内容许可证参考](../references/content_licenses.md)。 [^1]

# 架构

- `id`: INTEGER，帖子的唯一标识符。
- `title`: STRING，问题的标题。
- `body`: STRING，问题的正文内容。
- `accepted_answer_id`: INTEGER，被采纳回答的 ID（如有）。
- `answer_count`: INTEGER，问题的回答数量。
- `comment_count`: INTEGER，问题的评论数量。
- `community_owned_date`: TIMESTAMP，问题转为社区所有的日期。
- `creation_date`: TIMESTAMP，问题创建的日期和时间。
- `favorite_count`: INTEGER，问题被收藏的次数。
- `last_activity_date`: TIMESTAMP，问题上最近活动的日期和时间。
- `last_edit_date`: TIMESTAMP，问题最近一次编辑的日期和时间。
- `last_editor_display_name`: STRING，最近编辑者的显示名称。
- `last_editor_user_id`: INTEGER，最近编辑者的用户 ID。
- `owner_display_name`: STRING，问题所有者的显示名称。
- `owner_user_id`: INTEGER，问题所有者的用户 ID（链接到 [users](users.md)）。
- `parent_id`: STRING，父 ID（问题通常不使用的）。
- `post_type_id`: INTEGER，帖子类型（问题为 1）。参见 [帖子类型参考](../references/post_types.md)。 [^1]
- `score`: INTEGER，问题的评分。
- `tags`: STRING，与问题关联的标签，以 '|' 分隔。
- `view_count`: INTEGER，问题的浏览次数。

# 常见查询模式

```sql
SELECT
    id,
    title,
    view_count
FROM
    `bigquery-public-data.stackoverflow.posts_questions`
ORDER BY
    view_count DESC
LIMIT 10;
```

```sql
SELECT
    p.title,
    p.score,
    u.display_name AS owner_name
FROM
    `bigquery-public-data.stackoverflow.posts_questions` AS p
JOIN
    `bigquery-public-data.stackoverflow.users` AS u
ON
    p.owner_user_id = u.id
WHERE
    p.creation_date BETWEEN '2022-01-01' AND '2022-01-31'
ORDER BY
    p.score DESC
LIMIT 5;
```

```sql
SELECT
    EXTRACT(DATE FROM creation_date) AS question_date,
    COUNT(id) AS number_of_questions
FROM
    `bigquery-public-data.stackoverflow.posts_questions`
GROUP BY
    question_date
ORDER BY
    question_date DESC;
```

# 指标

- [采纳回答率](../references/metrics/accepted_answer_rate.md) — 计算拥有被采纳回答的问题所占的比例。 [^1]

# 连接

- [posts_answers](../references/joins/posts_answers__posts_questions.md) — 通过 `id` ↔ `parent_id` 连接，将回答附加到问题。 [^1]
- [comments](../references/joins/comments__posts.md) — 通过 `id` ↔ `post_id` 连接，将评论与问题关联。 [^1]
- [votes](../references/joins/posts__votes.md) — 通过 `id` ↔ `post_id` 连接，查找问题上的投票/标记。 [^1]
- [post_links](../references/joins/post_links__posts.md) — 通过 `id` ↔ `post_id` 连接，发现重复与相关问题的链接。 [^1]

[^1]: 来源于 Meta Stack Exchange 上的 [Database Schema Documentation](https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede)。
