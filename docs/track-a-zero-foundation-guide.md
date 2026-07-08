# FPT 2026 Track A / LLM4HLS 0 基础入门笔记

整理时间：2026-07-07  
适用对象：第一次接触 FPGA、HLS、Agent 的队友  
目标：先看懂比赛在做什么，再知道一个月内该优先学什么、做什么。

---

## 1. 一句话理解这个比赛

Track A 要做的不是普通 HLS 代码，也不是普通聊天机器人，而是：

> 做一个会自动修改 HLS C++ kernel 的 agent。它要在有限预算下调用 `csim`、`synth`、`cosim`，把代码先修正确，再尽量优化性能。

最小闭环是：

```text
读题目和初始 kernel.cpp
  ↓
让 LLM 生成/修改 HLS C++ 代码
  ↓
调用 csim / synth / cosim
  ↓
读取日志、latency、资源等结果
  ↓
判断下一步：修 bug、优化、回滚、停止
  ↓
提交当前最好版本
```

所以它本质是一个“带工具、带预算、带评测反馈的自动改代码循环”。

---

## 2. Track A 到底要求什么

官方 Track A 名字是 **Budgeted End-to-End LLM4HLS Agent**。

可以拆成三层：

```text
LLM       负责理解任务、生成/修改 kernel.cpp
HLS 工具  负责验证代码是否正确、是否能综合、性能如何
Agent     负责调度 LLM 和工具，并控制预算
```

比赛任务通常会给：

- 初始 C/C++ kernel
- 固定 header
- public testbench
- 任务描述和接口约束
- 目标 FPGA / HLS 版本 / clock
- 调用工具的预算

最终不是看 LLM 说得好不好，而是看 hidden test 和 PPA 分数。

---

## 3. Harness 是什么

官方给的 harness 不是“正式提交模板那么简单”，而是：

> 一个 Track A 的参考 agent + 评测 harness。

它的作用是让我们能本地模拟比赛流程：

```text
task package
  → metered csim / synth / cosim tools
  → reference agent loop
  → hidden-test grading + PPA scorecard
```

我们可以从它开始改，而不是从零写整个系统。

重要结论：

- 它只是示例实现，不是必须完全照抄。
- 参赛者可以实现自己的 agent 和 harness。
- 但它已经给出了比赛最核心的接口、预算、评分方式。
- 一个月时间紧，建议先基于它增强，而不是重做大框架。

---

## 4. HLS 是什么

HLS = High-Level Synthesis，高层次综合。

普通 C/C++：

```text
C/C++ → 编译器 → CPU 指令 → CPU 执行
```

HLS：

```text
HLS C/C++ → Vitis HLS → RTL 硬件描述 → FPGA 上的电路
```

也就是说，HLS C++ 不是普通 CPU 程序。它最后会变成硬件电路。

一句话：

> HLS 是把 C/C++ 风格的代码变成 FPGA 硬件电路的工具链。

---

## 5. CPU 和 FPGA 差在哪

CPU 像一个很快的工人：

```text
一条指令
一条指令
一条指令
顺序执行
```

FPGA 像一块可以重新搭建的工厂：

```text
读数据模块
乘法模块
加法模块
缓存模块
控制模块
可以同时工作
```

所以 FPGA/HLS 关心的不是“代码看起来短不短”，而是：

- 会生成多少硬件
- 能不能并行
- 能不能流水线
- 数据能不能供得上
- 时钟能不能过
- 资源会不会爆

---

## 6. 三个工具：csim / synth / cosim

比赛 harness 里最重要的三个工具：

```text
csim  = 1 credit
synth = 4 credits
cosim = 20 credits
```

### csim

`csim` = C simulation。

作用：

```text
编译 kernel + testbench
在 C/C++ 层面运行
检查功能是否对
```

特点：

- 最便宜
- 速度最快
- 主要查功能 bug
- 不能发现所有硬件结构问题

### synth

`synth` = C synthesis。

作用：

```text
把 HLS C++ 综合成硬件
检查是否可综合
生成 latency / II / resource report
```

它会告诉我们：

- latency 多少 cycles
- II 是多少
- LUT / FF / DSP / BRAM / URAM 用了多少
- 是否综合失败

### cosim

`cosim` = C/RTL co-simulation。

作用：

```text
先综合成 RTL
再用 testbench 跑 RTL 仿真
检查 C 层面正确的代码，在真实硬件语义下是否也正确
```

特点：

- 最贵
- 最慢
- 能发现 stream deadlock、RTL/C mismatch 等问题
- 不能随便跑，要判断值得花 20 credits 时再跑

一句话：

```text
csim 查功能
synth 查能不能变硬件和性能
cosim 查 RTL 级别是否真的能跑
```

---

## 7. 为什么不能让 csim 干 cosim 的活

不能。

