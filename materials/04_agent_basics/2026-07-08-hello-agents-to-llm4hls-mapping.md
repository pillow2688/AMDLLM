# Hello Agents 到 LLM4HLS Track A 的映射笔记

Status: summarized  
Owner: team  
Checked: 2026-07-08  
Source: https://helloagents.pro/#home  
Expires/Risk: medium

## Why This Matters

Hello Agents 是一个中文 Agent 系统化教程，适合给 0 基础队友补 Agent 概念。它不能直接替代比赛 harness，但能帮助我们理解 workflow、ReAct、Plan-and-Solve、Reflection、框架路线等概念，并映射到 Track A 的 reference agent。

## What Hello Agents Covers

这个教程覆盖：

- Agent 定义、发展史、大语言模型基础
- ReAct、Plan-and-Solve、Reflection 等经典范式
- Coze、Dify、n8n 等低代码 workflow 平台
- LangChain、LangGraph、AutoGen、AgentScope 等框架
- 从零构建 Agent 框架
- 记忆、上下文工程、协议、Agentic-RL、评估和综合案例

对我们最有用的是：

```text
第 1 章：初识智能体
第 4 章：经典范式构建
第 6 章：框架开发实践
第 7 章：构建你的 Agent 框架
第 12 章：智能体性能评估
```

暂时不优先：

```text
记忆与检索
通信协议
Agentic-RL
旅行助手/深度研究/赛博小镇等综合案例
```

## Core Takeaway

Track A 不是纯 workflow，也不是完全自由 agent。

更准确的理解是：

> Track A 是一个 workflow-guided agent，或者 budget-aware agentic workflow。

也就是说：

```text
外层 workflow 锁住比赛规则：
  correctness first
  synth after correctness
  cosim is expensive
  budget is limited
  hidden grading decides final score

内层 LLM 负责关键智能动作：
  read tool feedback
  repair kernel code
  optimize latency
  infer failure cause
  propose next candidate
```

## Mapping To LLM4HLS

| Hello Agents 概念 | 通用含义 | LLM4HLS Track A 对应物 |
| --- | --- | --- |
| Perception | 感知环境状态 | 读取 task description、kernel、tool result、日志 |
| Planning | 拆解任务和制定步骤 | 先修 correctness，再 synth，再 optimize |
| Action | 调工具或执行操作 | 调用 `csim`、`synth`、`cosim`，生成新 kernel |
| Observation | 接收工具结果 | `ToolResult.phase`、log、latency、resource report |
| Memory | 保留状态和经验 | best code、verified code、budget、history、失败原因 |
| ReAct | 思考-行动-观察循环 | 分析日志 → 修改代码 → 跑工具 → 再分析 |
| Plan-and-Solve | 先计划再执行 | correctness phase → PPA phase → final check |
| Reflection | 反思和改进 | 根据失败日志判断上一版为什么错 |
| Workflow | 固定流程编排 | `agent.py` 中的 correctness-before-PPA loop |
| Evaluation | 评估 agent 能力 | hidden test、synth、cosim、scorecard、token cost |

## ReAct In Track A

Hello Agents 里的 ReAct 是：

```text
Thought → Action → Observation → repeat
```

Track A 里可以映射成：

```text
Thought:
  LLM 分析 csim/synth/cosim 日志

Action:
  生成新 kernel.cpp
  或请求调用 csim/synth/cosim

Observation:
  ToolServer 返回 ToolResult

Repeat:
  agent 根据 pass/fail、budget、latency 决定下一轮
```

注意：我们不建议让 LLM 完全自由决定工具调用。因为 `cosim` 很贵，应该由外层 budget policy 控制。

## Plan-And-Solve In Track A

Hello Agents 里的 Plan-and-Solve 是：

```text
先制定计划
再逐步执行
最后整合结果
```

Track A 里的计划非常明确：

```text
1. Load task
2. Establish initial correctness state
3. Repair until csim/cosim pass
4. Run synth to get latency/resource
5. Optimize candidate code
6. Verify each candidate by csim + synth
7. Accept only if better
8. Stop on no improvement, round cap, or budget exhaustion
9. Return best verified kernel
```

