# 开放模型选择快照

Status: summarized<br>
Owner: team<br>
Checked: 2026-07-10

Sources:

- https://api-docs.deepseek.com/
- https://openrouter.ai/models
- https://www.datalearner.com/leaderboards/open-source
Expires/Risk: high，模型、slug、价格、provider 和 license 会快速变化

## 文档边界

这是一份日期快照，不是最终模型排名。没有公开榜单专门衡量 HLS Agent，最终选择必须通过 reference harness 和后续正式任务实测。

## 合规概念

必须分清：

```text
API 可调用
开放权重
开源许可证
比赛允许
```

它们不是同一件事。OpenRouter 是 provider；provider 能调用某模型，不代表该模型自动满足比赛要求。

每个候选模型至少记录：

- exact model ID / OpenRouter slug；
- model family 与具体版本；
- provider；
- license 的一手证据链接；
- 是否开放权重；
- context length；
- tool calling 与 structured output 支持；
- 价格、速率限制和稳定性；
- 检查日期。

## 当前候选方向

可优先关注这些开放模型家族：

- DeepSeek；
- Qwen Coder / Qwen 系列；
- Kimi K2 系列；
- GLM 系列；
- 其他在代码修改和工具调用评测中表现稳定的开放模型。

这不是排序。模型越新并不自动代表 HLS 代码、长日志诊断和严格格式输出更好。

## DeepSeek 当前 API 快照

DeepSeek 官方 API 文档在 2026-07-10 列出的主要 API model name 为：

```text
deepseek-v4-flash
deepseek-v4-pro
```

文档同时说明 `deepseek-chat` 与 `deepseek-reasoner` 将在 2026-07-24 15:59 UTC deprecated，并分别映射到 V4 Flash 的非 thinking 与 thinking 模式。

这些是 DeepSeek API model name，不应直接假设等于 OpenRouter slug，也不应在没有权重仓库或 license 文件证据时只凭 API 名称断言许可证。

## OpenRouter Fusion

Fusion 更像模型路由或融合产品，而不是一个固定、可复现的开放权重模型。当前不建议作为正式参赛 Agent backend，原因是：

- 背后模型组成可能变化；
- 可能混入不符合比赛要求的 proprietary model；
- 难以在论文中准确报告 exact model、license 和 token；
- 相同输入的 provider 路由不容易复现。

它可以用于赛前研究，但正式实验应锁定明确 slug、provider 和版本。

## 我们真正要测的指标

- kernel 格式正确率；
- public csim repair success；
- synth success；
- structural/cosim repair success；
- latency 和资源改善；
- tool-decision 合法率；
- 重复动作率；
- token 消耗；
- API 失败率与延迟；
- 固定预算下的最终 score。

## 当前决定

在环境跑通前不宣布“最佳模型”。先使用一个可稳定调用的开放 coder model 建 baseline，再用相同任务、prompt、预算和随机性设置比较候选。
