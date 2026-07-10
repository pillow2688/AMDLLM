# LLM4HLS Track A Team Knowledge Base

这个仓库用于团队共享 FPT 2026 Track A / LLM4HLS 的学习资料、阅读笔记、环境说明和实验记录。

## Start Here

新队友先读这四份：

1. [Track A 比赛总览](docs/track-a-competition-overview.md)
2. [Track A 0 基础入门手册](docs/track-a-zero-foundation-guide.md)
3. [文档入口](docs/README.md)
4. [资料库目录规则](materials/README.md)

然后按主题继续读：

- [HLS 基础资料](materials/03_hls_basics/README.md)
- [Agent 基础资料](materials/04_agent_basics/README.md)
- [模型与 API 资料](materials/05_models_and_apis/README.md)
- [环境资料](materials/06_environment/README.md)

## Repository Layout

```text
docs/       面向全队阅读的成品文档和文档入口
materials/ 资料库、阅读笔记、模型记录、环境记录、实验记录
```

不提交到 Git：

```text
_external/  下载的外部仓库，例如 fpt26-harness
runs/       HLS/agent 运行输出
.env        API key、账号、私密环境变量
.agents/    本地 agent 状态
.codex/     本地 Codex 状态
```

## Current Key Materials

- [Track A 比赛总览](docs/track-a-competition-overview.md)
- [Track A 0 基础入门手册](docs/track-a-zero-foundation-guide.md)
- [官方规则快照](materials/01_official/2026-07-10-track-a-official-summary.md)
- [Reference harness 分析](materials/02_harness/2026-07-10-reference-harness-analysis.md)
- [LangGraph 与 Track A 混合 Agent 架构](materials/04_agent_basics/2026-07-10-langgraph-track-a-architecture.md)
- [预算感知工具策略与最优停止](materials/04_agent_basics/2026-07-10-budget-aware-tool-policy-and-optimal-stopping.md)
- [Datawhale Hello-Agents 到 LLM4HLS Track A 的映射笔记](materials/04_agent_basics/2026-07-09-datawhale-hello-agents-to-llm4hls-mapping.md)
- [hls-generator 资源笔记](materials/03_hls_basics/2026-07-08-hls-generator-resource-note.md)

## Team Rules

1. 正式资料只认本仓库。
2. 群聊只发链接，不发散乱文件。
3. 新资料先按 `materials/README.md` 分类。
4. 模型/API 相关资料必须写清楚来源、检查日期和 license 风险。
5. 实验记录必须使用 `materials/_templates/experiment-log-template.md`。
6. 不提交 API key、license server 信息、私有服务器密码。
7. 文档必须区分官方规则、reference harness 实现和团队设计判断。

## Upstream References

- FPT 2026 Design Competition: <https://fpt2026.uark.edu/fpt26-design-competition/>
- FPT26 harness: <https://anonymous.4open.science/r/fpt26-harness/README.md>
- hls-generator: <https://github.com/Eriemon/hls-generator>
- Datawhale Hello-Agents: <https://hello-agents.datawhale.cc/#/./README>
- Datawhale Hello-Agents GitHub: <https://github.com/datawhalechina/hello-agents>
