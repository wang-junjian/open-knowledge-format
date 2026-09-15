---
type: Reference
resource: https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede
title: 投票类型参考
description: Stack Overflow 投票表中 VoteTypeId 列的枚举查找值。
tags:
- lookup
- enum
- votes
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T23:02:36+00:00'
sources:
- id: meta_schema_doc
  resource: https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede
  title: Database schema documentation for the public data dump and SEDE
- title: List of Vote type IDs
  resource: https://meta.stackexchange.com/questions/171176/list-of-vote-type-ids
  id: meta_vote_types
---

# 投票类型参考

该查找目录定义了 `votes` 表中 `VoteTypeId` 属性的含义。

## 查找目录

| VoteTypeId | 名称 | 描述 |
|---|---|---|
| -1 | InformModerator | 提起标记以引起版主对某个帖子的注意。 |
| 0 | UndoMod | 撤销某项审核操作或投票。 |
| 1 | AcceptedByOriginator | 提问者采纳了某个回答。 |
| 2 | UpMod | 对问题/回答的赞同票。 |
| 3 | DownMod | 对问题/回答的反对票。 |
| 4 | Offensive | 被标记为攻击性或辱骂性内容。 |
| 5 | Favorite | 收藏（现已弃用，由 Saves 取代）。 |
| 6 | Close | 投票关闭某个问题。（此处不再填充；关闭投票位于 PostHistory 中。） |
| 7 | Reopen | 投票重新打开某个问题。 |
| 8 | BountyStart | 用户在某个问题上发起悬赏。 |
| 9 | BountyClose | 某个问题的悬赏已关闭/发放。 |
| 10 | Deletion | 投票删除某个帖子。 |
| 11 | Undeletion | 投票恢复某个帖子。 |
| 12 | Spam | 被标记为垃圾信息。 |
| 15 | ModeratorReview | 版主审核了被标记的帖子。 |
| 16 | ApproveEditSuggestion | 投票批准某项建议的编辑。 |
| 17-28 | Teams Reactions | 在 Stack Overflow for Teams 中实现的反应（如庆祝、微笑、爱心）。 |
| 29 | Outdated | 回答被标记为过时。 |
| 30 | NotOutdated | 断言某回答不过时的投票。 |
| 31 | PreVote | 预投票操作。 |
| 32 | CollectiveDiscussionUpvote | 对 Collectives 讨论的赞同票。 |
| 33 | CollectiveDiscussionDownvote | 对 Collectives 讨论的反对票（已弃用）。 |
| 35 | privateAiAnswerCorrect | 认为 AI 回答正确的投票（实验性）。 |
| 36 | privateAiAnswerIncorrect | 认为 AI 回答不正确的投票（实验性）。 |
| 37 | privateAiAnswerPartiallyCorrect | 认为 AI 回答部分正确的投票。 |

[^1]: 经 Meta Stack Exchange 上的 [Database Schema Documentation](https://meta.stackexchange.com/questions/2677/database-schema-documentation-for-the-public-data-dump-and-sede) 与 [List of Vote type IDs](https://meta.stackexchange.com/questions/171176/list-of-vote-type-ids) 核对。
