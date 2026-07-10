# Agent Basics

放 agent 基础、tool-calling、状态管理、budget policy、框架对比等资料。

适合放：

- agent 最小闭环
- hand-written loop
- LangChain / LangGraph / AutoGen / CrewAI 对比
- tool use / observation / action / memory
- budget-aware policy
- failure classifier 设计

不适合放：

- 纯 HLS 教程
- API key 或私密配置
- 单次实验 scorecard

命名建议：

```text
YYYY-MM-DD-agent-loop-basics.md
YYYY-MM-DD-langchain-vs-handwritten-loop.md
YYYY-MM-DD-budget-policy-design.md
```

## Current Notes

- [Datawhale Hello-Agents 到 LLM4HLS Track A 的映射笔记](2026-07-09-datawhale-hello-agents-to-llm4hls-mapping.md)
- [LangChain v1 到 LLM4HLS Track A 的映射](2026-07-10-langchain-v1-track-a-mapping.md)
- [LangGraph 与 Track A 混合 Agent 架构](2026-07-10-langgraph-track-a-architecture.md)
- [预算感知工具策略与最优停止](2026-07-10-budget-aware-tool-policy-and-optimal-stopping.md)
