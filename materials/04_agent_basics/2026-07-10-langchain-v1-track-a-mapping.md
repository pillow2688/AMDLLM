# LangChain v1 到 LLM4HLS Track A 的映射

Status: verified + team interpretation<br>
Owner: team<br>
Checked: 2026-07-10

Sources:

- 中文辅助阅读：https://langchain-doc.cn/v1/python/langchain/releases/langchain-v1.html
- 官方 v1 发布说明：https://docs.langchain.com/oss/python/releases/langchain-v1
- 官方 v1 迁移指南：https://docs.langchain.com/oss/python/migrate/langchain-v1
- 官方 middleware 概览：https://docs.langchain.com/oss/python/langchain/middleware/overview
- 官方 LangGraph v1 说明：https://docs.langchain.com/oss/python/releases/langgraph-v1

Expires/Risk: high，LangChain API 和集成支持范围可能继续变化；实现时必须再次核对官方文档和锁定的包版本

## 先说结论

这个页面对比赛有帮助，但它证明的不是“我们必须改用 LangChain”，而是：

> LangChain v1 已经把 Agent 最小循环、middleware 和 LangGraph 的关系整理成了清晰的分层，我们可以借用这套职责划分设计 Track A Agent。

对我们最有价值的顺序是：

1. `middleware`：启发预算注入、工具拦截、日志和结果校验；
2. structured output：约束模型提出可检查的动作；
3. `create_agent`：快速搭建标准工具调用循环；
4. LangGraph：管理跨多轮的修复、优化、回滚和停止；
5. standard content blocks：可能方便切换模型供应商，但不是当前主线；
6. namespace 精简：主要影响导入和迁移，对比赛策略没有直接收益。

## 最小心智模型

先不要把这些名字看成四套竞争框架：

```text
LangChain create_agent
  = 已封装好的基础 Agent loop
  = 模型 → 工具调用 → 工具结果 → 模型 → 结束

middleware
  = 插在这个 loop 各阶段的可复用控制逻辑

LangGraph
  = create_agent 底下的运行与状态图基础
  = 也可直接用于更复杂的全局编排

fpt26-harness
  = 比赛参考运行环境和 Agent 起点
  ≠ LangChain 或 LangGraph
```

LangChain v1 官方把 `langchain.agents.create_agent` 作为构建 Agent 的标准高层入口，取代旧教程里常见的 `langgraph.prebuilt.create_react_agent`。`create_agent` 本身仍构建在 LangGraph 上，所以“用 LangChain”与“用 LangGraph”不是二选一。

## v1 三个核心变化

### 1. `create_agent`

它封装了通用的工具调用循环。开发者提供：

- 模型；
- 工具；
- system prompt；
- 可选 middleware；
- 可选结构化输出格式。

框架负责反复调用模型和工具，直到模型不再提出工具调用。

对 Track A 的意义：它可以少写一部分通用 loop 代码，但不会自动理解 HLS、credits、候选代码回滚或最终提交标准。这些仍是团队必须实现的比赛逻辑。

### 2. Standard content blocks

`content_blocks` 尝试把不同模型供应商的文本、推理、工具调用等内容表示统一起来。

可能的好处：

- 更换兼容的模型集成时，解析层改动更少；
- 统一读取文本和 tool call；
- 减少供应商专用字段散落在业务代码里。

当前限制：

- 支持范围依赖具体 LangChain integration 和版本；
- OpenRouter 的 OpenAI-compatible 接口不等于所有 content block 都会被完整归一化；
- harness 已有自己的工具结果结构时，仍应保留稳定的 `ToolResult` parser。

因此它属于“模型适配层优化”，优先级低于预算控制和候选管理。

### 3. 简化 namespace

v1 把 Agent 核心入口集中在 `langchain.agents`、`langchain.messages`、`langchain.tools` 等模块，旧 chains 和 retrievers 等能力移到 `langchain-classic`。

