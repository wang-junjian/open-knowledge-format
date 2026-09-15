---
type: Log
title: Acme Retail bundle 历史
---

# Bundle 历史

## 2026-07-01

- **校验（Verified）**：对整个 bundle 进行了 OKF v0.2 合规性校验。`human:kliu@acme` 审核了所有 `verified` 与 `sources` 条目。

## 2026-06-30

- **重新生成（Re-generated）**：在 Finance 发布 FY2026 收入确认策略增补后，重新生成了 `metrics/revenue.md`、`computations/revenue-ytd.md` 与 `policies/revenue-recognition.md`。将两个收入相关概念的 `stale_after` 更新为 `2026-12-31T00:00:00Z`。

## 2026-04-15

- **弃用（Deprecated）**：旧版毛利率定义。原始文件已移动至 `metrics/gross-margin-legacy.md`，状态为 `status: deprecated`。新定义位于 `metrics/gross-margin.md`，实现了 FY2026 成本分摊标准（将履约成本与运费计入 COGS）。

## 2026-02-10

- **Bundle 引导生成（Bundle bootstrapped）**：由 `reference_agent/gemini-2.5-pro` 基于 BigQuery 的 `INFORMATION_SCHEMA` 以及 `region-us.INFORMATION_SCHEMA.JOBS_BY_PROJECT` 的 90 天样本生成。初始信任等级：全盘机器确认；财务关键概念已标记待人工复核。
