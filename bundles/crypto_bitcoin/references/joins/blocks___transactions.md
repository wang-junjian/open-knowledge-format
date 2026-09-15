---
type: Reference
resource: https://github.com/blockchain-etl/bitcoin-etl
title: 区块到交易的 Join Path
description: 通过区块高度 / 区块号将区块关联到其对应交易的 join 关系。
tags:
- join
- bitcoin
generated:
  by: reference_agent/gemini-3.5-flash
  at: '2026-07-10T23:15:51+00:00'
sources:
- id: bitcoin-etl
  resource: https://github.com/blockchain-etl/bitcoin-etl
  title: Bitcoin ETL Parser
---

该 join 路径表示区块与其中包含的所有交易之间的关联。它有助于分析区块密度、挖矿手续费占比，并验证相对于区块生产时间的交易确认耗时。

```sql
SELECT
  b.number AS block_height,
  b.hash AS block_hash,
  b.timestamp AS block_timestamp,
  t.hash AS transaction_hash,
  t.fee AS transaction_fee
FROM
  `bigquery-public-data.crypto_bitcoin.blocks` AS b
JOIN
  `bigquery-public-data.crypto_bitcoin.transactions` AS t
ON
  b.number = t.block_number;
```
