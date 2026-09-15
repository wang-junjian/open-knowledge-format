# 贡献指南

感谢你有兴趣为 Open Knowledge Format（OKF）
做出贡献！

贡献一般分为两类，
审核方式有所不同：

- **格式本身**（[`SPEC.md`](SPEC.md)）—— OKF 的目标是成为一种通用、
  厂商中立的格式，因此规范改动的标准更高。在提交 pull request 之前，
  请先开一个 issue 说明问题以及拟议的改动，以便先就设计展开讨论。
- **参考 agent、查看器、样例与 bundle** —— 用于演示如何生成与消费
  OKF 的概念验证工具。欢迎提交普通的 pull request。

开始贡献的
步骤：

1. 签署贡献者许可协议
   （详见下文）。
1. Fork 本仓库，
   开发并测试你的代码改动。
1. 确保你的代码
   符合现有的代码风格。
1. 确保你的代码包含一组适当的单元测试且全部通过。
1. 通过运行 `.venv/bin/pytest` 确保所有测试通过（环境配置
   参见 [README.md](README.md)）。
1. 提交一个 pull request。

## 贡献者许可协议

对本项目的贡献必须随附一份贡献者许可协议（Contributor License
Agreement）。你（或你的雇主）保留所贡献内容的版权；该协议只是
授予我们将你的贡献作为项目的一部分使用并再分发的许可。前往
<https://cla.developers.google.com/> 可查看你已签署的文件或签署新协议。

通常你只需提交一次 CLA 即可，因此如果你此前已经提交过（即使是为
另一个项目提交的），大概率无需再次提交。

## 代码评审

所有提交，包括项目成员的提交，都需要经过评审。我们
使用 GitHub 的 pull request 来进行评审。如需了解如何使用
pull request，请参阅 [GitHub Help](https://help.github.com/articles/about-pull-requests/)。
