# 开放知识格式 (OKF)

**版本 0.2**

OKF 是一种开放、对人类与智能体都友好的格式，用于表示*知识*：围绕数据与系统存在的元数据、上下文以及经过筛选的洞见。它既可由人编写、由智能体生成，也能跨组织交换，并被两者共同消费。

该格式刻意保持极简：一个由带 YAML frontmatter 的 markdown 文件组成的目录。没有 schema 注册表、没有中心化权威，也没有强制要求的工具链。只要你能 `cat` 一个文件，就能读懂 OKF；只要你能 `git clone` 一个仓库，就能发布它。

本文档是自包含的：它完整规定了生成与消费 OKF v0.2 所需的全部内容。相对 v0.1 的变更摘要见 §13。

---

## 1. 动机

面向 AI 智能体的知识表示领域正在快速演进，许多互不兼容的约定也随之涌现。OKF 秉持这样一种立场：知识最好用常见、通用且成熟的格式来表示，这类格式应当：

- **无需工具即可被人阅读**。
- **无需专用 SDK 即可被智能体解析**。
- **可在版本控制中进行 diff**。
- **可跨工具、跨组织、跨时间移植**。

如今，一个知识语料库往往不是一经编写便只读：它正**持续由智能体写入与维护**。当大多数概念都是机器生成时，消费方需要一些用纯 markdown 加 frontmatter 约定无法作为一等公民表达的答案：

1. 它源自什么，又是如何被验证的？（**来源溯源**）
2. 我应该多大程度上信任它？（**信任**）
3. 它现在仍然为真吗？（**时效性**）
4. 它是当前版本吗？（**生命周期**）
5. 这个数字是否按照我们规定的方式产生？（**认证**）

OKF v0.2 将来源溯源、信任、生命周期与认证提升为一等公民，同时让格式保持极小的主观倾向。该格式的主观倾向极小，它只标准化一小部分让知识语料库能够自我描述所必需的结构约定——除此之外一概交由生产方决定。

### 目标

1. 定义一种通用格式，供**生产方**（人、智能体、导出流水线）写入。
2. 告知**消费方**（智能体、UI、搜索索引、确定性代码）应当如何读取与遍历它。
3. 促进知识在系统与组织之间的**交换**。
4. 标准化一小部分让智能体维护的语料库**可信任**的 frontmatter 字段，但不规定任何运行时。

### 非目标

- 定义一套固定的概念类型分类法。
- 规定存储、服务或查询基础设施。
- 取代领域特定 schema（Avro、Protobuf、OpenAPI 等）。OKF *引用*它们，而非吞并它们。
- 为执行器或认证器所指向的代码规定打包或调用标准。OKF 固定的是接口，而非打包方式。

---

## 2. 术语

- **Knowledge Bundle**（或 **bundle**）：一个自包含的、层次化的知识文档集合。分发的单元。
- **概念（Concept）**：bundle 内一个知识单元，表现为一个 markdown 文档。它可以描述一个有形资产（一张表、一个 API）、一个抽象概念（一个指标、一个业务流程），或两者之间的任何事物。
- **概念 ID（Concept ID）**：概念文件在 bundle 内的路径，去掉 `.md` 后缀。
- **Frontmatter**：位于 markdown 文件顶部、由 `---` 界定的 YAML 元数据块。
- **正文（Body）**：frontmatter 之后文件内的全部内容。
- **链接（Link）**：从一个概念到另一个概念的标准 markdown 链接，用于表达隐含的父子层级之外的关联。
- **来源（Source）**：概念所源自的材料，可在 bundle 内部或外部，记录在 `sources` frontmatter 字段中。
- **来源溯源（Provenance）**：概念所源自的来源集合。
- **可信度信号（Credibility signal）**：客观的、按来源记录的事实（`author`、`usage_count`、`last_modified`），用于推断信任；OKF 记录的是信号，而非结论（见 §5.1）。
- **行为主体（Actor）**：标识谁或什么执行了某个动作的字符串，约定为：智能体使用 `<producer>/<version>`，人使用 `human:<id>`，自动化流程使用 `process:<id>`（见 §7）。
- **信任层级（Trust tier）**：由概念的 `verified` 字段推导出的层级：未验证、机器确认或人工复核（见 §5.3）。
- **认证计算（Attested Computation）**：一种概念（`type: Attested Computation`），携带一种被认可的计算某值的方式，使消费方能够确认该值是通过运行它而产生的（见 §10）。
- **执行器（Executor）**：运行计算并返回回执的执行指令或代码（见 §10.2）。
- **回执（Receipt）**：一次运行返回的、由 `executor.receipt` 塑造的证据；是一种运行时产物，不存储在 bundle 中（见 §10）。
- **认证器（Attester）**：检查回执并返回结论的确定性（无 LLM）代码（见 §10.2）。

