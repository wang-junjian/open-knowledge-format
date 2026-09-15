---
type: BigQuery Table
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/ga4_obfuscated_sample_ecommerce/tables/events_*
title: GA4 事件导出
description: 包含用户交互日志的 Google Analytics 4 事件级每日分片导出表。
tags:
- analytics
- e-commerce
- ga4
- sharded-tables
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T21:15:20+00:00'
sources:
- title: 'Google Analytics Help: BigQuery Export Schema'
  id: ga4-export-docs
  resource: https://support.google.com/analytics/answer/7029846
- title: BigQuery Table Metadata
  id: metadata
  resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/ga4_obfuscated_sample_ecommerce/tables/events_*
- title: Sample queries for audiences based on BigQuery data - Analytics Help
  resource: https://support.google.com/analytics/answer/9037342
  id: sample_queries
---

`events_` 表族包含来自 Google Merchandise Store 的模糊化 Google Analytics 4（GA4）事件级导出数据[^ga4-export-docs]。其结构为一系列每日分片表，从 `events_20201101` 到 `events_20210131`[^metadata]。每一行代表由用户与在线商店交互触发的单个事件（例如 `page_view`、`scroll`、`session_start`、`view_item`、`purchase`）。

该数据可用于行为分析、漏斗转化映射和电商追踪。关键的用户属性（如地理位置、设备平台和获客流量来源）嵌套在顶层记录中。此外，与特定事件或产品相关的参数存储在重复记录字段（`event_params` 和 `items`）中，查询时需要扁平化或展开（unnest）操作。

# Schema

以下是分片每日事件表的扁平化 schema 表示。

| Field Name | Type | Mode | Description |
| :--- | :--- | :--- | :--- |
| **event_date** | STRING | NULLABLE | 事件被记录时的日期（格式为 `YYYYMMDD`）。 |
| **event_timestamp** | INTEGER | NULLABLE | 事件被注册时的 POSIX 时间戳（微秒）。 |
| **event_name** | STRING | NULLABLE | 事件名称（例如 `page_view`、`purchase`、`session_start`）。 |
| **event_params** | RECORD | REPEATED | 与事件关联的键值参数。 |
| *event_params.key* | STRING | NULLABLE | 参数名称。 |
| *event_params.value* | RECORD | NULLABLE | 参数值，按数据类型嵌套。 |
| *event_params.value.string_value* | STRING | NULLABLE | 当参数值为字符串时的值。 |
| *event_params.value.int_value* | INTEGER | NULLABLE | 当参数值为整数时的值。 |
| *event_params.value.float_value* | FLOAT | NULLABLE | 当参数值为浮点数时的值。 |
| *event_params.value.double_value* | FLOAT | NULLABLE | 当参数值为双精度数时的值。 |
| **event_previous_timestamp** | INTEGER | NULLABLE | 上一个事件的时间戳（微秒）。 |
| **event_value_in_usd** | FLOAT | NULLABLE | 事件的货币价值，已转换为美元。 |
| **event_bundle_sequence_id** | INTEGER | NULLABLE | 上传 bundle 的序列 ID。 |
| **event_server_timestamp_offset** | INTEGER | NULLABLE | 服务器时间与设备记录时间之差。 |
| **user_id** | STRING | NULLABLE | 用户的唯一标识（登录时）。 |
| **user_pseudo_id** | STRING | NULLABLE | 用户的匿名设备标识（例如 GA 客户端 ID）。 |
| **privacy_info** | RECORD | NULLABLE | 同意与隐私设置。 |
| *privacy_info.analytics_storage* | INTEGER | NULLABLE | 分析存储同意状态。 |
| *privacy_info.ads_storage* | INTEGER | NULLABLE | 广告存储同意状态。 |
| *privacy_info.uses_transient_token* | STRING | NULLABLE | 是否使用临时令牌。 |
| **user_properties** | RECORD | REPEATED | 自定义用户属性。 |
| *user_properties.key* | INTEGER | NULLABLE | 属性名称/键。 |
| *user_properties.value* | RECORD | NULLABLE | 嵌套的自定义值及更新时间戳。 |
| **user_first_touch_timestamp** | INTEGER | NULLABLE | 用户首次与站点交互的时间（微秒）。 |
| **user_ltv** | RECORD | NULLABLE | 用户生命周期价值详情。 |
| *user_ltv.revenue* | FLOAT | NULLABLE | 归属于该用户的历史总收入。 |
| *user_ltv.currency* | STRING | NULLABLE | 生命周期收入值所使用的货币。 |
| **device** | RECORD | NULLABLE | 访客的设备信息。 |
| *device.category* | STRING | NULLABLE | 设备类别（例如 `mobile`、`desktop`、`tablet`）。 |
| *device.mobile_brand_name* | STRING | NULLABLE | 移动设备品牌名称（例如 `Apple`、`Samsung`）。 |
| *device.mobile_model_name* | STRING | NULLABLE | 移动设备型号名称。 |
| *device.mobile_marketing_name* | STRING | NULLABLE | 设备营销名称。 |
| *device.operating_system* | STRING | NULLABLE | 操作系统名称（例如 `iOS`、`Android`、`Web`）。 |
| *device.operating_system_version* | STRING | NULLABLE | 操作系统版本。 |
| *device.language* | STRING | NULLABLE | 浏览器/设备语言代码。 |
| *device.web_info.browser* | STRING | NULLABLE | Web 浏览器名称。 |
| *device.web_info.browser_version* | STRING | NULLABLE | Web 浏览器版本。 |
| **geo** | RECORD | NULLABLE | 由 IP 地址推导的地理信息。 |
| *geo.continent* | STRING | NULLABLE | 大洲名称。 |
| *geo.sub_continent* | STRING | NULLABLE | 次大洲名称。 |
| *geo.country* | STRING | NULLABLE | 国家名称。 |
| *geo.region* | STRING | NULLABLE | 地区或州名称。 |
| *geo.city* | STRING | NULLABLE | 城市名称。 |
| *geo.metro* | STRING | NULLABLE | 都会区名称。 |
| **app_info** | RECORD | NULLABLE | 应用特定信息。 |
| **traffic_source** | RECORD | NULLABLE | 用户获客来源。 |
| *traffic_source.medium* | STRING | NULLABLE | 媒介（例如 `organic`、`referral`、`cpc`）。 |
| *traffic_source.name* | STRING | NULLABLE | 广告系列名称。 |
| *traffic_source.source* | STRING | NULLABLE | 来源（例如 `google`、`direct`）。 |
| **stream_id** | INTEGER | NULLABLE | 数据流 ID。 |
| **platform** | STRING | NULLABLE | 数据采集平台（例如 `WEB`、`IOS`、`ANDROID`）。 |
| **event_dimensions** | RECORD | NULLABLE | 事件级元数据维度。 |
| *event_dimensions.hostname* | STRING | NULLABLE | 事件发生所在的目标主机名。 |
| **ecommerce** | RECORD | NULLABLE | 订单级交易明细。 |
| *ecommerce.total_item_quantity* | INTEGER | NULLABLE | 交易中的商品总数。 |
| *ecommerce.purchase_revenue_in_usd* | FLOAT | NULLABLE | 交易收入，已转换为美元。 |
| *ecommerce.transaction_id* | STRING | NULLABLE | 交易标识符。 |
| **items** | RECORD | REPEATED | 事件中涉及商品的商品级属性。 |
| *items.item_id* | STRING | NULLABLE | 商品 ID 或 SKU。 |
| *items.item_name* | STRING | NULLABLE | 商品名称。 |
| *items.item_brand* | STRING | NULLABLE | 商品品牌。 |
| *items.price_in_usd* | FLOAT | NULLABLE | 单价（美元）。 |
| *items.quantity* | INTEGER | NULLABLE | 商品数量。 |

