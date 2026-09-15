---
type: Skill
title: 在 BigQuery 上运行 Attested Computation
description: "面向 `runtime: bigquery` 的 `Attested Computation` 概念的执行器技能。绑定已声明参数、提交作业，并返回校验器将验证的回执。"
tags: [skill, executor, bigquery]
generated: { by: human:kliu@acme, at: 2026-06-30T14:00:00Z }
status: stable
---

# 技能：在 BigQuery 上运行

## 何时使用（When to use）

当某个 `Attested Computation` 的 `runtime` 为 `bigquery` 时，其 `executor.resource` 字段指向本技能。

## 前置条件（Preconditions）

- 调用方在概念所属 `resource` 项目上拥有带 `bigquery.jobs.create` 权限的服务账号。
- 调用方对计算逻辑的 `# Computation`（计算逻辑）代码块（或 `computation:` 指向的文件）所引用的每个表拥有读取权限。
- 概念已声明其 `parameters:` 列表。每个必需参数都已由调用方提供取值。

## 步骤（Steps）

1. **加载计算逻辑（Load the computation）。** 从概念正文读取 `# Computation`（计算逻辑）代码块，若设置了该字段则读取 `computation:` 指向的文件。结果是一个 SQL 字符串，其中包含每个已声明参数的 `@name` 形式绑定变量。
2. **绑定参数（Bind parameters）。** 将调用方提供的值作为 BigQuery [named query parameters](https://cloud.google.com/bigquery/docs/parameterized-queries) 传入。Do NOT 做字符串插值；校验器会拒绝 `executed_sql` 中显示字面量替换的回执。
3. **提交作业（Submit the job）。** 使用 `jobs.query`（或带 `Query` 配置的 `jobs.insert`）。设置 `useLegacySql: false`。设置作业的 `labels` 以包含 `okf_concept: <bundle-relative-path>`，便于审计。
4. **等待完成（Wait for completion）。** 轮询 `jobs.get` 直到 `status.state = 'DONE'`。若 `status.errorResult` 存在，则返回 `result: null` 且 `error: <errorResult>` 的回执，以便校验器区分“运行时失败的授权 SQL”与“执行器运行了错误的 SQL”。
5. **组装回执（Assemble the receipt）。** 严格返回 `executor.receipt` 中声明的字段。对本技能而言：

    ```json
    {
      "job_id": "bq://<project>/us/<jobId>",
      "executed_sql": "<queryConfig.query with parameters shown as @name>",
      "result": "<the first result row's cell values, in declared select-order>"
    }
    ```

6. **绝不改写计算逻辑（Never modify the computation）。** 若调用方提供的参数无法绑定（缺少必需项、类型错误），应拒绝并返回错误回执。Do NOT 为绕开缺失参数而改写 SQL。

## 后置条件（Post-conditions）

回执交由概念的 `attester.resource` 处理。在校验器返回 `verdict: ok` 之前，不得向用户展示该数值。
