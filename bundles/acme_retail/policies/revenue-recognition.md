---
type: Policy
title: Acme Retail — 收入确认策略（FY2026）
description: 定义客户订单何时被确认为收入的财务策略。每年审查一次。
resource: https://wiki.acme.internal/finance/revenue-recognition
tags: [finance, policy, revenue]
generated: { by: human:jsmith@acme, at: 2026-01-05T10:00:00Z }
verified:
  - { by: human:jsmith@acme, at: 2026-06-15T09:00:00Z }
status: stable
stale_after: 2026-12-31T00:00:00Z
---

# Acme Retail 收入确认策略 — FY2026

**负责人（Owner）：** VP Finance (jsmith@acme)
**生效（Effective）：** 2026-01-01
**下次计划审查：** 2026-12-31

## 确认触发条件（Recognition trigger）

当客户订单达到 `order_status = 'delivered'` **且** 退货窗口已关闭（送达日期 + 30 天）时，确认收入。更早状态的订单属于积压（backlog），而非收入。

## 确认金额（Recognized amount）

订单的确认金额等于 `net_amount = gross_amount - discount_amount`。依据美国 GAAP，运费与税不计入收入（属于过账负债，pass-through liabilities）。

## 币种（Currency）

多币种订单按 `finance.fx_daily_rates` 发布的当日参考汇率，以 `order_ts` 日期换算。所有报告均以 USD 计。

## 退款与取消（Refunds and cancellations）

退款与确认后取消，计入退款当期的备抵收入（contra-revenue），而非对原确认期做追溯调整。

## 财年（Fiscal year）

Acme Retail 采用美国日历年（1 月 1 日 – 12 月 31 日）。所有财年指标使用 `fiscal_year = EXTRACT(YEAR FROM order_ts)`。

## 本策略授权的内容（What this policy authorizes）

任何 `sources` 引用了本策略的 Attested Computation MUST 实现上述四条规则。偏离需经 Finance 审查的策略增补。

# 被以下引用（Cited by）

- [`tables/orders`](/tables/orders.md) — `order_status`、`order_ts` 与 `net_amount` 列实现了确认规则
- [`metrics/revenue`](/metrics/revenue.md) — 已确认收入定义源自本策略
- [`metrics/gross-margin`](/metrics/gross-margin.md) — 毛利率的收入侧遵循本策略
- [`computations/revenue-ytd`](/computations/revenue-ytd.md) — 授权 SQL 实现了全部四条规则
- [`computations/gross-margin-period`](/computations/gross-margin-period.md) — 收入部分使用这些确认规则
