# GA4 Google Merchandise Store 样例

对公共数据集 `bigquery-public-data.ga4_obfuscated_sample_ecommerce`（来自 Google Merchandise Store 的一份 GA4 导出）运行 reference agent，并以规范的 GA4 BigQuery Export 文档 URL 为 web pass 提供种子。

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
  公共数据集可读，但查询字节数会记在调用方项目账上。
- Gemini 凭据 —— 二选一：`GEMINI_API_KEY`（AI Studio）**或** Vertex AI（`GOOGLE_GENAI_USE_VERTEXAI=true`、`GOOGLE_CLOUD_PROJECT=<id>`、`GOOGLE_CLOUD_LOCATION=<region>`）。

## 运行

```
.venv/bin/python -m reference_agent enrich \
    --source bq \
    --dataset bigquery-public-data.ga4_obfuscated_sample_ecommerce \
    --web-seed-file samples/ga4_merch_store/seeds.txt \
    --out ./bundles/ga4
```

要对单个概念迭代，添加 `--concept tables/events_`。要跳过 web pass，添加 `--no-web`。要调高或调低 web 预算，使用 `--web-max-pages N`（默认 100）。

## 你会得到什么

位于 `./bundles/ga4/` 下的一个 bundle，每个 BQ 概念（dataset + 各张表）对应一份 OKF 文档，可选地用由作为种子的 GA4 文档页面 mint 出的 reference 文档进行扩充与交叉链接，外加每个目录层级自动生成的 `index.md`。
