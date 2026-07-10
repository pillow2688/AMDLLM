# Datawhale Hello-Agents 到 LLM4HLS Track A 的映射笔记

Status: verified  
Owner: team  
Checked: 2026-07-10
Source: https://hello-agents.datawhale.cc/#/./README  
GitHub: https://github.com/datawhalechina/hello-agents
Expires/Risk: medium

## Why This Matters

Datawhale Hello-Agents 是《从零开始构建智能体》的系统性中文教程，适合给 0 基础队友补 Agent 概念。它不是 FPT Track A 的比赛材料，也不能替代 `fpt26-harness`，但它能帮助我们建立 Agent 的通用语言：ReAct、Plan-and-Solve、Reflection、框架、评估、上下文工程等。

一句话：

> Hello-Agents 帮我们学 Agent 的通用范式；fpt26-harness 才是我们参赛的主战场。

## Important Correction

之前记录过 `helloagents.pro`，现在以 Datawhale 官方文档为准：

```text
Online docs:
  https://hello-agents.datawhale.cc/#/./README

GitHub:
  https://github.com/datawhalechina/hello-agents
```

后续资料里提到 Hello-Agents 时，优先引用 Datawhale 版本。

## What The Project Says

Datawhale README 里明确说，现在 Agent 构建主要分为两派：

```text
1. 软件工程类 Agent
   代表：Dify、Coze、n8n
   本质：流程驱动的软件开发，LLM 作为数据处理后端

2. AI Native Agent
   真正以 AI 驱动的 Agent
   重点：理解核心原理、经典范式、核心架构，并动手构建多智能体应用
```

这正好解释了我们当前路线：

```text
Track A 外层很像 workflow
但关键节点由 LLM 根据工具反馈做生成、分析和修复
```

所以 Track A 更准确地说是：

> workflow-guided agent，或者 budget-aware agentic workflow。

## Which Chapters Matter For Us Now

优先读：

```text
第 1 章：初识智能体
第 4 章：智能体经典范式构建
第 6 章：框架开发实践
第 7 章：构建你的 Agent 框架
第 12 章：智能体性能评估
```

暂时不用深挖：

```text
第 5 章：低代码平台
第 8 章：记忆与检索
第 9 章：上下文工程
第 10 章：通信协议
第 11 章：Agentic-RL
第 13-16 章：综合案例和毕业设计
```

原因：

```text
比赛时间只有一个月
Track A 首要问题是工具反馈闭环和预算控制
不是先做通用多智能体产品
```

## Mapping To LLM4HLS

| Hello-Agents 概念 | 通用含义 | LLM4HLS Track A 对应物 |
| --- | --- | --- |
| Perception | 感知环境状态 | 读取 task description、kernel、tool result、日志 |
| Planning | 拆解任务和制定步骤 | correctness phase → synth → optimize → final check |
| Action | 调工具或执行操作 | 生成 kernel、调用 `csim/synth/cosim` |
| Observation | 接收工具结果 | `ToolResult.phase`、log、latency、resource report |
| Memory | 保留历史和状态 | best code、verified code、budget、失败历史 |
| ReAct | 思考-行动-观察循环 | 分析日志 → 改代码 → 跑工具 → 再分析 |
| Plan-and-Solve | 先规划再执行 | 先修正确性，再优化 PPA |
| Reflection | 反思失败并改进 | 根据 compile/runtime/synth/cosim 失败日志修正策略 |
| Framework | 组织 agent 运行结构 | 当前是自写 Python loop，也可用 LangGraph 表达混合状态机 |
| Evaluation | 衡量 agent 能力 | hidden test、synth、cosim、score、token、credit |

## ReAct In Track A

Hello-Agents 第 4 章会讲 ReAct。对应 Track A：

```text
Thought:
  LLM 分析工具反馈

Action:
  生成新 kernel.cpp
  或在外层策略允许下触发 csim/synth/cosim

Observation:
  ToolServer 返回 ToolResult

Repeat:
  agent 根据 pass/fail、budget、latency 决定下一轮
```

注意：

```text
LLM 可以看到预算和工具结果并提出动作
昂贵工具必须经过确定性的 budget/reserve gate
```

## Plan-And-Solve In Track A