对新项目的实际提醒：

- 不要照搬 v0.x 教程的 import；
- 不要为了兼容旧教程主动引入 `langchain-classic`；
- Track A 不需要传统 RAG chains，namespace 变化不是我们的设计重点。

## Middleware 到 Track A 的映射

Middleware 最值得学习，因为它把“只写在 prompt 里的提醒”变成了代码控制点。

| Hook | 通用时机 | Track A 可承担的职责 |
| --- | --- | --- |
| `before_agent` | 整个 Agent 开始前 | 校验 task、绑定已有预算状态、加载 baseline 与提交要求 |
| `before_model` | 每次模型调用前 | 注入剩余预算、best candidate、最近工具事实，裁剪无关历史 |
| `wrap_model_call` | 包裹模型调用 | 超时、重试、模型 fallback、token 统计 |
| `after_model` | 模型返回后 | 校验 action schema、拒绝未知动作、检查参数完整性 |
| `wrap_tool_call` | 包裹工具调用 | 扣费检查、预留保护、重复调用拦截、缓存、统一异常转换 |
| `after_agent` | Agent 结束后 | 校验返回物、记录运行摘要、清理局部资源 |

其中 `wrap_tool_call` 很像我们讨论的 deterministic budget gate：

```text
模型提出 synth
  ↓
检查工具名和参数是否合法
  ↓
检查本次费用是否可承担
  ↓
检查是否侵占 final verification reserve
  ↓
检查相同代码是否已经跑过同一工具
  ↓
允许执行或返回结构化拒绝原因
```

但是，middleware 不是预算策略本身。团队仍要定义：

- 每个工具在当前 harness 中的真实费用；
- 何时必须预留最终验证；
- 什么结果值得升级到 synth 或 cosim；
- 什么条件触发停止；
- 哪个候选可以成为最终提交。

`wrap_tool_call` 只能拦截注册在该 `create_agent` 中的工具。如果 csim、synth 或 cosim 由外层 Python/LangGraph node 直接调用，它们不会自动经过这里。因此预算规则应先实现为独立、可复用的 policy/service，再由 middleware 和外层工具节点共同调用，不能维护两套 credits 账本。

## Structured Output 为什么重要

自然语言动作很难被可靠检查：

```text
“我觉得可以再综合一下看看。”
```

更适合比赛的是结构化决策：

```json
{
  "action": "run_synth",
  "candidate_id": "candidate_07",
  "hypothesis": "partial unroll may reduce latency within the DSP limit",
  "expected_value": 0.63,
  "estimated_cost": 4,
  "risk": "medium"
}
```

确定性代码随后检查：

- `action` 是否在允许集合；
- candidate 是否存在；
- `estimated_cost` 是否与真实计费一致；
- 剩余 credits 是否足够；
- 是否应该拒绝、降级或执行。

LangChain v1 可以帮助生成并解析结构化输出，但真实费用和安全判断必须由本地代码覆盖，不能相信模型自己填写的数字。

如果使用 `create_agent` 的 `response_format`，结构化结果通常代表 Agent 的返回物；模型在主循环中仍可能选择调用工具。对于“每轮先提出动作，再由 budget gate 批准”的架构，更清晰的做法是使用独立 planner node 产生上述 action，或把真实工具调用本身定义成有严格 schema 的 tool，而不是假设 `response_format` 会替代所有 tool call。

## `create_agent` 和 LangGraph 应如何分工

### 只用 `create_agent` 的情况

适合：

- 验证一个最小工具调用闭环；
- 工具少，状态简单；
- 没有复杂的回滚和多阶段验证；
- 先测试某个模型能否正确使用 HLS 工具说明。

### 外层再用 LangGraph 的情况

适合：

- repair、optimize、verify 是不同阶段；
- 不同失败类型要进入不同路径；
- 需要保存 `best_verified_code`；
- 需要在 csim、synth、cosim 之间做预算决策；
- 需要 checkpoint、恢复、终止原因和可复现实验轨迹。

