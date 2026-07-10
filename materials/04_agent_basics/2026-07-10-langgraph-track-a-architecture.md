# LangGraph 与 Track A 混合 Agent 架构

Status: verified + team design<br>
Owner: team<br>
Checked: 2026-07-10

Sources:

- https://docs.langchain.com/oss/python/langchain/overview
- https://docs.langchain.com/oss/python/langgraph/overview
- https://docs.langchain.com/oss/python/langgraph/workflows-agents
- https://docs.langchain.com/oss/python/langchain/middleware/overview
Expires/Risk: medium，框架 API 可能变化

## 需要修正的旧印象

LangChain 和 LangGraph 不是互斥的两条路线。

当前官方文档的定位是：

- LangChain `create_agent`：较高层、可配置的 Agent harness；
- LangGraph：较低层的 stateful orchestration framework；
- LangChain Agent 本身构建在 LangGraph 上；
- LangGraph 适合组合 deterministic workflow 与 agentic decision；
- evaluator-optimizer 是 LangGraph 官方列出的常见模式。

因此，Track A 不是“不适合 LangGraph”。更准确的判断是：

> Track A 不适合把三个计费工具直接交给一个无约束的通用 Agent loop，但很适合用 LangGraph 表达有状态、可分支、可回滚的混合 Agent。

## 推荐结构

```text
Task Loader
  ↓
Analyzer / Failure Classifier (LLM)
  ↓
Action Proposal (LLM, structured output)
  ↓
Budget And Safety Gate (deterministic Python)
  ↓
csim / synth / cosim (real tools)
  ↓
Tool Result Parser
  ↓
Tool-Grounded Reflector (LLM)
  ↓
Candidate Manager (accept / reject / rollback)
  ↓
Continue Or Stop
```

模型负责提出策略，规则层负责保证动作合法，工具负责提供事实。

## 建议 State

State 应保存原始数据，而不是预先拼好的 prompt：

```text
task_spec
starting_code
current_candidate
best_verified_code
best_latency
best_resources
verification_level
remaining_budget
reserved_budget
tool_history
failed_hypotheses
current_failure_type
requires_cosim
iteration
token_usage
time_remaining
stop_reason
```

其中 `verification_level` 可以是：

```text
unverified
csim_passed
synth_passed
cosim_verified
```

## LLM 应输出结构化决策

不要只让模型返回一段自然语言。建议约束成：

```json
{
  "action": "modify_and_synth",
  "hypothesis": "inner loop has II=4 because the accumulator dependency is not pipelined",
  "proposed_change": "restructure the reduction and apply a bounded partial unroll",
  "expected_effect": "lower II or total latency",
  "risk": "medium",
  "required_tools": ["csim", "synth"],
  "confidence": 0.68
}
```

具体字段需要通过模型兼容性实验确认，但核心原则是：模型必须说明动作、依据、预期收益和风险。

## 哪些必须由确定性代码控制

- 工具费用是否可承担；
- 是否侵占最终验证预留；
- 是否对完全相同的代码重复调用相同工具；
- 候选是否已经过要求的验证；
- 失败候选能否覆盖 `best_verified_code`；
- 最大 token、时间和轮数；
- 返回代码时必须选择已知最优的已验证版本。

这些约束可以放在 LangGraph node、conditional edge 或自定义 middleware 中，不能只写在 prompt 里。

## Reflection 如何落地

Track A 应使用 Tool-Grounded Reflection：

```text
不是：模型觉得自己的代码可能更好

而是：
  csim/synth/cosim 给出真实结果
  → parser 提取事实
  → reflector 分析失败原因和下一假设
  → refiner 生成新候选
```

这对应 LangGraph 的 evaluator-optimizer，但 evaluator 的核心证据来自 Vitis 工具，而不是另一个模型的主观评分。

## 为什么不是固定 workflow

图的节点集合可以固定，但路径不需要固定：

- 功能错误可以多轮 `csim → repair`；
- synth 失败可以回到专门的 synth-fix node；
- latency 不改善可以换另一类 HLS 策略；
- structural risk 高时可以请求 cosim；
- 预算不足时直接返回 best。

固定的是安全边界，动态的是诊断、优化假设和路径选择。

## 当前不需要优先学习的功能

- Deep Agents；
- 多 Agent 协作；
- 长期记忆与 RAG；
- 人工审批流程；
- LangSmith cloud deployment。

这些能力不一定无用，但不是一个月内 Track A baseline 的主要瓶颈。LangSmith tracing 可以作为可选开发工具，不应成为参赛系统的硬依赖。

## 迁移路线

1. 先跑通 reference Python loop；
2. 把 state、failure classifier 和 candidate manager 显式化；
3. 用普通 Python 实现预算与停止策略；
4. 当分支、回滚和日志追踪开始复杂时，再用 LangGraph 表达同一状态机；
5. 对比纯 Python loop 与 LangGraph 版本的正确率、分数、credits 和 token。

框架不是创新本身。真正值得写进论文的是工具反馈如何改变决策，以及预算策略为何提高最终结果。