---

## 3. Bundle 结构

一个 bundle 是一个 markdown 文件的目录树。目录结构与领域无关：生产方可以按被捕获知识的特点自行组织概念。

```
path/to/bundle/
  index.md                      # 可选。用于渐进式展示的目录清单。
  log.md                        # 可选。按时间排序的更新历史。
  <concept>.md                  # bundle 根目录下的一个概念。
  <subdirectory>/               # 子目录将概念分组组织。
    index.md
    <concept>.md
    <subdirectory>/
      ...
```

一个 bundle 可以以下列任一形式分发：

- 一个 git 仓库（推荐，因为它提供历史、归属与 diff）。
- 该目录的 tarball 或 zip 压缩包。
- 一个更大仓库内的子目录。

### 3.1 保留文件名

下列文件名在层级的任意位置都有定义好的含义，且**不得**用于概念文档：

| 文件名     | 用途                                 |
|------------|--------------------------------------|
| `index.md` | 目录清单。见 §8。                     |
| `log.md`   | 更新历史。见 §9。                     |

其余所有 `.md` 文件均为概念文档。

标签通过 `tags` frontmatter 字段（§4.1）始终保持为一等公民概念。OKF 并未规定一种按标签聚合文档的独立文件格式；想要标签浏览视图的消费方，可以在消费时通过扫描 frontmatter 自行合成一个。

---

## 4. 概念文档

每个概念都是一个 UTF-8 的 markdown 文件，包含两部分：

1. 一个 **YAML frontmatter 块**，由文件开头的独立一行 `---` 与结尾的独立一行 `---` 界定。
2. 一个 **markdown 正文**，包含自由格式的内容。

### 4.1 Frontmatter

```yaml
---
type: <类型名称>                  # 必填
title: <可选的显示名称>
description: <可选的单行摘要>
resource: <底层资产的可选规范 URI>
tags: [<tag>, <tag>, ...]          # 可选
# ... 信任、生命周期、来源溯源与计算相关字段族（见 §5、§10）
# ... 其他由生产方定义的键值对
---
```

**必填：**

- `type`：一个标识概念种类的短字符串。消费方用它来做路由、过滤与展示。取值示例：`BigQuery Table`、`BigQuery Dataset`、`API Endpoint`、`Metric`、`Playbook`、`Reference`、`Attested Computation`。

  类型值**不**在中心统一注册。生产方**应该**选择具有描述性、能自解释的值；消费方**必须**以优雅的方式容忍未知类型，通常是将其当作通用概念处理。

`type` 是唯一始终必填的键；一个仅携带 `type` 的概念完全符合规范（§11）。

**推荐：**

- `title`：人类可读的显示名称。若省略，消费方**可以**从文件名推导出一个标题。
- `description`：一句话概括该概念。被 `index.md` 生成器、搜索摘要与预览所用。
- `resource`：一个 URI，唯一标识该概念所描述的底层资产。对于描述抽象概念而非物理资源的概念，此项缺省。
- `tags`：一组用于横切分类的短字符串 YAML 列表。

可选的**来源溯源**、**信任**与**生命周期**字段族（§5），以及用于认证计算概念的**计算**字段（§10）也可以出现。

**扩展：** 生产方**可以**包含任意额外键。消费方在往返处理时应保留未知键，且**不得**拒绝带有未识别字段的文档。

### 4.2 正文

正文是标准 markdown。生产方**应该**优先使用结构化的 markdown（标题、列表、表格、围栏代码块），而非自由散文，因为结构既有助于人类阅读，也有助于智能体检索。

正文没有强制要求的章节。下列标题具有**约定俗成**的含义，在适用时应使用：

| 标题            | 用途                                                   |
|-----------------|--------------------------------------------------------|
| `# Schema`      | 对资产列/字段的结构化描述。                            |
| `# Examples`    | 具体的使用示例，通常以围栏代码块形式出现。             |
| `# Computation` | 某个认证计算的被认可计算。见 §10。                     |

针对单条声明对外部来源进行归属，使用以 `sources` 条目为键的 markdown 脚注，而非正文的引用列表（§5.1）。

### 4.3 示例：绑定到资源的概念

```markdown
---
type: BigQuery Table
title: Customer Orders
description: One row per completed customer order across all channels.
resource: https://console.cloud.google.com/bigquery?p=acme&d=sales&t=orders
tags: [sales, orders, revenue]
generated: { by: reference_agent/gemini-2.5-pro, at: 2026-05-28T14:30:00Z }
---

# Schema

| Column        | Type      | Description                              |
|---------------|-----------|------------------------------------------|
| `order_id`    | STRING    | Globally unique order identifier.        |
| `customer_id` | STRING    | Foreign key into [customers](/tables/customers.md). |
| `total_usd`   | NUMERIC   | Order total in US dollars.               |
| `placed_at`   | TIMESTAMP | When the customer submitted the order.   |

# Joins

Joined with [customers](/tables/customers.md) on `customer_id`.
```