因为 `csim` 只是在 C/C++ 层面跑，它的某些行为比真实硬件“宽松”。

典型例子是 `hls::stream`：

```text
C-sim 中 stream 可能像无限队列
RTL 中 FIFO 深度有限，比如 depth=2
```

所以有些代码：

```text
csim 通过
cosim 死锁
```

这就是为什么比赛里 `cosim` 很贵但必要。

更好的策略不是“用 csim 替代 cosim”，而是：

> 先用静态规则/日志判断哪些任务有 cosim 风险，只在必要时花 20 credits。

---

## 8. HLS 常见性能和资源词

### latency

完成一次 kernel 需要多少 clock cycles。

越小越好。

### II

II = Initiation Interval，流水线启动间隔。

```text
II = 1
```

意思是每个 cycle 都能启动一个新的循环迭代。

### PIPELINE

把操作重叠起来执行。

例如原本：

```text
读 → 乘 → 加
读 → 乘 → 加
读 → 乘 → 加
```

流水线后：

```text
cycle 1: 第 1 个读
cycle 2: 第 1 个乘，第 2 个读
cycle 3: 第 1 个加，第 2 个乘，第 3 个读
cycle 4: 第 2 个加，第 3 个乘，第 4 个读
```

### UNROLL

复制多份硬件，让多个循环迭代同时做。

代价是资源增加。

### ARRAY_PARTITION

把数组拆成多个 bank，让多个并行硬件能同时读写。

如果 unroll 了但数组只有一个读口，就会被内存端口卡住。

### DATAFLOW

把函数/循环拆成多个 stage，让它们通过 stream 边生产边消费。

优点是并行度高。

风险是容易出现 stream deadlock。

---

## 9. 常见失败是什么意思

### DSP 不够

DSP 是 FPGA 上专门做乘法、乘加的硬件块。

原因可能是：

- 乘法太多
- unroll 太猛
- bit width 太大

### LUT 爆

LUT 是 FPGA 上的通用逻辑资源。

原因可能是：

- 控制逻辑太复杂
- unroll 导致硬件复制太多
- array partition 后 mux 和连线变多

### routing 很难

硬件连线太复杂，布线工具难以把模块连接好。

常见原因：

- 并行模块太多
- 一个信号 fanout 太大
- memory bank 太多

### clock 过不了

目标是 5 ns，也就是 200 MHz。

如果一条组合逻辑路径太长，就无法在 5 ns 内完成。

### synth 失败

HLS 无法把代码综合成硬件。

可能原因：

- 使用了不可综合 C++ 写法
- 动态内存
- 递归
- 不支持的数据结构
- pragma 冲突
- stream 使用不当

---

## 10. Agent 是什么

在这个比赛里，Agent 可以这样理解：

> Agent = LLM + 工具 + 状态 + 决策规则 + 停止条件。

对应到 harness：

```text
LLM        = llm.py 里的模型调用
工具       = csim / synth / cosim
状态       = 当前代码、最好代码、最好 latency、预算、日志
决策规则   = agent.py 里的 if / while
停止条件   = 正确、预算用完、轮数到、优化不再提升
```

它不是“更会聊天的机器人”，而是一个自动实验员：

```text
LLM 提出代码
工具验证代码
Agent 决定下一步
```

---

## 11. Reference Agent 最小闭环

官方 `agent.py` 做的事情很直接：

```text
1. 先用 csim 检查初始代码
2. 如果不正确，把错误日志给 LLM，让它修
3. 如果任务 requires_cosim，再跑 cosim
4. 正确后跑 synth，拿到 baseline latency
5. 让 LLM 优化 latency
6. 每个候选代码必须重新 csim + synth
7. latency 变小就接受，否则丢弃或停止
8. 如果 structural task 优化过，再 cosim 复查
```

伪代码：

```python
code = starting_kernel

correct, code = reach_correctness(code)
if not correct:
    return code

synth_result = synth(code)
best_latency = synth_result.latency

best = optimize(code, best_latency)

return best
```

最重要的原则：

> 不能相信 LLM 说“我修好了”，必须用工具验证。

---

## 12. Harness 主要文件地图

本地路径：

```text
_external/fpt26-harness/
```

核心文件：

```text
llm4hls/agent.py
```

参考 agent 主体，包含：

- `_reach_correctness()`
- `_optimize()`
- `run()`

```text
llm4hls/harness.py
```

给 agent 暴露三个带预算的工具：

- `server.csim(kernel_code)`
- `server.synth(kernel_code)`
- `server.cosim(kernel_code)`

```text
llm4hls/tools.py
```

真正调用 Vitis HLS 的封装，返回 `ToolResult`。

```text
llm4hls/budget.py
```

预算扣费逻辑。

```text
llm4hls/task.py
```

读取 task package。

