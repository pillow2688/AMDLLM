# Materials Index

这里放团队资料总索引、推荐阅读顺序和待归类条目。

## 推荐阅读顺序

1. `docs/track-a-competition-overview.md`：先看清比赛全貌和信息边界
2. `docs/track-a-zero-foundation-guide.md`：补齐 FPGA、HLS 和 Agent 基础
3. `01_official/`：确认比赛规则和关键日期
4. `02_harness/`：理解 reference harness
5. `03_hls_basics/`：补 FPGA/HLS 基础
6. `04_agent_basics/`：理解 Agent 闭环、LangGraph 与预算策略
7. `06_environment/`：确认 Vitis/HLS 工具链
8. `05_models_and_apis/`：选择开放模型和 API provider
9. `07_experiments/`：记录模型和 Agent 实验
10. `08_paper_and_demo/`：沉淀论文和 demo 材料

## 最新提交要求

- [Track-A Submission Guidelines 中文翻译与执行清单](../01_official/2026-07-10-track-a-submission-guidelines-zh.md)
- [FPT 2026 Track A 官方规则快照](../01_official/2026-07-10-track-a-official-summary.md)
- [开放模型选择快照](../05_models_and_apis/2026-07-10-open-model-selection-snapshot.md)
- [本机环境快照](../06_environment/2026-07-10-local-environment-snapshot.md)

## Agent 专题阅读顺序

1. [Datawhale Hello-Agents 到 Track A 的映射](../04_agent_basics/2026-07-09-datawhale-hello-agents-to-llm4hls-mapping.md)：先建立 Agent 通用概念
2. [LangChain v1 到 Track A 的映射](../04_agent_basics/2026-07-10-langchain-v1-track-a-mapping.md)：理解 `create_agent`、middleware 和 LangGraph 的关系
3. [LangGraph 与 Track A 混合 Agent 架构](../04_agent_basics/2026-07-10-langgraph-track-a-architecture.md)：理解团队推荐架构
4. [预算感知工具策略与最优停止](../04_agent_basics/2026-07-10-budget-aware-tool-policy-and-optimal-stopping.md)：进入比赛的核心决策问题

## 待归类资料

如果不确定某份资料该放哪里，先在 `triage.md` 记录，不要直接堆到根目录。