### 4.4 示例：未绑定到资源的概念

```markdown
---
type: Playbook
title: "Incident response: data freshness alert"
description: Steps to triage a freshness alert on the orders pipeline.
tags: [oncall, incident]
generated: { by: human:ahormati, at: 2026-04-12T09:00:00Z }
---

# Trigger

A freshness alert fires when `orders` lags more than 30 minutes behind its
expected SLA. See the [orders table](/tables/orders.md).

# Steps

1. Check the [ingestion job dashboard](https://example.com/dash).
2. ...
```

---

## 5. 来源溯源、信任与生命周期

这些 frontmatter 字段族让"它从何而来""我应多大程度信任它""它是否仍然是最新的"都能从 frontmatter 中回答。它们都是可选的。它们的缺失本身也携带含义：一个未验证的概念与一个已验证的概念是可区分的，但永远不会被拒绝（§11）。

OKF 中所有取值为时间戳的键都是带有显式 UTC 偏移的 ISO 8601 日期时间，例如 `2026-06-30T14:00:00Z`。

### 5.1 来源溯源：`sources`

`sources` 记录概念所源自的材料，可在 bundle 内部或外部。

```yaml
sources:
  - id: ga4-schema
    resource: https://developers.google.com/analytics/bigquery/export-schema
    title: GA4 BigQuery Export schema
    author: team:ga4-docs
    usage_count: 5000
    last_modified: 2026-05-30T00:00:00Z
usage_window: { from: 2026-06-01T00:00:00Z, to: 2026-06-30T00:00:00Z }
```

每个 `sources` 条目：

- `resource`：条目内**必填**。命名一个消费方可追踪的具体产物（绝对 URL、bundle 相对路径，或 `references/` 子目录内的路径，见 §6），或它无法追踪的某个总体/范围描述符（例如 `all queries in BigQuery project X`）。
- `id`：可选。用于归属单条声明的稳定键（见下文）。当正文引用该来源时，**应该**存在。
- `title`：可选。该来源的人类可读标签。
- 可选的可信度信号 `author`、`usage_count` 与 `last_modified`，见后文说明。

**来源可信度信号。** OKF 记录客观的、按来源记录的信号，使消费方可以通过判断概念被抽取自哪些来源，来推断应多大程度信任该概念。它并不存储一个可信度分数：分数是主观的、无法在消费方之间移植，并且会过时。可信度是从这些信号*推断*出来的，与信任层级的方式相同（§5.3），而非被存储下来的。每个信号都是可选的，且位于某个 `sources` 条目上：

- `author`：是谁或什么产生了该来源，采用行为主体约定（§7）。一个权威信号。
- `usage_count`：`resource` 在 `usage_window` 期间被使用（仪表盘查看、查询执行、页面阅读）的频率。一个采用度与活跃度信号。对于单个产物，它是该产物自身的被使用次数；对于一个范围描述符，它是在该范围内触及此概念的使用次数。
- `last_modified`：来源自身上次变更的时间。一个时效性信号，区别于记录概念被写入时间的 `generated.at`（§5.2）。
- `usage_window`：作为 `sources` 的同级只写一次，它用 `{ from, to }` 日期时间区间框定每一个 `usage_count`。单个条目**可以**携带自己的 `usage_window` 以覆盖共享的那个。

`usage_count` 是一个粗略的信号。它只在"存活 vs. 消亡"和数量级层面，以及对照某来源自身的历史时具有可比性，而不能作为精确的跨种类排名：一个定时查询的执行次数与一个人工刻意的仪表盘查看次数并不等重。消费方**应该**把它当作活跃度与趋势来读，而非一个分数。

血缘通过链接表达，而非某个专用字段。当某个 `resource` 指向另一个 OKF 概念时，派生边已经存在于 bundle 图中（§6），因此消费方**可以**递归进入该来源自身的 `sources`，并让可信度传播。外部的叶子来源仅携带其内在信号。更深层的血缘（显式的外部 `derived_from`，或数据血缘）不在 v0.2 的范围内。

**按声明归属。** 要归属某条具体声明，可使用一个 markdown 脚注，其标签为某个 `sources[].id`：

```markdown
The `events_` table is sharded daily as `events_YYYYMMDD`.[^ga4-schema]

[^ga4-schema]: GA4 BigQuery Export schema
```

