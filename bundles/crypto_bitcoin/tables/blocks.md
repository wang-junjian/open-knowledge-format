---
type: BigQuery Table
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/crypto_bitcoin/tables/blocks
title: 比特币区块表
description: 比特币区块链的全部区块，包含区块头、交易数量、大小与时间戳。
tags:
- bitcoin
- blockchain
- crypto
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T23:16:06+00:00'
sources:
- resource: https://github.com/blockchain-etl/bitcoin-etl
  title: Bitcoin ETL Export Tool
  id: bitcoin-etl
- id: bip-141
  resource: https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki
  title: BIP-141 Segregated Witness (Consensus layer)
---

`blocks` 表包含比特币区块链中每个区块的结构化记录 [^bitcoin-etl]。该表的每一行代表一个区块，记录了详细的区块头属性，如哈希、大小、交易数量、nonce、难度位（difficulty bits）以及该区块内所有交易的默克尔根（Merkle root）。

该数据集持续从活跃节点导出，代表自 2009 年 1 月创世区块起的比特币区块完整历史索引。该表按 `timestamp_month` 列做按月分区，以优化按日期筛选区块时的查询性能并降低数据扫描成本。

该表可与 [transactions](transactions.md) 做 join，下钻到单笔支付，或聚合区块级统计，如总交易手续费、交易密度与见证数据权重。

# 表结构（Schema）

| Field Name | Type | Mode | Description |
| :--- | :--- | :--- | :--- |
| **hash** | STRING | REQUIRED | 唯一标识该区块的区块哈希。 |
| **size** | INTEGER | NULLABLE | 区块数据的总大小（字节）。 |
| **stripped_size** | INTEGER | NULLABLE | 排除见证数据（witness data）后区块数据的大小（字节）。 |
| **weight** | INTEGER | NULLABLE | 依据 BIP-141 [^bip-141] 定义的“基础大小的三倍加上总大小”。 |
| **number** | INTEGER | REQUIRED | 区块的连续高度（序号）。 |
| **version** | INTEGER | NULLABLE | 区块头中指定的协议版本。 |
| **merkle_root** | STRING | NULLABLE | 默克尔树（Merkle tree）的根节点，其叶子为交易哈希。 |
| **timestamp** | TIMESTAMP | REQUIRED | 区块头中指定的区块创建时间戳。 |
| **timestamp_month** | DATE | REQUIRED | 区块创建时间戳的月份（用作分区键）。 |
| **nonce** | STRING | NULLABLE | 区块头中指定的难度解。 |
| **bits** | STRING | NULLABLE | 区块头中指定的难度阈值。 |
| **coinbase_param** | STRING | NULLABLE | 本区块 coinbase 交易中指定的数据。 |
| **transaction_count** | INTEGER | NULLABLE | 本区块包含的交易数量。 |

# 常见查询模式（Common query patterns）

### 1. 每日区块数量与每块平均交易数
了解每天挖出多少区块，以及特定月份每块的平均交易数。

```sql
SELECT
  DATE(timestamp) AS block_date,
  COUNT(1) AS blocks_mined,
  AVG(transaction_count) AS avg_transactions_per_block,
  SUM(transaction_count) AS total_transactions
FROM
  `bigquery-public-data.crypto_bitcoin.blocks`
WHERE
  timestamp_month = '2023-10-01'
GROUP BY
  block_date
ORDER BY
  block_date ASC;
```

### 2. 检索特定区块高度的详情
使用高度编号查询单个区块的元数据与结构。

```sql
SELECT
  number,
  hash,
  timestamp,
  size,
  transaction_count,
  version,
  coinbase_param
FROM
  `bigquery-public-data.crypto_bitcoin.blocks`
WHERE
  number = 800000;
```

### 3. 计算月度平均区块大小与权重
通过分析区块大小、剥离大小（stripped size）与 SegWit 权重的趋势，分析 SegWit 随时间的采用与影响 [^bip-141]。

```sql
SELECT
  timestamp_month,
  COUNT(1) AS blocks_mined,
  AVG(size) AS avg_block_size_bytes,
  AVG(stripped_size) AS avg_stripped_size_bytes,
  AVG(weight) AS avg_weight
FROM
  `bigquery-public-data.crypto_bitcoin.blocks`
WHERE
  timestamp_month >= '2020-01-01'
GROUP BY
  timestamp_month
ORDER BY
  timestamp_month DESC;
```

# 关联（Joins）

- [transactions](../references/joins/blocks___transactions.md) — 将区块关联到其包含的全部交易，用于追溯区块验证耗时、矿工手续费收入或交易密度。

[^bitcoin-etl]: https://github.com/blockchain-etl/bitcoin-etl
[^bip-141]: https://github.com/bitcoin/bips/blob/master/bip-0141.mediawiki
