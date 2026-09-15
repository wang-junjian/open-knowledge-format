---
type: Metric
title: 毛利率（旧版，FY2026 之前）
description: 已退役的毛利率定义，仅包含商品成本。保留用于历史查询可复现性。请勿用于新分析。
tags: [finance, margin, deprecated]
generated: { by: human:jsmith@acme, at: 2024-01-15T10:00:00Z }
verified:
  - { by: human:jsmith@acme, at: 2024-01-15T10:00:00Z }
status: deprecated
---

# 已弃用（Deprecated）

**该指标已退役。** 当前的毛利率定义为 [`metrics/gross-margin.md`](./gross-margin.md)，实现了 FY2026 成本分摊标准（商品成本 + 入站履约成本 + 出站运费 + 支付手续费）。

保留此概念，是为了使 2026-02-01 之前编写的历史报告仍可复现。新工作请勿引用它。

# 旧版定义（仅用于可复现性）

在 FY2026 之前的定义下，毛利率为：

```
gross-margin-legacy(period) = revenue(period) - SUM(products.cost * order_lines.quantity)  over orders recognized in period
```

也就是说，COGS 仅含商品成本；履约、运费与支付手续费计入营业费用（operating expenses），而非 COGS。Finance 在 2025 年第四季度得出结论：该定义低估了商品的运营成本，并使该数值无法与总账对账。

# 为何没有 Attested Computation

此指标没有对应的 `Attested Computation`。其退役时，SQL 已从授权集合中删除。任何重跑历史报告者，必须依据此叙事重建 SQL，并明确将结果标注为旧版（legacy）。
