---
type: Reference
resource: https://github.com/blockchain-etl/bitcoin-etl
title: 交易到输入的 Join Path
description: 交易与输入之间的 join 路径，用于追溯资金消耗明细。
tags:
- join
- bitcoin
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T23:15:56+00:00'
sources:
- id: bitcoin-etl
  title: Bitcoin ETL Parser
  resource: https://github.com/blockchain-etl/bitcoin-etl
---

该 join 路径将交易关联到其输入。在 UTXO（未花费交易输出，Unspent Transaction Output）数据库结构中，将主 `transactions` 表与扁平的 `inputs` 表 join，可供分析人员审计交易中所消耗资金的历史来源。

```sql
SELECT
  t.hash AS transaction_hash,
  t.block_timestamp AS transaction_timestamp,
  i.index AS input_index,
  i.spent_transaction_hash,
  i.spent_output_index,
  i.value AS input_value_satoshis
FROM
  `bigquery-public-data.crypto_bitcoin.transactions` AS t
JOIN
  `bigquery-public-data.crypto_bitcoin.inputs` AS i
ON
  t.hash = i.transaction_hash;
```
