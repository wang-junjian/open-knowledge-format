---
type: BigQuery Table
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/crypto_bitcoin/tables/inputs
title: 比特币交易输入
description: 比特币交易输入，详述所花费的 UTXO。
tags:
- bitcoin
- crypto
- blockchain
- utxo
- inputs
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T23:16:22+00:00'
sources:
- resource: https://github.com/blockchain-etl/bitcoin-etl
  title: Bitcoin ETL GitHub Repository
  id: bitcoin-etl
---

`inputs` 表包含比特币区块链上所有交易输入（被花费的 UTXO）的明细。每一行代表为给某笔交易提供资金而被消耗的一个输入 [^bitcoin-etl]。由于比特币采用未花费交易输出（UTXO）模型，每笔交易消耗既有输出（在新交易中成为“输入”），并创建新的输出 [^bitcoin-etl]。

该表特别适用于追踪资金流向、分析花费行为以及追溯交易谱系（lineage）。通过将输入的 `spent_transaction_hash` 与 `spent_output_index` 回链到 [outputs](outputs.md) 表，分析人员可完整重建交易图（transaction graph）。

数据使用开源工具 [bitcoin-etl](https://github.com/blockchain-etl/bitcoin-etl) 从区块链导出 [^bitcoin-etl]，存放于 [crypto_bitcoin](../datasets/crypto_bitcoin.md) 数据集。

# 表结构（Schema）

| Field Name | Type | Mode | Description |
| :--- | :--- | :--- | :--- |
| **transaction_hash** | STRING | NULLABLE | 包含本输入的交易哈希 |
| **block_hash** | STRING | NULLABLE | 包含本交易的区块哈希 |
| **block_number** | INTEGER | NULLABLE | 包含本交易的区块高度 |
| **block_timestamp** | TIMESTAMP | NULLABLE | 包含本交易的区块的时间戳 |
| **index** | INTEGER | NULLABLE | 本输入在交易内从 0 开始的索引 |
| **spent_transaction_hash** | STRING | NULLABLE | 本输入所花费输出所属交易的哈希 |
| **spent_output_index** | INTEGER | NULLABLE | 本输入在原交易中花费的输出的索引 |
| **script_asm** | STRING | NULLABLE | 输入脚本（scriptSig）的符号化表示 |
| **script_hex** | STRING | NULLABLE | 输入脚本（scriptSig）的十六进制表示 |
| **sequence** | INTEGER | NULLABLE | 交易输入序列号 |
| **required_signatures** | INTEGER | NULLABLE | 花费所需的签名数量（如适用） |
| **type** | STRING | NULLABLE | 脚本类型（如 `witness_v1_taproot`、`pubkeyhash`） |
| **addresses** | STRING | REPEATED | 与本输入关联的地址列表 |
| **value** | NUMERIC | NULLABLE | 被花费输出以 Satoshis 计的金额 |

# 常见查询模式（Common query patterns）

### 1. 识别给定时间段内最大的交易输入
该查询检索特定日期消耗的最大输入，演示如何发现大额 UTXO 合并（consolidations）或大额转账。

```sql
SELECT 
  block_timestamp,
  transaction_hash,
  value / 100000000 AS value_btc,
  addresses
FROM `bigquery-public-data.crypto_bitcoin.inputs`
WHERE block_timestamp >= '2024-04-17 00:00:00 UTC'
  AND block_timestamp < '2024-04-18 00:00:00 UTC'
ORDER BY value DESC
LIMIT 10;
```

### 2. 随时间追踪输入类型
通过按交易脚本类型分组统计输入，分析现代比特币脚本类型（如 Taproot）的采用情况。

```sql
SELECT 
  DATE(block_timestamp) AS block_date,
  type,
  COUNT(1) AS input_count,
  SUM(value) / 100000000 AS total_value_btc
FROM `bigquery-public-data.crypto_bitcoin.inputs`
WHERE block_timestamp >= '2024-01-01 00:00:00 UTC'
GROUP BY block_date, type
ORDER BY block_date DESC, input_count DESC;
```

### 3. 通过 join 输入与输出追溯来源溯源（provenance）
要查找交易中所花费资金的来源，可使用花费交易键将 inputs 表 join 到 outputs 表。

```sql
SELECT 
  inp.transaction_hash AS spending_tx,
  inp.block_timestamp AS spend_time,
  out.transaction_hash AS source_tx,
  out.block_timestamp AS source_time,
  inp.value / 100000000 AS value_btc
FROM `bigquery-public-data.crypto_bitcoin.inputs` AS inp
JOIN `bigquery-public-data.crypto_bitcoin.outputs` AS out
  ON inp.spent_transaction_hash = out.transaction_hash
  AND inp.spent_output_index = out.index
WHERE inp.block_timestamp >= '2024-04-17 00:00:00 UTC'
  AND inp.block_timestamp < '2024-04-17 01:00:00 UTC'
LIMIT 10;
```

# 关联（Joins）

- [transactions](../references/joins/inputs___transactions.md) — 将此被花费的输入关联到花费它的父交易记录。

[^bitcoin-etl]: https://github.com/blockchain-etl/bitcoin-etl