脚注标签是进入 `sources` 的 join 键；消费方通过匹配的条目来解析归属，而非解析脚注散文。标签采用键值而非位置（`sources[0]`），是因为智能体会不断重写这些文档：位置索引一旦列表被重排便会静默地错归，而稳定的 `id` 在重排后依然有效。

### 5.2 信任：`generated` 与 `verified`

`generated` 记录当前内容是如何产生的。`verified` 记录谁或什么已针对其来源或 `resource` 确认了内容。二者被刻意区分，因为*编写*某个概念的人不必是*确认*它的人。

```yaml
generated: { by: reference_agent/gemini-2.5-pro, at: 2026-06-20T22:53:05Z }
```

- `generated.by`：`generated` 内**必填**。一个行为主体（§7）。
- `generated.at`：一个 ISO 8601 日期时间，标记内容最后一次有意义的变更。消费方用它来区分一次最近的编辑与一个过时的旧事实。

```yaml
verified:
  - { by: human:ahormati, at: 2026-06-25T09:00:00Z }
  - { by: process:finance-nightly, at: 2026-06-26T02:00:00Z }
```

- `verified`：一组验证事件，每个带有 `by`（一个行为主体）与 `at`（一个 ISO 8601 日期时间）。多个条目捕获相互独立的检查，例如一次人工签名加上一个夜间流程。"有多近"以最新的那个 `at` 为准。
- `verified` 独立于 `generated.at`：内容可以变更而无需重新确认，事实也可以重新确认而无需重新生成。
- 单个验证者**可以**写成一个不带列表破折号的 `{ by, at }` 映射。消费方**必须**将一个裸映射当作单元素列表处理：

```yaml
verified: { by: human:ahormati, at: 2026-06-25T09:00:00Z }
```

### 5.3 信任层级

消费方从 `verified` 推导出一个信任层级，由低到高：

- 无 `verified` 键 ⇒ **未验证**。
- 仅由非 `human:` 的行为主体 `verified` ⇒ **机器确认**。
- 由 `human:<id>` 行为主体 `verified` ⇒ **人工复核**。

一个没有信任 frontmatter 的概念仍然是可消费的；消费方**不得**拒绝它（§11）。信任层级是建议性信号，而非访问控制。

### 5.4 生命周期：`status`

```yaml
status: stable        # draft | stable | deprecated
```

- `draft`：尚未评审；可能不完整。
- `stable`：默认值；可供消费。
- `deprecated`：为链接与历史而保留；不再是当前版本。

缺省 `status` ⇒ `stable`。

### 5.5 生命周期：`stale_after`

```yaml
stale_after: 2026-09-23T00:00:00Z   # 内容在该时刻或之后即视为过期
```

可选。一个绝对时刻。当 `now >= stale_after` 时，概念即已过期。采用绝对时刻而非相对 TTL，是为了让过期判定只是一个单纯的比较，无需参考概念被读取的时间。

---

## 6. 交叉链接与路径

### 6.1 概念间的链接

概念**可以**使用标准 markdown 链接链接到其他概念。支持两种形式：

- **绝对（bundle 相对）：** 以 `/` 开头，相对于 bundle 根目录解释。这是**推荐**形式，因为当文档在其子目录内移动时它保持稳定。

  ```markdown
  See the [customers table](/tables/customers.md) for the join key.
  ```

- **相对：** 一个标准 markdown 相对路径。

  ```markdown
  See the [neighboring concept](./other.md).
  ```

从概念 A 到概念 B 的一个链接断言了一种*关系*。其具体种类（父子、引用、join-with、依赖）由周围的散文传达，而非由链接本身传达。构建图视图的消费方通常将所有链接视为无类型关系的定向边。

消费方**必须**容忍断链：一个目标在 bundle 中不存在的链接并不算格式错误；它也许只是代表尚未编写的知识。

### 6.2 取值为路径的字段

若干字段命名了一个路径或 URI：`resource`、`sources[].resource`、`computation`、`executor.resource` 与 `attester.resource`（§10）。`sources[].resource` 也可以是一个范围描述符（§5.1），此时它并非路径。每个取值为路径的字段接受：

- 一个绝对 URL（例如 `https://...`），
- 一个以 `/` 开头的 bundle 相对路径，或
- 一个相对路径（例如 `../computations/revenue.md`）。

### 6.3 `references/` 约定

`references/` 子目录按约定将外部材料、运行指令或代码镜像为 bundle 内的头等概念。来源、执行器与认证器通常会指向它（例如 `references/attesters/revenue.py`）。这是一种命名约定，而非强制要求。

---

## 7. 行为主体约定

记录身份的字段（`generated.by`、`verified[].by`）采用单一的行为主体约定：

- 智能体与工具使用 `<producer>/<version>`，例如 `reference_agent/gemini-2.5-pro`。
- 人使用 `human:<id>`，例如 `human:ahormati`。
- 自动化流程使用 `process:<id>`，例如 `process:finance-nightly`。

