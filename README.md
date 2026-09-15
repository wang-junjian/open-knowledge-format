# 开放知识格式（OKF）

### 📖 [阅读 Open Knowledge Format v0.2 规范 → SPEC.md](SPEC.md)

> **本仓库主要围绕 [Open Knowledge Format (OKF)](SPEC.md) 展开。**
>
> OKF 是一种**通用、厂商中立的格式**，用带 YAML frontmatter 的纯
> markdown 文件来表示知识。它**不绑定任何特定的 agent、框架、
> 模型提供方或服务系统**。目标很简单：
>
> - **任何人都能生成** OKF —— 既可以手工编写，也可以基于任意框架
>   （Google ADK、LangChain、自建框架）构建 agent，或从现有目录
>   （Dataplex、Unity Catalog、Collibra 等）导出，又或写脚本遍历
>   数据库。
> - **任何人都能服务和消费** OKF —— 静态文件服务器、知识管理 UI
>   （Obsidian、Notion、MkDocs）、把文件载入上下文的 LLM、
>   搜索索引，或是本仓库内置的图谱查看器。
>
> 下面的 agent 是一个**概念验证**，演示了自动生成 OKF bundle 的
> *一种*方式。格式本身才是贡献所在；这个 agent 与可视化查看器
> 是为了让格式在「生产」与「消费」两端都变得可感可触。
>
> **看看 OKF 的实际应用** —— 由本 agent 生成的三个可直接浏览的
> bundle，已提交至 [`bundles/`](bundles/)：
>
> - [`bundles/ga4/`](bundles/ga4/) —— GA4 电商数据集
>   （[viz.html](bundles/ga4/viz.html)）
> - [`bundles/stackoverflow/`](bundles/stackoverflow/) —— Stack Overflow
>   公开数据集（[viz.html](bundles/stackoverflow/viz.html)）
> - [`bundles/crypto_bitcoin/`](bundles/crypto_bitcoin/) —— Bitcoin
>   区块/交易（[viz.html](bundles/crypto_bitcoin/viz.html)）
> - [`bundles/acme_retail/`](bundles/acme_retail/) —— Acme Retail
>   （[viz.html](bundles/acme_retail/viz.html)）

## 为什么选择 OKF？

OKF 用带 YAML frontmatter 的纯 markdown 文件表示目录知识，
并按目录层级组织。
这种做法释放了几个在「服务自有的
元数据存储」中难以获得的特征：

- **人和 agent 都可阅读。** 读者与内容之间不需要任何 SDK 或
  查询语言。工程师可以直接 `cat` 一个概念；
  LLM 也能逐字
  把它读入上下文。
- **开箱即用的版本控制。** bundle 就存放在 git 里。Pull request、
  逐行 diff、blame 与代码评审流程都能直接用 ——
  知识整理
  变成了普通的软件工程活动。
- **可移植、无锁定。** 一个 bundle 就是一个目录。可以打包成
  tarball 分发、托管到任意仓库、挂载到任意文件系统，
  或同步到
  任何支持文件的系统。你的元数据与任何专有 API 之间
  都没有隔阂。
- **结构化与非结构化数据有意混合。** 用 frontmatter 存放你想要
  查询、过滤或索引的少数字段（`type`、`resource`、`tags`、
  `generated`、`status`）；用 markdown 正文承载 LLM 与人类真正会
  去读的散文式内容、schema 与
  示例查询。
- **可信度、来源与时效是一等公民。** v0.2 把可查询信号放进
  frontmatter —— 概念来自哪里（`sources`，含各来源的信誉信号）、
  由谁生成并确认（`generated`、`verified`，
  消费者据此推导信任
  等级）、以及它是否仍然最新（`status`、`stale_after`）——
  于是 agent 维护的语料库无需任何定制运行时也能保持可信。
- **最小化约束、可自由扩展。** 一小部分必需键保证了互操作性，
  但 bundle 可以携带任意额外的 frontmatter 键与任意正文段落，
  而不会破坏消费者。
- **可与现有工具组合。** 许多知识工具 —— Notion、Obsidian、
  MkDocs、Hugo、Jekyll —— 原本就支持 markdown 加 YAML
  frontmatter，因此 bundle 无需定制 UI 即可
  浏览、编辑或渲染。
- **内置渐进式披露。** 自动生成的 `index.md` 文件让 agent 或人类
  可以一次浏览一层层级，
  而不必把整个 bundle 载入上下文。
- **呈图谱结构，而非仅仅是树状。** 概念之间通过普通 markdown 链接
  相互关联，
  表达出比目录结构所隐含的父子关系更丰富的关系。

最终效果是：参考 agent、消费 agent 与人类在同一个工件上协作，
方式与他们已经在源代码上的协作
别无二致。

## 安装

```
python3.13 -m venv .venv
.venv/bin/pip install --index-url https://pypi.org/simple/ -e .[dev]
```

## 凭证

- BigQuery：执行 `gcloud auth application-default login` 并
  准备一个
  用于计费的项目（`gcloud config set project <id>`）。
  公开数据集可读，
  但查询字节数会记到调用方项目上。
- Gemini：设置 `GEMINI_API_KEY`（AI Studio）**或**通过设置
  `GOOGLE_GENAI_USE_VERTEXAI=true`、`GOOGLE_CLOUD_PROJECT=<id>` 与
  `GOOGLE_CLOUD_LOCATION=<region>` 来使用 Vertex AI。