```text
llm4hls/scoring.py
```

hidden test + PPA 评分。

```text
scripts/run_poc.py
```

端到端入口。

---

## 13. 为什么先学手写 loop，而不是 LangChain

LangChain、LangGraph、AutoGen、CrewAI 都能做 agent，但这场比赛先不建议从框架入手。

原因：

### 1. 比赛工具固定

核心工具只有：

```text
csim
synth
cosim
```

关键不是“怎么接很多工具”，而是“什么时候花预算调用哪个工具”。

### 2. 成败必须靠工具验证

普通 agent 可能看最终回答质量。

这里必须看：

```text
csim pass?
synth pass?
cosim pass?
latency 降了吗?
hidden test pass?
```

### 3. 框架不懂 HLS

框架不会天然知道：

- `runtime_fail` 多半是功能 bug
- `synth_error` 多半是不可综合
- `cosim timeout` 可能是 stream deadlock
- latency 没变小应该回滚
- cosim 很贵，不能乱跑

### 4. 时间只有一个月

如果一开始上复杂框架，会同时学习：

```text
Agent 本质
框架语法
框架状态管理
框架调试方式
HLS 工具链
比赛规则
```

学习负担太重。

更稳路线：

```text
先吃透 reference agent 的 Python loop
再增强日志解析、预算策略、prompt、优化策略
最后再决定是否引入框架
```

---

## 14. LangChain 那类框架是什么意思

LangChain 可以理解成：

> 把 LLM、prompt、工具、记忆、检索、输出格式、agent loop 串起来的 Python 框架。

不用 LangChain：

```python
response = llm.complete(prompt)
result = server.csim(code)
if result.ok:
    ...
```

用 LangChain：

```text
LLM
PromptTemplate
Tool(csim)
Tool(synth)
Tool(cosim)
AgentExecutor
```

它能让 LLM 选择工具并调用。

但比赛里要小心：

```text
LangChain 能帮你操作工具
但不会自动帮你省 cosim 预算
也不会自动理解 HLS 失败类型
```

所以推荐：

```text
前期：不用 LangChain
中期：可以借用 prompt/template/output parser/tracing
后期：流程复杂后再考虑 LangGraph
```

---

## 15. OpenRouter 和开源模型要求

比赛材料写的是：

```text
LLM provider: OpenRouter, open-source models only
```

意思是：

> 可以通过 OpenRouter API 调模型，但模型本身必须是开源/开放权重模型。

关键区分：

```text
API 服务开放调用 ≠ 模型开源
模型权重/许可证开放 = 比赛更可能合规
```

例如：

```text
Provider: OpenRouter
Model: deepseek-ai/DeepSeek-V4-Pro
License: MIT
```

这种写法比只写 `DeepSeek` 更清楚。

最终提交时要写清楚：

- exact model id / slug
- provider
- license
- token consumption
- 是否使用搜索/外部工具

---

## 16. OpenRouter Fusion 能不能用

结论：

> Fusion 可以当赛前研究助手，但不建议作为正式参赛 agent 模型。

原因：

- 它更像多模型融合/路由产品，不是单一开源权重模型。
- 背后可能混入 proprietary models。
- 内部路由不透明，论文里难复现。
- 比赛更需要稳定生成 HLS C++ 和控制工具预算，web search 不一定有帮助。

建议：

```text
Fusion：用于学习、分析、头脑风暴
正式 agent：用明确开源权重、明确许可证、明确 slug 的模型
```

---

## 17. 模型选择的初步方向

模型排名变化很快，以下只作为 2026-07-07 对话阶段的候选方向，最终提交前必须复查。

优先关注榜单指标：

```text
SWE-bench Verified：代码修复能力
Aider：代码编辑能力
Terminal-Bench：命令行/工具环境能力
tau2 / tool-use 类指标：多步工具调用能力
```

候选模型方向：

- DeepSeek V4 Pro / V4 Flash
- Kimi K2 系列
- GLM-4.5 / GLM-5 系列
- Qwen Coder / Qwen3.x 系列

注意：

> 没有一个公开榜单专门评测 HLS agent，所以必须在 harness 上自己测。

我们要测的不是“模型聊天能力”，而是：

- 修 bug 成功率
- synth 成功率
- cosim/deadlock 修复能力
- latency 改善
- token 消耗
- 格式错误率
- 预算内最终 score

---

## 18. 是否需要真实 FPGA

目前理解：

> 不需要实机 FPGA 板卡来做主要开发。

比赛主要依赖 Vitis HLS：

- C simulation
- C synthesis
- C/RTL co-simulation

这些是软件工具链完成的。

但需要一个能跑 Vitis 2025.2 的环境，通常是：

- Linux
- WSL2 + Linux
- Docker + Vitis
- 实验室服务器
- 远程开发机

当前 Windows 本机环境检查结论：

