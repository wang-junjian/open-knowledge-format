---
type: Metric
title: 收入
description: 依据 Acme FY2026 收入确认策略的某期间已确认收入。由 Attested Computation 支撑。
tags: [finance, revenue, headline-metric]
generated: { by: reference_agent/gemini-2.5-pro, at: 2026-06-30T14:00:00Z }
verified:
  - { by: human:jsmith@acme, at: 2026-07-01T09:00:00Z }
status: stable
stale_after: 2026-12-31T00:00:00Z
sources:
  - id: revenue-policy
    resource: policies/revenue-recognition.md
    title: Revenue Recognition Policy (FY2026)
    author: human:jsmith@acme
    last_modified: 2026-06-15T00:00:00Z
---

# 定义

某一财年的收入，是对满足以下条件的订单 `net_amount` 求和：(a) 达到 `order_status = 'delivered'`，(b) 已完成 30 天退货窗口，且 (c) 按 `order_ts` 归属该财年。多币种订单按 `order_ts` 当日参考汇率换算为 USD。[^revenue-policy]

授权计算逻辑为 [`computations/revenue-ytd.md`](../computations/revenue-ytd.md)。消费方 MUST 运行并认证该计算逻辑，而不得自行编写 SUM。校验器会拒绝任何执行 SQL 与授权形式不符的回执。

# 报表维度（Reporting cuts）

- **按财年：** 授权计算逻辑以 `year` 为唯一参数。
- **按渠道或品类：** 这些属于经批准的叙事，并非新指标。请在客户端将回执的逐行结果 join 到 `orders.channel`，或 join 到 `order_lines` × `products.category`。Do NOT 改写授权 SQL。

# 信任与时效

- **已校验（Verified）：** 2026-07-01 由 VP Finance 签字确认，依据 FY2026 策略。
- **2026-12-31 之后失效：** Finance 每年一月重新发布收入确认策略。在 2027-01-01 之后使用此概念的消费方 MUST 在对外提供前，依据新策略重新校验该定义。

[^revenue-policy]: Revenue Recognition Policy (FY2026)