## 参考 agent 的工作原理

参考 agent 分两轮运行。
**BQ 轮**仅利用 BigQuery 元数据，为源所声明的
每个概念各写一份 OKF 文档。**web 轮**则把 LLM 当作自己的爬虫来运行：
它接收一组种子 URL（通过 `--web-seed` 或 `--web-seed-file` 提供），
用 `fetch_url` 工具抓取这些种子，并依据这些外链是否看起来像是既有
概念的权威文档来决定是否值得跟进。
对于抓取到的每个页面，agent 会选择：
(a) 丰富一个或多个既有概念文档，(b) 新生成一个独立的
`references/<slug>` 文档，或 (c) 跳过。工具内部强制实施一个
硬性的
`--web-max-pages` 上限与同域允许的 host 过滤器（可通过
`--web-allowed-host` 配置），因此 agent 不会越界。使用 `--no-web`
可跳过 web 轮。

## 运行

最小调用方式 —— 指向一个 BigQuery 数据集与
一个 bundle 输出
目录。web 轮的种子需显式给出；省略它们（或传入
`--no-web`）即可只跑 BQ 轮：

```
.venv/bin/python -m reference_agent enrich \
    --source bq \
    --dataset <project>.<dataset> \
    --web-seed-file <path/to/seeds.txt> \
    --out ./bundles/<name>
```

通过加上 `--concept <type>/<name>`（例如 `--concept tables/events_`）
来针对单个概念迭代；可重复使用。

## 样例

每个样例把一个**配方**（位于 `samples/<name>/`，内含种子
URL 与
精确的 `enrich` 命令）与配方所生成的**产物 bundle**（位于
`bundles/<name>/`）配对。打开配方可复现结果；打开 bundle 可直接浏览结果。

- **GA4 Google Merchandise Store** —— 公开电商数据集，以权威的
  GA4 BigQuery Export 文档 URL 作为种子。
  · [配方](samples/ga4_merch_store/README.md)
  · [bundle](bundles/ga4/)
  · [viz.html](bundles/ga4/viz.html)
- **Stack Overflow** —— 公开数据集（Stack Exchange Data Dump 的
  镜像），以社区权威的 schema 参考作为种子。演示了跨页文档
  带来的多概念丰富。
  · [配方](samples/stackoverflow/README.md)
  · [bundle](bundles/stackoverflow/)
  · [viz.html](bundles/stackoverflow/viz.html)
- **Bitcoin（crypto）** —— 来自 `bitcoin-etl` 流水线的公开数据集
  （区块、交易、输入、输出）。演示了正文中跨表外键关系。
  · [配方](samples/crypto_bitcoin/README.md)
  · [bundle](bundles/crypto_bitcoin/)
  · [viz.html](bundles/crypto_bitcoin/viz.html)

## 可视化

`visualize` 子命令可将任意 OKF bundle 渲染为一个**自包含的交互式
HTML 文件** —— 单文件、无后端、查看端无需安装。可在任意现代浏览器中
打开、作为制品分享、托管到静态文件服务器，或像本仓库一样提交到
bundle 旁边。

这个查看器本身就是 OKF 的一个概念验证*消费者*，
与参考 agent 作为
概念验证*生产者*遥相呼应。任何能读 markdown 的工具都能消费 OKF
bundle；这只是其中一种形态。

### 它展示的内容

- 一张囊括 bundle 内每个概念的**力导向图**，节点按类型
  （datasets、tables、references 等）着色，有向边由各 markdown
  正文中的交叉链接绘制而成。
- 选中概念时的**详情面板**，展示其 frontmatter（描述、资源链接、
  标签）以及渲染后的 markdown 正文 —— 其中的内部
  `[…](/path/to/concept.md)` 链接被重新接线，以便在查看器内部
  导航，而非跟随路径。
- 每个概念下方的**「Cited by」反向链接**列表（由链接图的
  反向计算得出）。
- 一个**搜索框**（匹配标题、概念 id 与标签）、一个**类型过滤器**，
  以及可切换的图谱布局（cose / concentric / breadth-first / circle / grid）。

### 生成

```
.venv/bin/python -m reference_agent visualize --bundle ./bundles/<name>
```

这会把文件写入 `bundles/<name>/viz.html`。可用参数：

| 参数           | 默认值              | 说明                                        |
|----------------|---------------------|---------------------------------------------|
| `--bundle`     | *(必填)*            | bundle 根目录。                             |
| `--out`        | `<bundle>/viz.html` | 输出 HTML 路径。                            |
| `--name`       | bundle 目录名       | 在查看器标题中显示的名称。                  |

示例，把输出写到别处并重写标题：

```
.venv/bin/python -m reference_agent visualize \
    --bundle ./bundles/crypto_bitcoin \
    --out /tmp/btc.html \
    --name "Bitcoin OKF"
```

### 构建方式

该 HTML 将 bundle 作为 JSON blob 嵌入，并使用
[Cytoscape.js](https://js.cytoscape.org/) 绘制图谱、使用
[marked](https://marked.js.org/) 在浏览器内渲染 markdown，两者均从
CDN 加载。数据不会离开页面；bundle 在生成时解析一次并序列化进文件。

## 测试

```
.venv/bin/pytest
```
