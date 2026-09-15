---
type: BigQuery Table
title: 客户订单
description: 覆盖 web、移动端、市场渠道的每笔已完成客户订单一行。粒度（grain）为订单，而非行项目；逐行商品明细位于 `order_lines`。
resource: https://console.cloud.google.com/bigquery?p=acme&d=sales&t=orders
tags: [sales, orders, revenue]
generated: { by: reference_agent/gemini-2.5-pro, at: 2026-06-30T14:00:00Z }
verified:
  - { by: human:kliu@acme, at: 2026-07-01T16:00:00Z }
status: stable
stale_after: 2026-12-31T00:00:00Z
sources:
  - id: warehouse-schema
    resource: https://wiki.acme.internal/data/warehouse/schemas/sales
    title: Acme Retail warehouse schema — sales dataset
    author: team:data-platform
    usage_count: 1240
    last_modified: 2026-06-15T00:00:00Z
  - id: revenue-policy
    resource: policies/revenue-recognition.md
    title: Revenue Recognition Policy (FY2026)
    author: human:jsmith@acme
    last_modified: 2026-06-15T00:00:00Z
usage_window: { from: 2026-04-01T00:00:00Z, to: 2026-06-30T00:00:00Z }
---

# Schema

| Column | Type | Description |
|---|---|---|
| `order_id` | STRING | 全局唯一的订单 id，于订单创建时生成。[^warehouse-schema] |
| `customer_id` | STRING | 指向 `customers` 的外键（FK）。已完成订单中永不为空。[^warehouse-schema] |
| `order_ts` | TIMESTAMP | 下单时间（UTC）。用于财年归属的时间戳。[^revenue-policy] |
| `order_status` | STRING | 取值之一：`pending`、`paid`、`shipped`、`delivered`、`cancelled`、`refunded`。仅当 `order_status = 'delivered'` 且 30 天退货窗口已关闭时才确认收入。[^revenue-policy] |
| `gross_amount` | NUMERIC(18,4) | 折扣前小计，以 `currency` 计。不含税与运费。[^warehouse-schema] |
| `discount_amount` | NUMERIC(18,4) | 应用的折扣总额（促销码、会员积分、价格调整）。[^warehouse-schema] |
| `net_amount` | NUMERIC(18,4) | `gross_amount - discount_amount`。即依据策略的已确认收入金额。[^revenue-policy] |
| `shipping_amount` | NUMERIC(18,4) | 向客户计收的承运商费用。不计入收入（过账负债）。[^revenue-policy] |
| `tax_amount` | NUMERIC(18,4) | 代收销售税。不计入收入（过账负债）。[^revenue-policy] |
| `currency` | STRING | ISO 4217 货币代码。非 USD 订单按 `order_ts` 日期经 `finance.fx_daily_rates` 换算。[^revenue-policy] |
| `channel` | STRING | 订单来源：`web`、`mobile`、`marketplace`。市场渠道订单（Amazon、eBay）按净额结算，在平台打款时确认，而非在 `order_status = 'delivered'` 时确认。 |

# 给消费方的说明（Notes for consumers）

- 粒度假设常让新分析师踩坑：`SUM(net_amount) GROUP BY order_id` 是空操作（no-op），因为每笔订单恰好只有一行。要获取按 SKU 的收入，请 join `order_lines`。
- 本表中 `refunded` 状态为终止状态；退款事件本身位于 `finance.refunds`，以 `order_id` 为键。

[^warehouse-schema]: Acme Retail warehouse schema — sales dataset
[^revenue-policy]: Revenue Recognition Policy (FY2026)
