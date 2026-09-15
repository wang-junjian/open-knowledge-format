---
type: Attested Computation
title: 某一财年的收入
description: 依据 Acme FY2026 收入确认策略，生成指定财年已确认收入数值的授权 SQL。
tags: [finance, revenue, attested]
runtime: bigquery
parameters:
  - { name: year, type: integer, required: true }
executor:
  resource: skills/run-on-bq.md
  receipt: [job_id, executed_sql, result]
attester:
  resource: attesters/sql_equality.py
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
  - id: orders-table
    resource: tables/orders.md
    title: Customer Orders (BigQuery table)
    author: team:data-platform
    last_modified: 2026-07-01T00:00:00Z
---

# Computation

```sql
SELECT
  SUM(
    CASE
      WHEN o.currency = 'USD' THEN o.net_amount
      ELSE o.net_amount * fx.rate_to_usd
    END
  ) AS revenue_usd
FROM `acme.sales.orders` AS o
LEFT JOIN `acme.finance.fx_daily_rates` AS fx
  ON fx.currency = o.currency
  AND fx.rate_date = DATE(o.order_ts)
WHERE o.order_status = 'delivered'
  AND DATE_DIFF(CURRENT_DATE(), DATE(o.order_ts), DAY) >= 30
  AND EXTRACT(YEAR FROM o.order_ts) = @year
```

本计算逻辑实现了 FY2026 收入确认策略的四条规则：[^revenue-policy]

1. **确认触发条件（Recognition trigger）：** `order_status = 'delivered'` 且 30 天退货窗口已关闭。
2. **确认金额（Recognized amount）：** `net_amount`（不含运费与税）。
3. **币种（Currency）：** 非 USD 订单按 `order_ts` 当日汇率换算。
4. **财年（Fiscal year）：** 取自 `order_ts` 的日历年。

# 校验器（attester）检查的内容

`attesters/sql_equality.py` 接收 `skills/run-on-bq.md` 返回的回执，并校验两项内容：

1. **来源溯源（Provenance）：** `receipt.executed_sql` 经规范化（去空白、去除注释、关键字大小写统一）后，须与上方 SQL 以相同方式规范化的结果相等。任何改写（替换表、新增过滤条件、丢弃 JOIN）都将导致校验失败。
2. **保真度（Fidelity）：** 调用方即将展示的数值须等于 `receipt.result[0]`。

SQL 不匹配的运行被视为未经认证（unattested）；消费方 MUST 拒绝展示该数值。

# 时效（Freshness）

`stale_after: 2026-12-31T00:00:00Z` 与收入确认策略的年度审查周期一致。按具备记忆感知的消费方契约，在 2027-01-01，运行此计算的消费方 SHOULD 在对外提供结果前，将其标记需重新校验。

[^revenue-policy]: Revenue Recognition Policy (FY2026)