这个流程已经体现在 reference `agent.py` 中。

## Reflection In Track A

Reflection 的核心是：

```text
不要只生成结果，还要看结果哪里错了，再改。
```

Track A 中最适合做 reflection 的地方：

- `compile_error`：判断是不是语法、include、函数签名、类型问题
- `runtime_fail`：判断是不是功能逻辑 bug
- `synth_error`：判断是不是不可综合写法或 pragma 冲突
- `cosim_fail`：判断是不是 RTL/C mismatch 或 stream deadlock
- `timeout`：判断是不是综合太慢、仿真挂起或死锁
- `latency no improvement`：反思是否优化方向错误，是否该回滚

这里可以发展成我们的 `failure classifier` 和 prompt template。

## Workflow Vs Agent

Hello Agents 提到低代码平台和 workflow 平台，比如 Coze、Dify、n8n。它们适合做：

```text
固定步骤
外部 API 编排
信息流处理
快速原型
```

Track A 更适合代码里的 workflow，而不是低代码平台：

```text
Python workflow:
  更容易接入 fpt26-harness
  更容易控制 budget
  更容易记录 scorecard
  更容易做实验和论文复现
```

所以我们当前路线是：

```text
不用低代码平台搭正式 agent
不用一开始上复杂多 Agent 框架
先基于 fpt26-harness 写清楚 agentic workflow
```

## Framework Notes

Hello Agents 介绍了 LangChain/LangGraph、AutoGen、AgentScope 等。

对 Track A 的判断：

- LangGraph：适合后期把复杂状态机画成图，比如 repair/optimize/cosim-risk/checkpoint。
- AutoGen：适合多 agent 角色协作，比如 coder/reviewer/verifier，但一个月内不是第一优先。
- AgentScope：中文生态友好，可以学习思想。
- 自建框架：最贴近当前 harness，因为 reference agent 本来就是简单 Python loop。

当前建议：

```text
第 1 阶段：手写 loop，吃透 harness
第 2 阶段：加入 failure classifier / budget policy / prompt templates
第 3 阶段：如果流程变复杂，再考虑 LangGraph
第 4 阶段：如果角色明显分工，再考虑多 agent
```

## Competition-Specific Warnings

1. 教程示例常使用 GPT/Claude/Gemini 等 proprietary model。正式比赛要使用 open-source/open-weight model。
2. 通用 Agent 教程里的工具通常是 search、calculator、browser、code executor；Track A 工具是 `csim/synth/cosim`。
3. 通用 agent 常追求“完成任务”；Track A 必须追求“hidden correctness + synthesizability + PPA + budget efficiency”。
4. 不要把 workflow 平台当成比赛 harness。harness 的预算、固定文件、hidden grading 是核心边界。

## What We Should Do With It

- 把 Hello Agents 当作 Agent 基础学习材料。
- 用第 4 章理解 ReAct / Plan-and-Solve / Reflection。
- 用第 6/7 章理解框架和自建 loop 的差异。
- 用第 12 章启发我们设计 agent 评估表。
- 不把它作为最终技术路线本身；最终路线仍以 fpt26-harness 为主。

## Reading Assignment For New Teammates

建议 0 基础队友按这个顺序读：

```text
1. Hello Agents 第 1 章：知道 Agent 是什么
2. Hello Agents 第 4 章：知道 ReAct / Plan-and-Solve / Reflection
3. 本笔记：把通用概念映射到 LLM4HLS
4. docs/track-a-zero-foundation-guide.md：理解比赛全局
5. fpt26-harness 的 agent.py：看真实代码
```

## Open Questions

- 后期是否需要用 LangGraph 表达 repair/optimize/cosim-risk 状态机？
- 是否需要把 Reflection 独立成一个 LLM call，还是合并在 repair prompt 里？
- 是否需要多 Agent，例如 coder + HLS reviewer + budget manager？