Track A 的安全边界较固定，但具体修复和优化路径可以动态分支：

```text
1. Load task
2. Establish initial correctness state
3. Repair until csim/cosim pass
4. Run synth to get latency/resource
5. Optimize candidate code
6. Verify each candidate by csim + synth
7. Accept only if better
8. 根据新假设价值、验证预留、轮数和预算决定继续或停止
9. Return best verified kernel
```

这说明我们不需要“完全自由的 Agent”，也不应把 workflow 理解成一条固定直线。

更合适的是确定性边界内的动态 Agent：LLM 决定诊断和策略，状态机保证预算、验证与回滚。

## Reflection In Track A

Reflection 最适合做成 Tool-Grounded Reflection：failure classifier 是第一步，后续还要根据真实工具结果形成新假设、生成候选并判断是否接受。

失败类型和反思方向：

```text
compile_error:
  语法、include、函数签名、类型错误

runtime_fail:
  功能逻辑错误，public testbench 没过

synth_error:
  不可综合写法、pragma 冲突、资源/约束问题

cosim_fail:
  RTL/C mismatch、stream deadlock、结构问题

timeout:
  工具超时、仿真挂起、可能死锁

latency no improvement:
  优化方向无效，应回滚或换策略
```

这可以直接指导我们的 prompt 模板：

```text
repair_compile_prompt
repair_runtime_prompt
repair_synth_prompt
repair_cosim_deadlock_prompt
optimize_latency_prompt
```

## Framework Reading Notes

Hello-Agents 会介绍低代码平台和框架。

对我们来说：

```text
Dify / Coze / n8n:
  了解即可，不适合作为 Track A 主体。

LangGraph:
  官方定位是组合 deterministic workflow 与 agentic decision 的底层编排框架。
  Track A 很适合这种混合模式，但应先理解 reference loop 再迁移。

AutoGen / AgentScope:
  适合多 agent 分工，但不是第 1 优先级。

从 0 构建 Agent 框架:
  最值得看，因为 fpt26-harness reference agent 本质就是一个小型自建框架。
```

## Competition-Specific Warnings

1. Hello-Agents 的示例可能使用 OpenAI 原生 API 或其他 proprietary model。正式比赛要使用 open-source/open-weight model。
2. 通用 Agent 教程里的工具通常是搜索、浏览器、计算器、API；Track A 工具是 `csim/synth/cosim`。
3. 通用 Agent 追求“完成任务”；Track A 追求 hidden correctness、synthesizability、PPA、credit efficiency。
4. 不要把低代码 workflow 平台当成比赛 harness。harness 的预算、固定文件、hidden grading 是核心边界。
5. Hello-Agents 是学习材料，不是可以直接提交的参赛系统。

## What We Should Do With It

短期：

- 用第 1 章统一 Agent 基础语言。
- 用第 4 章理解 ReAct / Plan-and-Solve / Reflection。
- 用第 7 章理解为什么我们可以先做自写 agent loop。
- 用第 12 章启发我们的评估表。

中期：

- 把 Reflection 变成 failure classifier。
- 把 Reflection 扩展成基于工具证据的诊断与新假设生成。
- 把 Plan-and-Solve 变成带条件分支、预算预留和回滚的稳定 workflow。
- 把 ReAct 思路用于日志分析和候选代码迭代。
- 对照阅读本仓库的 LangGraph 架构和最优停止专题。

不做：

- 不用低代码平台搭最终 agent。
- 不急着做多 agent。
- 不急着做记忆/RAG/通信协议/Agentic-RL。

## Reading Assignment For New Teammates

建议顺序：

```text
1. docs/track-a-zero-foundation-guide.md
2. Hello-Agents 第 1 章
3. Hello-Agents 第 4 章
4. 本笔记
5. fpt26-harness 的 agent.py
6. Hello-Agents 第 7 章
7. Hello-Agents 第 12 章
```

## Open Questions

- 是否需要把 reference agent 改成显式状态机？
- 是否要用 LangGraph 表达 repair/optimize/cosim-risk/checkpoint？
- Reflection 是单独一次 LLM call，还是合并在 repair prompt 里？
- 是否需要 coder + reviewer + budget manager 这种多 Agent 分工？