对信任进行分类的消费方（§5.3）以 `human:` 前缀为依据，因此生产方**必须**将它用于手工编写或人工确认的内容。

---

## 8. 索引文件

`index.md` 文件**可以**出现在任意目录中，包括 bundle 根目录。它枚举该目录的内容，以支持**渐进式展示**：让人类或智能体在打开单个文档之前先看到有哪些内容可用。

索引文件不含 frontmatter，唯一的例外是：bundle 根的 `index.md` **可以**携带一个 `okf_version` 键（§12）。正文使用一个或多个分区，每个分区在标题下对概念进行分组：

```markdown
# Section / Group Heading

* [Title 1](relative-url-1) - short description of item 1
* [Title 2](relative-url-2) - short description of item 2

# Another Section

* [Subdirectory](subdir/) - short description of the subdirectory
```

条目**应该**包含所链接概念 frontmatter 中的描述。生产方**可以**自动生成 `index.md`；消费方在缺失时**可以**即时合成一个。

---

## 9. 日志文件

`log.md` 文件**可以**出现在层级的任意位置，以记录该范围内变更的履历。格式为按日期分组的扁平条目列表，最新的在前：

```markdown
# Directory Update Log

## 2026-05-22
* **Update**: Added a BigQuery table reference for [Customer Metrics](/tables/customer-metrics.md).
* **Creation**: Established the [Dataplex Playbook](/playbooks/dataplex.md).

## 2026-05-15
* **Initialization**: Created foundational directory structure.
```

日期标题**必须**使用 ISO 8601 的 `YYYY-MM-DD` 形式。日志条目是散文；开头加粗的词（`**Update**`、`**Creation**`、`**Deprecation**`）是约定，而非强制要求。

---

## 10. 认证计算概念

一个认证计算概念不仅携带某个值*意味着什么*，还携带一种被认可的*计算*它的方式，使消费方能确认智能体运行的是被认可的计算，而非自行即兴发挥。来源溯源（§5.1）回答"这条声明从何而来"；认证回答"这个数字是否按照我们规定的方式产生"。OKF 记录计算以及检查它的手段；它本身不执行任何东西。

### 10.1 一个计算本身就是一个概念

一个被认可的计算是一个独立的 `type: Attested Computation` 概念。需要该值的概念（一个 `Metric`、一个 `BigQuery Table`）用一条普通 markdown 链接（§6）指向它。有三个特性促使它成为独立概念：

- **`runtime` 定义了 `parameters` 的含义。** 一个参数究竟是 SQL 绑定变量、dbt 变量还是 Python 参数，取决于 runtime。将 `runtime` 与 `parameters` 放在同一个 frontmatter 中，使绑定语义不言自明。
- **一次计算，多个消费方。** 同一个计算可以支撑一个指标、一个仪表盘概念以及一份报告；作为一个概念，它被引用一次并可复用。
- **信任状态按计算独立。** `verified`、`stale_after` 与单个 `attester` 描述的是同一件事物。收入、利润与毛利率各自独立验证与认证，这是三个概念，而非一个 frontmatter 中的三个条目。

### 10.2 契约字段

契约就是该概念顶层的 frontmatter。除来源溯源、信任与生命周期字段族（§5）之外，一个认证计算概念还携带：

- `runtime`：该类型**必填**。唯一说明如何运行计算，从而说明执行器与认证器如何解释它、以及 `parameters` 含义的字段。取值示例：`bigquery`、`postgres`、`dbt`、`python`、`Looker`。
- `parameters`：一组智能体可以填充的类型化、具名"空位"。每个条目为 `{ name, type, required }`。绑定语义遵循 `runtime`。
- `computation`：可选。一个指向持有计算内容的文件（§6.2）的路径，用于取代正文内的围栏（见 §10.3）。缺省 ⇒ 正文的 `# Computation` 围栏即为计算。
- `executor`：计算如何运行。`resource` 命名运行指令或代码；一个运行器（智能体，或确定性消费方代码）遵循它。`receipt` 声明一次运行必须返回的字段，即认证器所检查的证据（例如一个 BigQuery `job_id` 与该作业实际执行的 SQL）。
- `attester`：确定性检查。`resource` 命名代码（无 LLM），它接收一个回执并返回一个结论。它意在消费方一侧运行。

`resource` 背后是什么（一个 Skill、一段脚本、一个容器）是一个打包选择；OKF 固定的是接口，而非打包方式（§1）。

