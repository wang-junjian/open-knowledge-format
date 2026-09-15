---
type: Reference
resource: https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede
title: 帖子类型参考
description: Stack Overflow 帖子表中 PostTypeId 列的枚举查找值。
tags:
- lookup
- enum
- posts
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T23:02:30+00:00'
sources:
- resource: https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede
  title: Database schema documentation for the public data dump and SEDE
  id: meta_schema_doc
- id: meta_post_types
  title: Meaning of Values for PostTypeId in data explorer or in data-dump
  resource: https://meta.stackexchange.com/questions/99265/meaning-of-values-for-posttypeid-in-data-explorer-or-in-data-dump
---

# 帖子类型参考

该查找目录定义了在 `posts_answers`、`posts_questions`、`posts_moderator_nomination`、`posts_orphaned_tag_wiki`、`posts_privilege_wiki`、`posts_tag_wiki`、`posts_tag_wiki_excerpt`、`posts_wiki_placeholder` 和 `stackoverflow_posts` 等主表中使用的 `PostTypeId` 属性的含义。

## 查找目录

| PostTypeId | 名称 | 描述 |
|---|---|---|
| 1 | Question | 用户提交的问题。 |
| 2 | Answer | 用户对某个问题提交的回答。 |
| 3 | Orphaned tag wiki | 已被删除标签的标签 wiki。 |
| 4 | Tag wiki excerpt | 标签的简短介绍/摘要文本。 |
| 5 | Tag wiki | 详述标签使用规范的完整正文。 |
| 6 | Moderator nomination | 版主候选人提名帖子。 |
| 7 | Wiki placeholder | 辅助站点内容（如帮助中心介绍、选举说明、导览介绍）。 |
| 8 | Privilege wiki | 权限说明页面。 |
| 9 | Article | 文章帖子类型。 |
| 10 | HelpArticle | 帮助中心文章。 |
| 12 | Collection | 内容合集。 |
| 13 | ModeratorQuestionnaireResponse | 候选人对版主问卷的回答。 |
| 14 | Announcement | 站点公告。 |
| 15 | CollectiveDiscussion | Stack Overflow Collectives 讨论串。 |
| 17 | CollectiveCollection | Stack Overflow Collectives 合集。 |

[^1]: 经 Meta Stack Exchange 上的 [Database Schema Documentation](https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede) 与 [PostTypeId Meanings](https://meta.stackexchange.com/questions/99265/meaning-of-values-for-posttypeid-in-data-explorer-or-in-data-dump) 核对。
