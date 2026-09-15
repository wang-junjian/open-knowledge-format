---
type: Attested Computation
title: 某一期间的毛利率
description: 依据 Acme FY2026 成本分摊标准（完整 COGS = 商品成本 + 履约成本 + 运费 + 支付手续费）生成的某期间毛利率数值的授权 SQL。
tags: [finance, margin, attested]
runtime: bigquery
parameters:
  - { name: period_start, type: date, required: true }
  - { name: period_end, type: date, required: true }
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

# Computation

```sql
WITH recognized_orders AS (
  SELECT
    o.order_id,
    CASE
      WHEN o.currency = 'USD' THEN o.net_amount
      ELSE o.net_amount * fx.rate_to_usd
    END AS revenue_usd
  FROM `acme.sales.orders` AS o
  LEFT JOIN `acme.finance.fx_daily_rates` AS fx
    ON fx.currency = o.currency
    AND fx.rate_date = DATE(o.order_ts)
  WHERE o.order_status = 'delivered'
    AND DATE_DIFF(CURRENT_DATE(), DATE(o.order_ts), DAY) >= 30
    AND DATE(o.order_ts) BETWEEN @period_start AND @period_end
),
cogs_full AS (
  SELECT
    ol.order_id,
    SUM(ol.quantity * p.cost) AS product_cost,
    SUM(fc.allocated_cost) AS fulfillment_cost,
    SUM(sc.shipping_cost) AS shipping_cost,
    SUM(pf.fee_amount) AS payment_fee
  FROM `acme.sales.order_lines` AS ol
  JOIN `acme.catalog.products` AS p ON p.product_id = ol.product_id
  LEFT JOIN `acme.logistics.fulfillment_cost` AS fc ON fc.order_id = ol.order_id
  LEFT JOIN `acme.logistics.shipment_cost` AS sc ON sc.order_id = ol.order_id
  LEFT JOIN `acme.finance.payment_fees` AS pf ON pf.order_id = ol.order_id
  GROUP BY ol.order_id
)
SELECT
  SUM(r.revenue_usd) - SUM(
    COALESCE(c.product_cost, 0)
    + COALESCE(c.fulfillment_cost, 0)
    + COALESCE(c.shipping_cost, 0)
    + COALESCE(c.payment_fee, 0)
  ) AS gross_margin_usd
FROM recognized_orders AS r
LEFT JOIN cogs_full AS c USING (order_id)
```

# 关于 COGS 构成的说明

依据 FY2026 成本分摊标准，四项 COGS 构成缺一不可。[^margin-standard] 若执行 SQL 丢弃了 `cogs_full` 上的任意一个 LEFT JOIN，其回执将无法通过认证（attestation），因为规范化 SQL 的相等性校验包含 join 图（join graph）。

收入侧依据策略，使用与 [`computations/revenue-ytd.md`](./revenue-ytd.md) 相同的确认规则。

# 时效（Freshness）

`stale_after: 2026-12-31T00:00:00Z` 与成本分摊标准的年度审查周期一致。该标准预期在本财年内保持稳定，但在 2027-01-01 之后运行此计算的消费方 MUST 重新依据 FY2027 标准校验，方可对外提供。

[^margin-standard]: Cost Allocation & Margin Standard (FY2026)
[^revenue-policy]: Revenue Recognition Policy (FY2026)