```markdown
---
type: Attested Computation
title: Revenue for fiscal year
description: Recognized revenue for a fiscal year, per Finance's definition.
status: stable
runtime: bigquery
parameters:
  - { name: year, type: integer, required: true }
executor:
  resource: references/skills/run-on-bq.md
  receipt: [job_id, executed_sql, result]
attester:
  resource: references/attesters/revenue.py
generated: { by: reference_agent/gemini-2.5-pro, at: 2026-06-20T22:53:05Z }
verified: { by: human:ahormati, at: 2026-06-25T09:00:00Z }
stale_after: 2026-09-23T00:00:00Z
sources:
  - id: rev-policy
    resource: https://wiki.acme/finance/revenue-recognition
    title: Revenue recognition policy
---

# Computation

    SELECT SUM(amount) AS revenue
    FROM finance.recognized_revenue
    WHERE fiscal_year = @year

The computation binds only the declared `parameters`, per the recognition
policy.[^rev-policy]

[^rev-policy]: Revenue recognition policy
```

### 10.3 计算本身

通过以下两种方式之一提供计算：

- **内联：** 正文 `# Computation` 下的单个围栏代码块。最适合与契约一同审阅的简短计算。
- **文件：** 将 `computation` 设为一个路径（§6.2）并省略正文围栏。最适合较长或生成式的计算，或已经作为真实文件与非 OKF 工具共享的计算。

```yaml
runtime: bigquery
computation: references/computations/lib/revenue.sql
parameters:
  - { name: year, type: integer, required: true }
```

智能体**只能**为声明的 `parameters` 提供*值*；它**不得**编写或编辑计算。将计算与参数值绑定到可执行产物是消费方的职责，而认证器会独立地重新推导出同一个绑定，以与真正运行过的内容做比较。由于比较针对的是回执所携带的展开后、已编译的产物（`executed_sql`、`compiled_sql`），任何被重写的查询、被替换的计算文件或被篡改的依赖都会使检查失败。一个类型化、仅含参数的接口，正是让"被认可的那件事是否真的运行了"成为一个机械化比较、而非主观判断的原因。

### 10.4 使用某个计算的概念

一份文档很少只包含一个计算。一份讨论收入、利润与毛利率的利润表概述仍是一个可读的概念，并为每个数字链接到一个认证计算：

```markdown
---
type: Metric
title: Revenue
description: Recognized revenue for a fiscal year.
tags: [finance, revenue]
status: stable
generated: { by: reference_agent/gemini-2.5-pro, at: 2026-06-20T22:53:05Z }
---

# Definition

Recognized revenue sums `amount` over rows booked to the fiscal year,
computed by [the revenue computation](../computations/revenue.md).
```

因为每个计算本身就是独立的概念，收入可以仍然新鲜，而利润已经过了它的 `stale_after`，且各自在自己的运行中认证。将它们放在一起是一个目录选择（一个带 `index.md` 的 `computations/` 文件夹），而非 frontmatter 的选择。

### 10.5 消费方如何使用它（资料性）

本小节为资料性内容，非规范性。下列运行时产物**不**存储在 bundle 中。

1. **发现（Discover）**：通过 `type: Attested Computation`，一个可提升进 `index.md` 的 frontmatter 信号；消费方可以直接到达一个，也可以顺着使用它的概念所发出的链接找到它。
2. **加载（Load）**：从 frontmatter 加载契约，从正文（或 `computation` 命名的文件）加载计算。
3. **参数化（Parameterize）**：智能体为声明的参数提供值。
4. **执行（Execute）**：执行器运行绑定后的计算，并返回一个由 `executor.receipt` 塑造的回执。
5. **认证（Attest）**：消费方针对回执运行认证器。它确认来源溯源（运行过的计算等于用所声称参数绑定后的 `computation`，而非智能体编写的 SQL）与保真度（显示的值匹配回执中权威来源的值，通过 job id 重新读取，而非取自智能体的文本）。
6. **把关（Gate）**：拒绝展示一次失败的认证；当 `now >= stale_after` 时给出警告或拒绝。成功时，呈现该结论（例如一个指向作业日志的链接），使信任可见。

### 10.6 验证与认证的区别

`verified`（§5.2）与认证是不同的两件事，且二者并存：

- `verified` 确认*定义*仍然符合策略。它是文档级的、缓慢的，并记录在 bundle 中。
- 认证确认单次*运行*以被认可的方式产出了该值。它是每次调用级的、运行时的，且不存储在 bundle 中。

一个定义已过时的概念仍然可以干净地通过认证，而一个刚刚验证过的定义在每次运行时仍需要认证，这正是二者都需要的原因。

---

## 11. 一致性

一个 bundle 若要**符合** OKF v0.2，需满足：

1. 树中每个非保留的 `.md` 文件都包含一个可解析的 YAML frontmatter 块。
2. 每个 frontmatter 块都包含一个非空的 `type` 字段。
3. 每个保留文件名（`index.md`、`log.md`）在出现时，分别遵循 §8 与 §9 中的结构。

