# Bitcoin 公共数据集样例

对公共数据集 `bigquery-public-data.crypto_bitcoin`（blocks、transactions、inputs、outputs —— 由开源 `bitcoin-etl` 流水线生成）运行 reference agent，并以规范 schema 来源与奠基性的 Google Cloud blockchain-on-BigQuery 公告为 web pass 提供种子。

本样例与 GA4（单一反规范化 events 表）和 Stack Overflow（众多独立实体）形成对比，它演练的是**一组紧密相关的 fact 表**，其中 `transactions` 中的每一行都引用 `blocks`、`inputs` 与 `outputs` 中的行。适合用于观察 agent 如何在散文中揭示跨表外键关系。

## 先决条件

- 安装 agent（在仓库根目录执行）：
  ```
  python3.13 -m venv .venv
  .venv/bin/pip install --index-url https://pypi.org/simple/ -e .[dev]
  ```
- BigQuery 访问：
  ```
  gcloud auth application-default login
  gcloud config set project <your-billing-project>
  ```
  公共数据集可读，但查询字节数会记在调用方项目账上。`crypto_bitcoin` 表非常庞大（`transactions` 约数百 GB）—— 迭代时请将 `--web-max-pages` 保持在较小值，并优先使用 `--concept` 进行冒烟运行。
- Gemini 凭据 —— 二选一：`GEMINI_API_KEY`（AI Studio）**或** Vertex AI（`GOOGLE_GENAI_USE_VERTEXAI=true`、`GOOGLE_CLOUD_PROJECT=<id>`、`GOOGLE_CLOUD_LOCATION=<region>`）。

## 运行

```
.venv/bin/python -m reference_agent enrich \
    --source bq \
    --dataset bigquery-public-data.crypto_bitcoin \
    --web-seed-file samples/crypto_bitcoin/seeds.txt \
    --out ./bundles/crypto_bitcoin
```

要对单个概念迭代，添加 `--concept tables/transactions`。要跳过 web pass，添加 `--no-web`。要调高或调低 web 预算，使用 `--web-max-pages N`（默认 100）。

## 你会得到什么

位于 `./bundles/crypto_bitcoin/` 下的一个 bundle，每个 BQ 概念（dataset + 每张表）对应一份 OKF 文档，并用由作为种子的 blockchain-etl 与 Google Cloud 页面 mint 出的 reference 文档进行扩充与交叉链接，外加每个目录层级自动生成的 `index.md`。
