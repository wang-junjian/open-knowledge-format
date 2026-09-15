---
type: BigQuery Table
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/crypto_bitcoin/tables/transactions
title: 比特币交易表
description: 全部比特币交易，包含输入、输出、区块元数据与手续费结构。
tags:
- bitcoin
- crypto
- blockchain
- transactions
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T23:16:14+00:00'
sources:
- title: Bitcoin ETL Parser
  resource: https://github.com/blockchain-etl/bitcoin-etl
  id: bitcoin-etl
- resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/crypto_bitcoin/tables/transactions
  id: bq-metadata
  title: BigQuery transactions Table Schema
- resource: https://cloud.google.com/blog/topics/public-datasets/bitcoin-in-bigquery-blockchain-analytics-on-public-data
  id: gcp-blog
  title: 'Bitcoin in BigQuery: blockchain analytics on public data'
---

本表包含自 2009 年 1 月创世区块起的全部比特币交易。数据使用开源工具 `bitcoin-etl` 从比特币区块链导出 [^bitcoin-etl]。该表的粒度为每笔交易一行。

每笔交易包含结构化数据，如哈希、大小、coinbase 标志、区块元数据（如区块号、哈希与时间戳）、手续费，以及表示其花费输入与生成输出的嵌套记录。为执行高性价比的查询，该表按 `block_timestamp_month` 列分区。

本表直接关联到 [crypto_bitcoin](../datasets/crypto_bitcoin.md) 数据集中的多个兄弟表，例如 [blocks](blocks.md)。虽然输入与输出在此处作为重复记录嵌套，它们也被扁平化为专用的兄弟表：[inputs](inputs.md) 与 [outputs](outputs.md)。

# 表结构（Schema）

| Field Name | Type | Mode | Description |
|---|---|---|---|
| **hash** | STRING | REQUIRED | 本交易的唯一 SHA-256 哈希 |
| **size** | INTEGER | NULLABLE | 本交易的大小（字节） |
| **virtual_size** | INTEGER | NULLABLE | 虚拟交易大小（对于 SegWit/见证交易不同于 size） |
| **version** | INTEGER | NULLABLE | 包含本交易的区块中指定的协议版本 |
| **lock_time** | INTEGER | NULLABLE | 矿工可将该交易纳入的最早时间/区块高度 |
| **block_hash** | STRING | REQUIRED | 包含本交易的区块的哈希 |
| **block_number** | INTEGER | REQUIRED | 包含本交易的区块的编号 |
| **block_timestamp** | TIMESTAMP | REQUIRED | 包含本交易的区块的时间戳 |
| **block_timestamp_month** | DATE | REQUIRED | 分区列；包含本交易的区块的月份 |
| **input_count** | INTEGER | NULLABLE | 交易中输入的数量 |
| **output_count** | INTEGER | NULLABLE | 交易中输出的数量 |
| **input_value** | NUMERIC | NULLABLE | 交易中输入的总金额 |
| **output_value** | NUMERIC | NULLABLE | 交易中输出的总金额 |
| **is_coinbase** | BOOLEAN | NULLABLE | 若本交易为 coinbase 交易（挖出的区块奖励）则为 true |
| **fee** | NUMERIC | NULLABLE | 支付给矿工的交易手续费（input_value - output_value） |
| **inputs** | RECORD | REPEATED | 交易输入的嵌套数组 |
| *inputs.***index** | INTEGER | REQUIRED | 交易内某输入的从 0 开始的编号 |
| *inputs.***spent_transaction_hash** | STRING | NULLABLE | 本输入所花费的输出所属交易的哈希 |
| *inputs.***spent_output_index** | INTEGER | NULLABLE | 本输入所花费的输出的索引 |
| *inputs.***script_asm** | STRING | NULLABLE | 脚本签名的符号化表示 |
| *inputs.***script_hex** | STRING | NULLABLE | 脚本签名的十六进制表示 |
| *inputs.***sequence** | INTEGER | NULLABLE | 用于 locktime 修改的序列号 |
| *inputs.***required_signatures** | INTEGER | NULLABLE | 授权该被花费输出所需的签名数量 |
| *inputs.***type** | STRING | NULLABLE | 被花费输出的地址类型（如 pubkeyhash、scripthash） |
| *inputs.***addresses** | STRING | REPEATED | 拥有被花费输出的地址数组 |
| *inputs.***value** | NUMERIC | NULLABLE | 附加在被花费输出上的基础货币（satoshis）金额 |
| **outputs** | RECORD | REPEATED | 交易输出的嵌套数组 |
| *outputs.***index** | INTEGER | REQUIRED | 用于后续引用该特定输出的从 0 开始的编号 |
| *outputs.***script_asm** | STRING | NULLABLE | 脚本公钥（script pubkey）的符号化表示 |
| *outputs.***script_hex** | STRING | NULLABLE | 脚本公钥的十六进制表示 |
| *outputs.***required_signatures** | INTEGER | NULLABLE | 授权花费该输出所需的签名数量 |
| *outputs.***type** | STRING | NULLABLE | 该输出的地址类型 |
| *outputs.***addresses** | STRING | REPEATED | 拥有该输出的地址数组 |
| *outputs.***value** | NUMERIC | NULLABLE | 附加在该输出上的基础货币（satoshis）金额 |

# 常见查询模式（Common query patterns）

### 1. 计算一个月内的平均交易手续费与大小
以下查询聚合特定分区月份的每日交易量、平均手续费与平均大小。

```sql
SELECT
  DATE(block_timestamp) AS transaction_date,
  COUNT(1) AS transaction_count,
  AVG(fee) AS avg_fee_satoshis,
  AVG(size) AS avg_size_bytes
FROM
  `bigquery-public-data.crypto_bitcoin.transactions`
WHERE
  block_timestamp_month = '2023-10-01'
GROUP BY
  transaction_date
ORDER BY
  transaction_date;
```

### 2. 识别给定月份中价值最高的交易
该查询按输出金额检索最大的交易，排除 coinbase 交易（区块奖励）。

```sql
SELECT
  hash,
  block_number,
  block_timestamp,
  output_count,
  output_value
FROM
  `bigquery-public-data.crypto_bitcoin.transactions`
WHERE
  block_timestamp_month = '2023-10-01'
  AND is_coinbase = FALSE
ORDER BY
  output_value DESC
LIMIT 10;
```

### 3. 分析输出类型与金额（unnesting 重复记录）
要分析不同比特币地址类型的分布（如 `scripthash` 或 `witness_v0_keyhash`），必须对 `outputs` 重复记录做 UNNEST。

```sql
SELECT
  out.type AS address_type,
  COUNT(1) AS output_count,
  SUM(out.value) AS total_value
FROM
  `bigquery-public-data.crypto_bitcoin.transactions`,
  UNNEST(outputs) AS out
WHERE
  block_timestamp_month = '2023-10-01'
GROUP BY
  address_type
ORDER BY
  total_value DESC;
```

# 指标（Metrics）

- [Duplicate transactions across blocks](../references/metrics/duplicate_transactions.md) — 用于在 BIP-0030 实施前发现旧重复交易 ID 的异常检测查询。

# 关联（Joins）

- [blocks](../references/joins/blocks___transactions.md) — 关联区块元数据，查找挖出本交易的区块。
- [inputs](../references/joins/inputs___transactions.md) — 将交易关联到其 UTXO 花费来源。
- [outputs](../references/joins/outputs___transactions.md) — 将交易关联到其输出回执。

[^bitcoin-etl]: Blockchain ETL on GitHub: https://github.com/blockchain-etl/bitcoin-etl
