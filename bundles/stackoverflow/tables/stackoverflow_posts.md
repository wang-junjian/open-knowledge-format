---
type: BigQuery Table
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/stackoverflow_posts
title: Stack Overflow 帖子（已弃用）
description: 包含 Stack Overflow 帖子的已弃用表。请改用 posts_answers 或 posts_questions 表。
tags: stackoverflow, posts, deprecated
status: deprecated
generated:
  by: reference_agent/gemini-2.5-flash
  at: '2026-07-10T22:50:28+00:00'
sources:
- title: Deprecated Stack Overflow Posts Table
  id: stackoverflow-posts-table
  resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/stackoverflow_posts
---

`stackoverflow_posts` 表包含来自 Stack Overflow 的帖子合集。每一行代表一条帖子，可以是问题、回答或其他类型的帖子。关键信息包括帖子的标题、正文、创建日期、评分以及关联的标签。

**警告：** 该表**已弃用**，不应用于新的开发或分析。如需最新且更专业的数据，请使用单独的帖子类型表，具体而言，问题请使用 [posts_questions](posts_questions.md)，回答请使用 [posts_answers](posts_answers.md)。

# 架构

`stackoverflow_posts` 表包含以下字段：

*   `id`: `INTEGER` (REQUIRED) - 帖子的唯一标识符。
*   `title`: `STRING` - 帖子的标题（如对问题而言）。
*   `body`: `STRING` - 帖子的正文内容。
*   `accepted_answer_id`: `INTEGER` - 被采纳回答的 ID（如适用）。
*   `answer_count`: `INTEGER` - 问题的回答数量。
*   `comment_count`: `INTEGER` - 帖子上的评论数量。
*   `community_owned_date`: `TIMESTAMP` - 帖子转为社区所有的日期。
*   `creation_date`: `TIMESTAMP` - 帖子创建的日期和时间。
*   `favorite_count`: `INTEGER` - 帖子被收藏的次数。
*   `last_activity_date`: `TIMESTAMP` - 帖子上最近活动的日期。
*   `last_edit_date`: `TIMESTAMP` - 帖子最近一次编辑的日期。
*   `last_editor_display_name`: `STRING` - 最近编辑者的显示名称。
*   `last_editor_user_id`: `INTEGER` - 最近编辑者的用户 ID。
*   `owner_display_name`: `STRING` - 帖子所有者的显示名称。
*   `owner_user_id`: `INTEGER` - 帖子所有者的用户 ID。
*   `parent_id`: `INTEGER` - 对回答而言，为其所回答问题 ID。
*   `post_type_id`: `INTEGER` - 帖子类型（如 1 为 Question，2 为 Answer）。
*   `score`: `INTEGER` - 帖子的当前评分。
*   `tags`: `STRING` - 与帖子关联的标签，通常针对问题（如 `<python><django>`）。
*   `view_count`: `INTEGER` - 帖子的浏览次数。

# 常见查询模式

```sql
-- 危险：该表已弃用。请勿用于新查询。
-- 选择基础帖子信息的示例（仅供历史参考）。
SELECT
    id,
    title,
    creation_date,
    score,
    tags
FROM
    `bigquery-public-data.stackoverflow.stackoverflow_posts`
WHERE
    creation_date BETWEEN '2016-01-01' AND '2016-01-31'
LIMIT 100;
```