- Docker 命令存在，但 daemon/权限未确认
- WSL 存在，但没有默认 distro
- Vitis / `vitis-run` 不在 PATH
- 当前不能直接跑官方 HLS loop

所以技术第一优先级是：

> 找到或搭好一个能跑 Vitis 2025.2 的环境。

---

## 19. 如何判断我们做成功了

不是“LLM 输出了代码”就成功。

最小成功标准：

```text
1. 能加载 task
2. agent 能调用 csim/synth/cosim
3. budget 正常扣费
4. 能生成最终 kernel
5. scoring.py 能跑 hidden grading
6. 能输出 scorecard
```

对每个任务：

```text
functional pass
synthesizable pass
latency 有改善
score 提高
budget 没爆
```

---

## 20. 一个月内建议路线

### 第 1 周：环境和 harness

目标：

- 确认报名和规则
- 搭 Vitis 2025.2 环境
- 跑通 reference harness
- 跑 `projection_bugfix` 和 `dotProduct_optimize`
- 读懂 `agent.py` 的主流程

不要急着大改 agent。

### 第 2 周：agent 增强

目标：

- 加强日志分类
- 区分 compile/runtime/synth/cosim/deadlock
- 加强 prompt 模板
- 保存最好版本和失败候选
- 记录 token 和工具调用轨迹

重点：

```text
让 agent 更稳地达到 correctness
```

### 第 3 周：HLS 优化策略

目标：

- 针对常见模式写优化 prompt 和规则
- pipeline / unroll / partition / dataflow
- 学会根据 synth report 判断下一步
- 加 cosim 风险判断

重点：

```text
正确之后再优化 latency
```

### 第 4 周：评测、消融和论文

目标：

- 多模型对比
- 多策略对比
- budget policy 对比
- 写 2 页主文档
- 准备 demo 和答辩材料

论文要能回答：

```text
我们的 agent 流程是什么？
为什么这样控制预算？
如何处理 csim/synth/cosim 反馈？
token 消耗是多少？
相比 reference agent 提升在哪里？
```

---

## 21. 我们后续最值得改强的点

按优先级：

### 1. Failure classifier

自动识别失败类型：

- compile error
- runtime fail
- synth error
- cosim fail
- timeout
- possible deadlock

### 2. Budget policy

决定什么时候调用：

- csim
- synth
- cosim

例如：

```text
先 csim 多试几次
功能正确后再 synth
只有 stream/dataflow 风险高时才 cosim
预算低时停止优化
```

### 3. Prompt templates

不同失败类型用不同 prompt：

```text
修功能 bug prompt
修综合失败 prompt
修 deadlock prompt
优化 latency prompt
降低资源 prompt
```

### 4. Best-code memory

永远保存：

- 最后一次正确代码
- 最低 latency 代码
- cosim verified 代码
- 失败候选和失败原因

### 5. HLS optimization library

总结常见优化动作：

- loop pipeline
- partial/full unroll
- array partition
- local buffer
- bitwidth reduction
- dataflow restructuring
- stream balance

---

## 22. 给 0 基础队友的学习顺序

不要从论文和复杂框架开始。

推荐顺序：

```text
1. 看懂这个 Markdown
2. 看懂 csim / synth / cosim 是什么
3. 看懂 harness 的文件地图
4. 读 scripts/run_poc.py
5. 读 llm4hls/harness.py
6. 读 llm4hls/agent.py 的 run()
7. 读 _reach_correctness()
8. 读 _optimize()
9. 再补 HLS 的 pipeline/unroll/partition
10. 最后再看 LangChain/LangGraph/AutoGen
```

一句话：

> 先学比赛闭环，再学 HLS 优化，再学 agent 框架。

---

## 23. 重要链接

- FPT 2026 Design Competition Track A: <https://fpt2026.uark.edu/fpt26-design-competition/>
- Reference harness: <https://anonymous.4open.science/r/fpt26-harness/README.md>
- DataLearner open-source leaderboard: <https://www.datalearner.com/leaderboards/open-source>
- OpenRouter models API: <https://openrouter.ai/api/v1/models>
- DeepSeek API docs: <https://api-docs.deepseek.com/>
- LangGraph docs: <https://docs.langchain.com/oss/python/langgraph/overview>
- AutoGen docs: <https://microsoft.github.io/autogen/stable/>
- CrewAI docs: <https://docs.crewai.com/>

---

## 24. 最后记住这三句话

第一句：

> Track A 的核心不是写一次 HLS 代码，而是做一个能反复试错、验证、优化的 agent。

第二句：

> csim 便宜但不够，synth 看性能和可综合，cosim 贵但能抓硬件级结构问题。

第三句：

> 一个月内最稳的路线是基于官方 harness 改强 reference agent，而不是一开始追复杂 agent 框架。