# 常见查询模式

### 1. 按事件名称统计事件数与活跃用户数
本查询统计所记录的事件总数，并针对全部表范围内的每种事件类型统计去重用户数（`user_pseudo_id`）。

```sql
SELECT
  event_name,
  COUNT(1) AS event_count,
  COUNT(DISTINCT user_pseudo_id) AS unique_users
FROM
  `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
WHERE
  _TABLE_SUFFIX BETWEEN '20201101' AND '20210131'
GROUP BY
  1
ORDER BY
  event_count DESC;
```

### 2. 从 event_params 中提取嵌套的 page_location
由于 `event_params` 是一个重复记录（ARRAY），必须对其展开（unnest）或使用子查询进行过滤，才能提取出诸如页面浏览的 `page_location` 这样的特定参数。

```sql
SELECT
  event_date,
  (SELECT value.string_value FROM UNNEST(event_params) WHERE key = 'page_location') AS page_path,
  COUNT(1) AS page_views
FROM
  `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
WHERE
  event_name = 'page_view'
  AND _TABLE_SUFFIX BETWEEN '20210101' AND '20210115'
GROUP BY
  1, 2
ORDER BY
  page_views DESC;
```

### 3. 从 items 数组计算最畅销商品
为了分析商品销量，我们对购买事件上的重复 `items` 记录结构进行展开（unnest），并对数量进行聚合。

```sql
SELECT
  item.item_id,
  item.item_name,
  SUM(item.quantity) AS units_sold,
  ROUND(SUM(item.item_revenue_in_usd), 2) AS total_revenue_usd
FROM
  `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`,
  UNNEST(items) AS item
WHERE
  event_name = 'purchase'
  AND _TABLE_SUFFIX BETWEEN '20201101' AND '20210131'
GROUP BY
  1, 2
ORDER BY
  total_revenue_usd DESC
LIMIT 10;
```

# Metrics
以下预定义以及自定义的受众同期群指标均可从事件日志表中推导得出：
* [Purchasers](../references/metrics/purchasers.md) — 记录了 `in_app_purchase` 或 `purchase` 的用户。
* [N-Day Active Users](../references/metrics/n_day_active_users.md) — 在最近 N 天内记录了至少一个带有 `engagement_time_msec > 0` 的事件的用户。
* [N-Day Inactive Users](../references/metrics/n_day_inactive_users.md) — 最近 M 天内活跃、但在最近 N 天内未记录任何带有 `engagement_time_msec > 0` 的事件的用户（M > N）。
* [Frequently Active Users](../references/metrics/frequently_active_users.md) — 在最近 M 天中至少 N 天记录了带有 `engagement_time_msec > 0` 的事件的用户。
* [Highly Active Users](../references/metrics/highly_active_users.md) — 在最近 M 天中活跃/互动超过 N 分钟的用户。
* [Acquired Users](../references/metrics/acquired_users.md) — 通过特定广告系列来源、媒介和名称获取的用户。
* [Google Acquired Cohorts](../references/metrics/google_acquired_cohorts.md) — 在由 Google 广告系列来源过滤的特定每周同期群中获取的用户。

[^ga4-export-docs]: [Google Analytics 帮助：BigQuery 导出 Schema](https://support.google.com/analytics/answer/7029846)
[^metadata]: 源数据集 `ga4_obfuscated_sample_ecommerce` 的表清单元数据。