当信任、生命周期、来源溯源或计算字段族出现时，生产方**应该**遵循 §5 至 §10，且消费方：

- **必须**将一个裸 `verified` 映射当作单元素列表处理（§5.2）。
- **不得**因为缺失任何可选字段族而拒绝一个概念（§5.3）。
- **应该**仅从本文规定的字段推导信任层级与过期状态，且**应该**暴露（而非静默丢弃）一次失败的认证（§10.5）。

消费方**应该**将所有其他约束视为柔性指导。具体而言，消费方**不得**因以下原因拒绝一个 bundle：

- 缺失可选的 frontmatter 字段。
- 未知的 `type` 值。
- 未知的额外 frontmatter 键。
- 断开的交叉链接。
- 缺失的 `index.md` 文件。

---

## 12. 版本管理

本文档规定 OKF **0.2** 版本。修订按 `<major>.<minor>` 版本化：

- **次版本（minor）**号递增引入向后兼容的增补（新的可选字段、新的约定章节标题）。
- **主版本（major）**号递增可能带来破坏性变更（重命名必填字段、更改保留文件名）。

bundle **可以**通过在一个 bundle 根的 `index.md` frontmatter 块中声明 `okf_version: "0.2"` 来标明它所面向的版本（这是 `index.md` 中唯一允许 frontmatter 的地方）。不理解所声明版本的消费方**应该**尝试尽力消费，而非拒绝该 bundle。

### 已考虑但推迟的事项

下列内容被有意留给未来的修订：

- 完整的运行时协议：回执与结论的线格式，以及围绕一次运行的认证生命周期。
- 认证器的 ABI、可移植性与沙箱化，可能与此后关于服务与 Skills 的工作一并处理。
- 认证缓存。
- 语义层模板（Looker、dbt），其中认证器的比较从 SQL 相等转向模型与绑定相等。

---

## 13. 相对 v0.1 的变更

v0.2 取代 OKF v0.1，并且按 §12 属于一次次版本递增，但有两处刻意的破坏性变更，如下文所指出，因为它们重命名或废弃了 v0.1 的字段。在本文注明的回退机制下，v0.1 的 bundle 可被 v0.2 的消费方消费。

### 13.1 破坏性变更

- **`timestamp` 被 `generated.at` 取代。** 概念内容的最后一次变更现在记录为 `generated: { by, at }`（§5.2）。当 `generated` 缺省时，消费方**可以**回退到遗留的 `timestamp`。
- **正文的 `# Citations` 列表被 `sources` 取代。** 来源溯源移至 frontmatter（§5.1）。消费方**应该**读取 `sources`，并且**可以**仍解析 v0.1 文档中遗留的 `# Citations` 正文列表。

### 13.2 增量变更

下列全部为增量：新的可选键、一种新的概念类型，以及一个新的约定标题。它们的缺失会得到一个纯粹的 v0.1 概念。

- 新的 frontmatter 字段族：带其按来源可信度信号的 `sources`（`author`、`usage_count`、`last_modified`）以及 `usage_window` 同级键；`generated`、`verified`；`status`、`stale_after`（§5）。
- 新的概念类型 `Attested Computation` 及其计算键 `runtime`、`parameters`、`computation`、`executor`、`attester`（§10）。
- 新的约定正文标题 `# Computation`（§4.2）。
- 用于 `generated.by` 与 `verified[].by` 的行为主体约定（§7）。

其余一切（bundle 结构、保留文件名、必填的 `type`、推荐的 `title`/`description`/`resource`/`tags`、交叉链接、索引文件、日志文件、宽松的一致性）均原样沿用。

---

## 附录 A：完整示例——一份利润表

一个用到了每个字段族的 bundle，呈现为一份带两个数字（收入与毛利）的利润表从 v0.1 到 v0.2 的迁移。

### v0.1 形式

一份单一文档：两个数字都在一个概念中，SQL 在散文里、可供智能体阅读、忽略或重写，引用是一个扁平列表，且唯一的时间戳只有 `timestamp`。

```markdown
---
type: Metric
title: Income statement (fiscal year)
description: Headline income-statement figures for a fiscal year.
tags: [finance, income-statement]
timestamp: '2026-05-28T22:53:05+00:00'
---

# Definition
The income statement reports revenue and gross profit for a fiscal year.

# Revenue
Recognized revenue sums `amount` over rows booked to the fiscal year:

    SELECT SUM(amount) AS revenue
    FROM finance.recognized_revenue
    WHERE fiscal_year = <year>

# Gross profit
Gross profit by segment, per the cost-allocation standard:

    SELECT gross_profit FROM fct_income_statement
    WHERE fiscal_year = <year> AND segment = <segment>

# Citations
- https://wiki.acme/finance/fpa-handbook
- https://wiki.acme/finance/revenue-recognition
- https://wiki.acme/finance/cost-allocation
```