推荐分层：

```text
Outer Python state machine / LangGraph
  ├─ task initialization
  ├─ repair route
  ├─ optimization route
  ├─ candidate manager
  ├─ budget and stop policy
  └─ final verification
         ↓
LangChain create_agent or a hand-written LLM node
  ├─ analyze tool evidence
  ├─ propose modification
  └─ return structured action
         ↓
middleware and tool wrappers
  ├─ validate calls
  ├─ enforce credits
  ├─ normalize errors
  └─ record observations
```

`create_agent` 可以作为外层图中的一个 node 或 subgraph。对于 Track A，外层全局状态比让一个通用 Agent loop 自己记住全部规则更可靠。

## 哪些规则绝不能只交给 LLM

- credits 不允许透支；
- final verification reserve 不允许被普通探索消耗；
- 相同代码和参数不重复调用昂贵工具；
- 未验证候选不能覆盖 `best_verified_code`；
- 工具报告优先于模型猜测；
- 达到硬停止条件后必须返回 best；
- 最终输出必须符合比赛文件和接口约束。

这些规则可以由普通 Python、LangChain middleware 或 LangGraph node 实现，但必须存在一个确定性的权威状态层。

## 中文页面该怎么用

这个中文页面适合第一次理解概念，但它是社区翻译，不是 LangChain 官方中文站。页面显示的更新时间早于本次检查日期，部分示例参数和当前官方示例已经不同。

使用规则：

1. 中文页面负责理解术语和整体结构；
2. 官方 release、migration 和 middleware 页面负责确认 API；
3. 实现时锁定依赖版本，并以对应版本 reference 为准；
4. 不复制页面中的模型名作为比赛模型选择；
5. 不因为示例能运行就认为它满足 Track A 的模型、预算和提交规则。

## 一个月赛程下怎么学

### 现在先学

1. 看懂 `create_agent` 的模型—工具循环；
2. 看懂六类 middleware hook 的执行位置；
3. 把 `csim/synth/cosim` 调用经过的 budget gate 画出来；
4. 看懂 structured action 为什么比自由文本可靠；
5. 对照 reference harness 找到模型调用、工具调用、预算和最终返回的位置。

### 有 baseline 后再做

1. 用同一批任务比较 hand-written loop 与 `create_agent`；
2. 只迁移一个职责，例如工具调用日志或 action validation；
3. 当分支和状态真的变复杂，再迁移到 LangGraph；
4. 用 correctness、score、credits、token、wall time 比较，不凭代码是否“高级”做判断。

### 当前先不做

- 不因发布说明而重写 reference harness；
- 不先做多 Agent；
- 不引入长期记忆、RAG 或人工审批；
- 不把所有 budget policy 写成 middleware 后就认为问题解决了；
- 不让框架选型替代 HLS 工具反馈实验。

## 对我们当前路线的影响

这份资料强化了现有判断：

> 推荐方向仍是 hybrid agent：LLM 负责诊断、生成和提出动作，确定性状态层负责预算、验证、回滚和停止。

它带来的新启发是：我们不必在“纯手写 loop”和“完整 LangGraph”之间一次性二选一。可以先在 reference loop 中实现清晰的 policy、state 和 tool wrapper，再按需要把某些职责映射到 LangChain middleware，最后才决定是否需要完整 LangGraph。

## 采用框架前的实验问题

- 同一模型、prompt 和任务下，`create_agent` 是否提高合法 tool call 比例？
- middleware 是否能完全阻止超预算和重复昂贵调用？
- structured output 失败率是多少，失败后如何恢复？
- 框架增加了多少 token、延迟和异常路径？
- checkpoint 是否真的帮助恢复长时间 synth/cosim 任务？
- 最终得分提升是否大于框架复杂度带来的风险？

只有实验结果能决定是否采用，发布说明本身不能。
