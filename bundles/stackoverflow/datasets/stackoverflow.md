---
type: BigQuery Dataset
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow
title: Stack Overflow 公开数据集
description: 该数据集包含 Stack Overflow 数据的公开存档，包括帖子、用户和标签。最后更新于
  2022-11-25，已不再主动更新。
tags: Stack Overflow, Q&A, developer, programming, public dataset
generated:
  by: reference_agent/gemini-2.5-flash
  at: '2026-07-10T22:46:36+00:00'
sources:
- title: Stack Overflow Public Dataset
  resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow
  id: stackoverflow-dataset-resource
---

`stackoverflow` 数据集托管于 BigQuery 公开数据计划，提供 Stack Overflow 社区生成内容的完整存档。它包含问题、回答、评论、用户、徽章和标签的信息，是分析开发者活动、编程趋势和社区动态的丰富资源。该数据最后更新于 2022-11-25，其原始来源已不再主动更新。它位于 `US` 多区域。

# 架构

本数据集包含以下表：

*   [`badges`](../tables/badges.md)：授予用户的徽章信息。
*   [`comments`](../tables/comments.md)：用户对帖子的评论。
*   [`post_history`](../tables/post_history.md)：帖子的历史修订与事件。
*   [`post_links`](../tables/post_links.md)：帖子之间的链接。
*   [`posts_answers`](../tables/posts_answers.md)：问题的回答。
*   [`posts_moderator_nomination`](../tables/posts_moderator_nomination.md)：与版主提名相关的帖子。
*   [`posts_orphaned_tag_wiki`](../tables/posts_orphaned_tag_wiki.md)：孤立的标签 wiki 帖子。
*   [`posts_privilege_wiki`](../tables/posts_privilege_wiki.md)：权限 wiki 帖子。
*   [`posts_questions`](../tables/posts_questions.md)：用户提交的问题。
*   [`posts_tag_wiki`](../tables/posts_tag_wiki.md)：标签 wiki 条目。
*   [`posts_tag_wiki_excerpt`](../tables/posts_tag_wiki_excerpt.md)：标签 wiki 条目的摘要。
*   [`posts_wiki_placeholder`](../tables/posts_wiki_placeholder.md)：wiki 内容的占位帖子。
*   [`stackoverflow_posts`](../tables/stackoverflow_posts.md)：所有帖子（问题和回答）的整合视图。
*   [`tags`](../tables/tags.md)：Stack Overflow 上所用标签的信息。
*   [`users`](../tables/users.md)：用户资料与统计。
*   [`votes`](../tables/votes.md)：帖子的投票记录。

# 常见查询模式

如需浏览本数据集中的表：

```sql
SELECT table_name
FROM `bigquery-public-data.stackoverflow.INFORMATION_SCHEMA.TABLES`
WHERE table_schema = 'stackoverflow';
```

如需查询某一年发布的提问数量：

```sql
SELECT
  EXTRACT(YEAR FROM creation_date) AS year,
  COUNT(*) AS num_questions
FROM `bigquery-public-data.stackoverflow.posts_questions`
GROUP BY 1
ORDER BY 1 DESC
LIMIT 100;
```
