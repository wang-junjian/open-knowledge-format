# 将 OKF bundle 发布到 Google Cloud Knowledge Catalog

把一个 OKF bundle 推送到 [Knowledge Catalog][kc] 的 EntryGroup，
再以干净的 OKF 拉取回来，
使用的工具是来自 [`toolbox/mdcode`][mdcode] 的 **`kcmd`**。

请先阅读 [限制](#limitations)。

[kc]: https://docs.cloud.google.com/dataplex/docs/catalog-overview
[mdcode]: https://github.com/GoogleCloudPlatform/knowledge-catalog/tree/main/toolbox/mdcode
[demo]: https://github.com/GoogleCloudPlatform/knowledge-catalog/tree/main/toolbox/mdcode/demo/okf

## 前置条件

一个已启用 Dataplex API 的 GCP 项目，
并安装 `gcloud` 与 [Bun](https://bun.sh)。

```bash
gcloud auth application-default login
gcloud config set project <your-project-id>
gcloud config set compute/region us-central1
```

两项都要设置。
`kcmd` 从 `gcloud config` 读取项目与位置；既无对应 flag，也无环境变量。

构建 CLI：

```bash
git clone https://github.com/GoogleCloudPlatform/knowledge-catalog
cd knowledge-catalog/toolbox/mdcode
npm install && npm run build      # 生成 dist/kcmd
```

## 1. 运行示例

在改动任何内容之前，
先确认环境正常。

```bash
cd demo/okf
bun setup.ts && bun push.ts && bun pull.ts
git diff --exit-code catalog/     # 应当为空
bun cleanup.ts
```

## 2. 复制连接器

把 [`demo/okf`][demo] 中除 `catalog/` 以外的所有内容复制到你的目录，
然后编辑：

- `setup.ts` 与 `cleanup.ts` 中的 `entryGroup` ——
  硬编码为 `okf_ga4`。
- `push.ts` 与 `pull.ts` 中的 `path.resolve(root, '../../dist/kcmd')`，前提是你
  的目录不在 `toolbox/mdcode` 下两层。这两个文件也 `import * as kcmd from 'kcmd'`，
  因此请保持该模块可被解析。

把 `.staging/` 与 `catalog.yaml` 加入 `.gitignore`。

## 3. 配置目录一侧

创建 EntryGroup、自定义的 `okf` aspect 类型，以及 `catalog.yaml`。
重复运行是安全的。

```bash
bun setup.ts
```

## 4. 添加你的 bundle

```bash
cp -R /path/to/bundle/. catalog/
```

## 5. 推送

```bash
bun push.ts
```

## 6. 验证往返一致性

```bash
bun pull.ts
git diff catalog/                 # 首次拉取：YAML 规范化
git add catalog/ && bun pull.ts
git diff --exit-code catalog/     # 此刻应干净
```

首次拉取会把每个 frontmatter 块改写成 `kcmd` 的 YAML 风格 ——
序列缩进、
时间戳去引号、映射键重排、行宽重新折行。值不会变化。把这次规范化提交一次，
之后的拉取便逐字节一致，这正是 `--exit-code` 值得运行的原因。
示例之所以跳过这步，
只是因为它的 `catalog/` 本来就由一次拉取生成。

还有两类内容差异是预期内的，
不属于转换损耗：

- 没有 `index.md` 的目录会多出一个 —— `kcmd` 会为每个目录合成一个 `index` 条目。
- 带有 `resource:` 却没有 `title:` 的文档，拉取回来时 `title:` 会被设为资源 URI。

任何其他差异都是转换未能携带的键。参见 [限制](#limitations)。

## 7. 清理

删除 EntryGroup。`okf` aspect 类型会原样保留 ——
它的作用域是项目而非你的
EntryGroup，因此项目内的每个 OKF bundle 共享同一个。当不再需要时，`cleanup.ts`
会打印出移除它的命令。

```bash
bun cleanup.ts
```

## 限制

- **会携带七个 frontmatter 键**，外加 markdown 正文。`title`、`description`
  与 `tags` 变为原生的 entry 字段，`resource` 变为
  `catalogEntry.resource.name`，而 `type`、`generated` 与 `sources` 挂在
  `okf` aspect 上。若要携带更多，请向 `okf-aspect.json` 与 `okf.ts`
  添加字段，并保持既有 `index` 值不变。
- **只携带 `.md` 文件。** bundle 中的其他任何内容 —— 图片、HTML、
  CSV —— 在双向传输中都会被忽略：既不推送，拉取时也不动。
- **交叉链接解析不到任何目标。** 相对路径（§6.1）会原样存储。
  不要在源文件中改写它们 —— 相对形式才是 GitHub 上渲染出来的样子。
- **标签会变成设为 `"true"` 的 entry 标签**，且只有取值恰好为该值的标签
  会被读回成标签。Dataplex 将标签键的长度上限设为 128 个字符。
- **重命名会让目录状态变成孤儿，删除会留下残留条目**，而且没有合并方案。
  以 git 为准：合并时再推送，不要往受跟踪的 bundle 里拉取。
- **没有条目级访问控制。** 项目上任何拥有基础角色的人都能读取并批量导出
  EntryGroup —— `roles/viewer` 是 `roles/dataplex.catalogViewer` 的严格超集，
  并额外增加了 `entryGroups.export`。默认不公开，但推送前请先确认你的
  bundle 会暴露什么。
- **规模未经测试**，仅验证过 14 个文件的示例。推送按文件进行，且 Dataplex
  会强制执行配额。
