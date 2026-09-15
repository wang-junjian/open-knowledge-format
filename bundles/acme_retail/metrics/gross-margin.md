---
type: Metric
title: 毛利率
description: 某期间的毛利率，依据 Acme FY2026 成本分摊标准（商品成本 + 入站履约成本 + 出站运费 + 支付手续费）。
tags: [finance, margin, headline-metric]
generated: { by: reference_agent/gemini-2.5-pro, at: 2026-06-30T14:00:00Z }
verified:
  - { by: human:jsmith@acme, at: 2026-07-01T09:00:00Z }
status: stable
stale_after: 2026-12-31T00:00:00Z
not:
  - term: "仅收入减去商品成本"
    why: "那是 FY2026 之前的定义（见 gross-margin-legacy）。它未包含履约、运费与支付手续费，且无法与总账对账。"
    instead: "收入减去完整 COGS（商品成本 + 入站履约成本 + 出站运费 + 支付手续费）"
sources:
  - id: margin-standard
    resource: policies/margin-standard.md
    title: Cost Allocation & Margin Standard (FY2026)
    author: human:jsmith@acme
    last_modified: 2026-06-15T00:00:00Z
  - id: revenue-policy
    resource: policies/revenue-recognition.md
    title: Revenue Recognition Policy (FY2026)
    author: human:jsmith@acme
    last_modified: 2026-06-15T00:00:00Z
---

# 定义

**非（Not）：** 仅收入减去商品成本（那是 2026 年前的公式；见 [`gross-margin-legacy`](./gross-margin-legacy.md)）。

某期间的毛利率等于已确认的[收入](./revenue.md)减去**完整 COGS**，其中完整 COGS 为商品成本、入站履约成本、出站运费与支付手续费之和。[^margin-standard]

```
gross_margin(period) = revenue(period) - cogs_full(period)
```

授权计算逻辑为 [`computations/gross-margin-period.md`](../computations/gross-margin-period.md)。消费方 MUST 运行并认证该计算逻辑。

# FY2026 中的变更

在 2026-02-01 之前，Acme 的毛利率定义仅包含商品成本，不含履约、运费与支付手续费。该旧版定义以 `status: deprecated` 保留于 [`metrics/gross-margin-legacy.md`](./gross-margin-legacy.md)，用于历史查询的可复现性。

该切换使报告的毛利率依据品类不同下降约 4–6 个百分点。同时使该数值与总账（general ledger）对齐，弥合了长期存在的对账缺口。

# 信任与时效

- **已校验（Verified）：** 2026-07-01 由 VP Finance 签字确认，依据 FY2026 毛利率标准。
- **2026-12-31 之后失效：** 成本分摊标准每年审查一次。消费方须在对外提供前依据 FY2027 标准重新校验。

[^margin-standard]: Cost Allocation & Margin Standard (FY2026)
[^revenue-policy]: Revenue Recognition Policy (FY2026)
