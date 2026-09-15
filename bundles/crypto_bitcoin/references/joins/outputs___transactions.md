---
type: Reference
resource: https://github.com/blockchain-etl/bitcoin-etl
title: 交易到输出的 Join Path
description: 交易与输出之间的 join 路径，用于审计目标收款方分布。
tags:
- join
- bitcoin
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T23:15:59+00:00'
sources:
- title: Bitcoin ETL Parser
  resource: https://github.com/blockchain-etl/bitcoin-etl
  id: bitcoin-etl
---

该 join 路径将一个交易关联到其生成的输出。将 `transactions` 与 `outputs` join，有助于追踪资金如何从父交易分发（拆分或转发）到目标地址。

```sql
SELECT
  t.hash AS transaction_hash,
  t.block_timestamp AS transaction_timestamp,
  o.index AS output_index,
  o.addresses,
  o.value AS output_value_satoshis
FROM
  `bigquery-public-data.crypto_bitcoin.transactions` AS t
JOIN
  `bigquery-public-data.crypto_bitcoin.outputs` AS o
ON
  t.hash = o.transaction_hash;
```
