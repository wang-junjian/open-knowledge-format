---
type: BigQuery Table
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/posts_privilege_wiki
title: Stack Overflow 权限 Wiki 帖子
description: 包含 Stack Overflow 权限 wiki 帖子的信息，详述用户能力与相应要求。
tags:
- stackoverflow
- wiki
- privilege
- posts
generated:
  by: reference_agent/gemini-2.5-flash
  at: '2026-07-10T22:49:00+00:00'
sources:
- resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/stackoverflow/tables/posts_privilege_wiki
  id: posts-privilege-wiki-resource
  title: Stack Overflow posts_privilege_wiki Table
---

该表是 [Stack Overflow 数据集](../datasets/stackoverflow.md) 的一部分，包含来自 Stack Overflow 平台、描述各类用户权限的特定帖子。这些"权限 wiki"帖子详述了用户在不同声望等级获得的能力，例如无需同行评审即可编辑问题和回答或重新打标签的能力。该表中的每条条目代表单条权限说明，对权限及其影响进行了全面描述。

`posts_privilege_wiki` 表的特征是 `post_type_id` 等于 `8`，表明其作为权限专属 wiki 帖子的性质。它帮助用户理解平台上声望与权限的运作机制。

# 架构

- id: INTEGER
- title: STRING
- body: STRING
    权限 wiki 帖子的完整 HTML 内容，详述该权限。
- accepted_answer_id: STRING
- answer_count: STRING
- comment_count: INTEGER
- community_owned_date: STRING
- creation_date: TIMESTAMP
    权限 wiki 帖子最初创建的日期和时间。
- favorite_count: STRING
- last_activity_date: TIMESTAMP
- last_edit_date: TIMESTAMP
    权限 wiki 帖子最近一次编辑的日期和时间。
- last_editor_display_name: STRING
- last_editor_user_id: INTEGER
- owner_display_name: STRING
- owner_user_id: INTEGER
    拥有或创建该权限 wiki 帖子的用户 ID。
- parent_id: STRING
- post_type_id: INTEGER
    权限 wiki 帖子始终为 `8`。
- score: INTEGER
- tags: STRING
- view_count: STRING

# 常见查询模式

```sql
-- 检索所有权限 wiki 帖子
SELECT
    id,
    title,
    body,
    creation_date
  FROM
    `bigquery-public-data.stackoverflow.posts_privilege_wiki`
  WHERE
    post_type_id = 8
  LIMIT 100;
```

```sql
-- 查找正文中提及 "edit" 的权限 wiki 帖子
SELECT
    id,
    title,
    creation_date
  FROM
    `bigquery-public-data.stackoverflow.posts_privilege_wiki`
  WHERE
    post_type_id = 8 AND CONTAINS_SUBSTR(body, 'edit');
```

```sql
-- 统计权限 wiki 帖子的数量
SELECT
    COUNT(id) AS privilege_wiki_post_count
  FROM
    `bigquery-public-data.stackoverflow.posts_privilege_wiki`
  WHERE
    post_type_id = 8;
```
