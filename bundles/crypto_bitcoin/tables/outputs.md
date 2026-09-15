---
type: BigQuery Table
resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/crypto_bitcoin/tables/outputs
title: 比特币输出表
description: 全部比特币交易的输出，包含脚本明细与以 Satoshis 计的金额。
tags:
- bitcoin
- blockchain
- crypto
- utxo
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T23:16:28+00:00'
sources:
- id: bitcoin-etl
  resource: https://github.com/blockchain-etl/bitcoin-etl
  title: Bitcoin ETL Export Tool
- title: BigQuery Bitcoin Outputs Table Metadata
  resource: https://bigquery.googleapis.com/v2/projects/bigquery-public-data/datasets/crypto_bitcoin/tables/outputs
  id: outputs-table
---

`outputs` 表包含结构化数据，代表比特币区块链中全部交易输出（在被花费前也称为 UTXO 或 Unspent Transaction Outputs）。每一行代表由某笔交易生成的单个输出，指定了 satoshis 金额（value）以及花费它所需的密码学条件（锁定脚本，locking script）。

数据使用开源工具 `bitcoin-etl` 从区块链账本提取并导出 [^bitcoin-etl]。该表属于 [crypto_bitcoin](../datasets/crypto_bitcoin.md) 数据集，可与 [transactions](transactions.md)、[inputs](inputs.md) 等兄弟表 join，以重建完整交易谱系并追踪资金在网络中的流动。

### 粒度与解读（Grain and Interpretation）

该表的粒度为**每笔交易输出一行**，由 `transaction_hash` 与输出 `index` 的组合唯一标识。`value` 字段表示以 Satoshis 计的输出金额（1 BTC = 100,000,000 Satoshis），以高精度 `NUMERIC` 类型表示。

# 表结构（Schema）

| Field Name | Type | Mode | Description |
|---|---|---|---|
| `transaction_hash` | STRING | NULLABLE | 包含本输出的交易哈希。 |
| `block_hash` | STRING | NULLABLE | 包含本交易的区块哈希。 |
| `block_number` | INTEGER | NULLABLE | 区块号/高度。 |
| `block_timestamp` | TIMESTAMP | NULLABLE | 区块被挖出时的时间戳。 |
| `index` | INTEGER | NULLABLE | 输出在交易内从 0 开始的索引。 |
| `script_asm` | STRING | NULLABLE | 锁定脚本的符号化（Assembly）表示。 |
| `script_hex` | STRING | NULLABLE | 锁定脚本的十六进制表示。 |
| `required_signatures` | INTEGER | NULLABLE | 花费该输出所需的签名数量（常见地址通常为 1）。 |
| `type` | STRING | NULLABLE | 脚本类型（如 `pubkeyhash`、`scripthash`）。 |
| `addresses` | STRING | REPEATED | 与本输出关联的比特币地址列表（通常仅含一个地址）。 |
| `value` | NUMERIC | NULLABLE | 该输出以 Satoshis 计的金额（1 BTC = 100,000,000 Satoshis）。 |

# 常见查询模式（Common query patterns）

### 1. 计算特定日期生成的交易输出总价值
以下查询聚合给定日期的输出金额，求出发行的总量，并将 Satoshis 转换为比特币（BTC）。

```sql
SELECT 
  DATE(block_timestamp) AS date,
  SUM(value) / 100000000.0 AS total_btc_volume,
  COUNT(1) AS output_count
FROM `bigquery-public-data.crypto_bitcoin.outputs`
WHERE block_timestamp >= '2023-01-01 00:00:00 UTC'
  AND block_timestamp < '2023-01-02 00:00:00 UTC'
GROUP BY 1;
```

### 2. 查找给定区块中最大的交易输出
该查询列出区块 301641 中价值最高的输出及其收款地址。

```sql
SELECT 
  transaction_hash,
  `index`,
  addresses,
  value / 100000000.0 AS btc_value,
  type
FROM `bigquery-public-data.crypto_bitcoin.outputs`
WHERE block_number = 301641
ORDER BY value DESC
LIMIT 5;
```

### 3. 分析特定时间范围内的输出类型
该查询识别不同脚本锁定类型（如 `pubkeyhash` 或 `scripthash`）在每周时间范围内的流行度。

```sql
SELECT 
  type,
  COUNT(1) AS output_count,
  SUM(value) / 100000000.0 AS total_btc
FROM `bigquery-public-data.crypto_bitcoin.outputs`
WHERE block_timestamp >= '2023-06-01 00:00:00 UTC'
  AND block_timestamp < '2023-06-08 00:00:00 UTC'
GROUP BY type
ORDER BY output_count DESC;
```

# 关联（Joins）

- [transactions](../references/joins/outputs___transactions.md) — 将此输出回链到创建它的父交易记录。

[^bitcoin-etl]: Blockchain ETL Bitcoin Extractor: https://github.com/blockchain-etl/bitcoin-etl
