# Stack Overflow 公共数据集样例

对公共数据集 `bigquery-public-data.stackoverflow`（Stack Overflow 的 Stack Exchange Data Dump 镜像 —— `posts_questions`、`posts_answers`、`users`、`votes`、`comments`、`badges`、`tags`、`post_history`、`post_links` 等）运行 reference agent，并以 Stack Exchange 社区维护的规范 schema 参考为 web pass 提供种子。

本样例与 GA4 样例形成对比，它演练的是 **多概念 enrich**：单个 schema-docs 页面通常描述多张表（`posts_questions` + `posts_answers` + `users`），因此 web agent 常常每抓取一个页面就更新不止一个概念。

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
  公共数据集可读，但查询字节数会记在调用方项目账上。`stackoverflow` 表很大 —— 迭代时请将 `--web-max-pages` 保持在较小值。
- Gemini 凭据 —— 二选一：`GEMINI_API_KEY`（AI Studio）**或** Vertex AI（`GOOGLE_GENAI_USE_VERTEXAI=true`、`GOOGLE_CLOUD_PROJECT=<id>`、`GOOGLE_CLOUD_LOCATION=<region>`）。

## 运行

```
.venv/bin/python -m reference_agent enrich \
    --source bq \
    --dataset bigquery-public-data.stackoverflow \
    --web-seed-file samples/stackoverflow/seeds.txt \
    --out ./bundles/stackoverflow
```

要对单个概念迭代，添加 `--concept tables/posts_questions`。要跳过 web pass，添加 `--no-web`。要调高或调低 web 预算，使用 `--web-max-pages N`（默认 100）。

## 你会得到什么

位于 `./bundles/stackoverflow/` 下的一个 bundle，每个 BQ 概念（dataset + 每张表）对应一份 OKF 文档，并用由作为种子的 Stack Exchange schema 页面 mint 出的 reference 文档进行扩充与交叉链接，外加每个目录层级自动生成的 `index.md`。
