---
type: BigQuery Table
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/comments
title: 评论
description: 包含 Stack Overflow 数据集中所有对帖子发表的评论。
tags:
- comments
- stackoverflow
- user activity
generated:
  by: reference_agent/gemini-2.5-flash
  at: '2026-07-10T22:47:09+00:00'
sources:
- id: stackoverflow-comments
  resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/comments
  title: Stack Overflow Comments Table
---

[Stack Overflow 数据集](../datasets/stackoverflow.md) 中的 `comments` 表记录了用户在所有问题和回答上发布的所有评论。每一行代表一条评论，提供评论文本、创建日期、所属帖子以及评论者等详细信息。可通过 `post_id` 将此表与 [posts_questions](posts_questions.md) 或 [posts_answers](posts_answers.md) 表连接以获取被评论的内容，并通过 `user_id` 与 [users](users.md) 表连接以获取评论者的更多信息。数据自 2008 年 9 月起。

# 架构

- `id`: 评论的唯一标识符。
- `text`: 评论的内容。
- `creation_date`: 评论创建的timestamp。
- `post_id`: 评论所属帖子（问题或回答）的 ID。
- `user_id`: 发表评论的用户 ID。
- `user_display_name`: 发表评论用户的显示名称（若用户匿名或已删除，可能为 NULL）。
- `score`: 评论获得的评分或赞同票。

# 常见查询模式

1.  **检索某个帖子的所有评论：**
    ```sql
    SELECT
      id,
      text,
      creation_date,
      user_display_name,
      score
    FROM
      `bigquery-public-data.stackoverflow.comments`
    WHERE
      post_id = 47885
    ORDER BY
      creation_date DESC
    LIMIT 100;
    ```

2.  **统计每位用户的评论数：**
    ```sql
    SELECT
      user_display_name,
      COUNT(id) AS total_comments
    FROM
      `bigquery-public-data.stackoverflow.comments`
    WHERE
      user_display_name IS NOT NULL
    GROUP BY
      user_display_name
    ORDER BY
      total_comments DESC
    LIMIT 10;
    ```

3.  **查找包含特定关键词的问题评论：**
    ```sql
    SELECT
      c.id,
      c.text AS comment_text,
      q.title AS question_title,
      q.body AS question_body,
      c.creation_date
    FROM
      `bigquery-public-data.stackoverflow.comments` AS c
    JOIN
      `bigquery-public-data.stackoverflow.posts_questions` AS q
    ON
      c.post_id = q.id
    WHERE
      q.title LIKE '%python%'
    ORDER BY
      c.creation_date DESC
    LIMIT 10;
    ```