### v0.2 形式

两个数字被拆分为从一份叙述性概念链接出去的认证计算。每个字段族都被填充，且两个计算被刻意置于不同的状态，使得一个消费方得出两个结论。

```
bundles/finance/
  metrics/income-statement.md      type: Metric  (narrates, links both)
  computations/revenue.md          type: Attested Computation  (runtime: bigquery)
  computations/profit.md           type: Attested Computation  (runtime: dbt)
  references/skills/run-on-bq.md, run-dbt.md
  references/attesters/sql-equality.py, dbt-binding.py
```

`metrics/income-statement.md`，那份可读文档；信任存在于它所链接的对象上，而非此处：

```markdown
---
type: Metric
title: Income statement (fiscal year)
description: Headline income-statement figures for a fiscal year.
tags: [finance, income-statement]
status: stable
generated: { by: reference_agent/gemini-2.5-pro, at: 2026-06-20T22:53:05Z }
verified: { by: human:ahormati, at: 2026-06-25T09:00:00Z }
stale_after: 2026-12-31T00:00:00Z
sources:
  - id: fpa-handbook
    resource: https://wiki.acme/finance/fpa-handbook
    title: FP&A reporting handbook
---

# Definition
The income statement reports [revenue](../computations/revenue.md) and
[gross profit](../computations/profit.md) for a fiscal year, per the FP&A
reporting handbook.[^fpa-handbook] Each figure is produced by a sanctioned,
attestable computation; this concept only narrates them.

[^fpa-handbook]: FP&A reporting handbook
```

`computations/revenue.md`，BigQuery SQL，经人工验证，新鲜，并有携带可信度信号的实时仪表盘来源佐证：

```markdown
---
type: Attested Computation
title: Revenue for fiscal year
description: Recognized revenue for a fiscal year, per Finance's definition.
tags: [finance, revenue]
status: stable
runtime: bigquery
parameters:
  - { name: year, type: integer, required: true }
executor:
  resource: references/skills/run-on-bq.md
  receipt: [job_id, executed_sql, result]
attester:
  resource: references/attesters/sql-equality.py
generated: { by: reference_agent/gemini-2.5-pro, at: 2026-06-28T14:00:00Z }
verified: { by: human:ahormati, at: 2026-06-25T09:00:00Z }
stale_after: 2026-12-31T00:00:00Z
sources:
  - id: rev-policy
    resource: https://wiki.acme/finance/revenue-recognition
    title: Revenue recognition policy
    author: team:finance-fpa
    last_modified: 2026-04-02T00:00:00Z
  - id: exec-rev-dash
    resource: dashboards/exec-revenue
    title: Executive revenue dashboard
    author: team:finance-fpa
    usage_count: 5000
    last_modified: 2026-06-18T00:00:00Z
usage_window: { from: 2026-06-01T00:00:00Z, to: 2026-06-30T00:00:00Z }
---

# Computation

    SELECT SUM(amount) AS revenue
    FROM finance.recognized_revenue
    WHERE fiscal_year = @year

Recognized revenue per the recognition policy,[^rev-policy] corroborated by
the executive revenue dashboard.[^exec-rev-dash]

[^rev-policy]: Revenue recognition policy
[^exec-rev-dash]: Executive revenue dashboard
```

`computations/profit.md`，一个 dbt 模型，经流程验证，且已过了它的 `stale_after`：

```markdown
---
type: Attested Computation
title: Gross profit for fiscal year
description: Gross profit by segment for a fiscal year, per the cost-allocation standard.
tags: [finance, profit]
status: stable
runtime: dbt
parameters:
  - { name: year, type: integer, required: true }
  - { name: segment, type: string, required: true }
executor:
  resource: references/skills/run-dbt.md
  receipt: [run_id, compiled_sql, result]
attester:
  resource: references/attesters/dbt-binding.py
generated: { by: reference_agent/gemini-2.5-pro, at: 2026-06-14T14:00:00Z }
verified: { by: process:finance-nightly, at: 2026-06-12T08:00:00Z }
stale_after: 2026-06-15T00:00:00Z
sources:
  - id: cost-alloc
    resource: https://wiki.acme/finance/cost-allocation
    title: Cost allocation standard
---

# Computation

    SELECT gross_profit
    FROM {{ ref('fct_income_statement') }}
    WHERE fiscal_year = {{ var('year') }}
      AND segment = {{ var('segment') }}

Gross profit by segment per the cost-allocation standard.[^cost-alloc]

[^cost-alloc]: Cost allocation standard
```
