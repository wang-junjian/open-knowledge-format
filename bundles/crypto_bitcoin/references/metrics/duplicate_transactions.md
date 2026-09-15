---
type: Reference
resource: https://cloud.google.com/blog/topics/public-datasets/bitcoin-in-bigquery-blockchain-analytics-on-public-data
title: 重复交易指标
description: 用于查找跨不同区块的历史重复交易的异常检测指标（metric）。
tags:
- metric
- anomaly-detection
- bitcoin
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T23:15:47+00:00'
sources:
- id: gcp-blog
  title: 'Bitcoin in BigQuery: blockchain analytics on public data'
  resource: https://cloud.google.com/blog/topics/public-datasets/bitcoin-in-bigquery-blockchain-analytics-on-public-data
---

该异常查询模式用于识别出现在多个区块中的交易。历史上，在比特币区块链中，由于最初的 BerkeleyDB 数据库引擎允许非唯一键的行为，交易可能被复制。后来通过实施比特币改进提案 [BIP-0030](https://github.com/bitcoin/bips/blob/master/bip-0030.mediawiki) 并过渡到 LevelDB 解决了该问题。

### standardSQL
```sql
SELECT
  transaction_id,
  COUNT(transaction_id) AS dup_transaction_count
FROM (
  SELECT
    hash AS transaction_id
  FROM
    `bigquery-public-data.crypto_bitcoin.transactions`
)
GROUP BY
  transaction_id
HAVING
  dup_transaction_count > 1;
```
