---
type: Policy
title: Acme Retail — 成本分摊与毛利率标准（FY2026）
description: 定义 COGS 构成与标准毛利率公式的财务策略。FY2026 引入（取代了不含履约/运费的旧版定义）。
resource: https://wiki.acme.internal/finance/margin-standard
tags: [finance, policy, margin, cogs]
generated: { by: human:jsmith@acme, at: 2026-02-01T10:00:00Z }
verified:
  - { by: human:jsmith@acme, at: 2026-06-15T09:00:00Z }
status: stable
stale_after: 2026-12-31T00:00:00Z
---

# Acme Retail 成本分摊与毛利率标准 — FY2026

**负责人（Owner）：** VP Finance (jsmith@acme)
**生效（Effective）：** 2026-02-01
**取代（Supersedes）：** 2026 年之前的毛利率定义（见 `metrics/gross-margin-legacy.md`）

## COGS 构成（COGS composition）

FY2026 中一笔已完成订单的 COGS 为以下各项之和：

1. **商品成本（Product cost）** — 取自下单时的 `products.cost`（在订单创建时锁定）
2. **入站履约成本（Inbound fulfillment cost）** — 由月度仓库汇总按单位分摊
3. **出站运费（Outbound shipping cost）** — 来自 `logistics.shipment_cost` 的承运商实际账单
4. **支付手续费（Payment processing fees）** — 来自 `finance.payment_fees` 的 Stripe 手续费

2026 年前的定义仅含 (1)。加入 (2)–(4) 使报告的毛利率下降约 4–6 个百分点，但产出一个 Finance 能与总账（GL）对账的数值。

## 毛利率公式（Gross margin formula）

对于期间 P：

```
gross_margin(P) = SUM(net_amount) - SUM(cogs_full)   over orders recognized in P
```

其中 `cogs_full` 为上述四项之和。

## 报表粒度（Reporting granularity）

该标准支持三个层级的毛利率：组合（portfolio）、品类（category）与 SKU。品类与 SKU 维度需要将 `orders` 按 `product_id` join 到 `products`。

## 本策略授权的内容（What this policy authorizes）

任何 `sources` 引用了本策略的 Attested Computation MUST 使用全部四项 COGS 构成。旧版公式（仅商品成本）保留于 `metrics/gross-margin-legacy.md` 用于历史查询可复现性；新分析请勿使用。

# 被以下引用（Cited by）

- [`metrics/gross-margin`](/metrics/gross-margin.md) — 当前毛利率指标实现了本标准
- [`metrics/gross-margin-legacy`](/metrics/gross-margin-legacy.md) — 已弃用指标，被本标准授权的定义取代
- [`computations/gross-margin-period`](/computations/gross-margin-period.md) — 授权 SQL 实现了此处定义的四项 COGS 构成
