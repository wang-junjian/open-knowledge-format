---
type: BigQuery Dataset
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/crypto_bitcoin
title: 比特币区块链数据集
description: 一个公开的 Google BigQuery 数据集，包含比特币区块链完整的交易账本与区块历史。
tags:
- bitcoin
- blockchain
- crypto
- public-data
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T23:14:11+00:00'
sources:
- title: BigQuery Dataset Metadata - crypto_bitcoin
  resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/crypto_bitcoin
  id: bq-crypto-bitcoin-meta
---

`crypto_bitcoin` 数据集是一个公开的 Google BigQuery 数据集，包含比特币完整的区块链交易历史。它持续更新，并以高度结构化、可查询的格式提供从创世区块（genesis block）起的区块与交易数据。

该数据集包含四张主表：
- [blocks](../tables/blocks.md)：代表比特币区块，包含哈希、大小、交易数量与区块奖励（block rewards）。
- [transactions](../tables/transactions.md)：包含顶层交易明细，如输入/输出总额、手续费与加密签名。
- [inputs](../tables/inputs.md)：包含交易输入（花费先前输出），代表资金来源。
- [outputs](../tables/outputs.md)：包含交易输出，代表资金去向（地址与金额）。

该数据集广泛用于区块链取证（forensics）、交易量的宏观经济分析、钱包余额追踪以及挖矿活动研究。

# 表结构（Schema）

作为 BigQuery 数据集，`crypto_bitcoin` 充当以下各表的命名空间与容器：

| Table ID | Description |
| :--- | :--- |
| **[blocks](../tables/blocks.md)** | 包含已验证并写入账本的交易区块。 |
| **[transactions](../tables/transactions.md)** | 在参与者之间转移价值的独立账本条目。 |
| **[inputs](../tables/inputs.md)** | 指向交易中正被花费的 UTXO（未花费交易输出，Unspent Transaction Outputs）的引用。 |
| **[outputs](../tables/outputs.md)** | 由交易创建的、成为新 UTXO 的输出。 |

# 常见查询模式（Common query patterns）

### 1. 按月统计区块数量及每块平均交易数
该查询计算每月的区块数量，以及每块包含的平均交易数。

```sql
SELECT
  TIMESTAMP_TRUNC(timestamp, MONTH) AS month,
  COUNT(1) AS total_blocks,
  AVG(transaction_count) AS avg_transactions_per_block
FROM
  `bigquery-public-data.crypto_bitcoin.blocks`
GROUP BY
  month
ORDER BY
  month DESC
LIMIT 12;
```

### 2. 最近 30 天的交易手续费统计（以 Satoshis 计）
该查询探查近期交易的手续费分布。

```sql
SELECT
  MIN(fee) AS min_fee,
  MAX(fee) AS max_fee,
  AVG(fee) AS avg_fee,
  APPROX_QUANTILES(fee, 2)[OFFSET(1)] AS median_fee
FROM
  `bigquery-public-data.crypto_bitcoin.transactions`
WHERE
  block_timestamp >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY);
```
