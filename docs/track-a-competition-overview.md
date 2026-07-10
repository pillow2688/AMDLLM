# FPT 2026 Track A 比赛总览

Checked: 2026-07-10
适用对象：第一次接触 Track A、HLS 或 Agent 的队友

## 先说结论

Track A 要提交的不是某一道题的固定 HLS 代码，而是一个能够自动处理多类 HLS 任务的 Agent：

> Agent 读取任务、规格和初始代码，在有限工具预算内生成、修复和优化 HLS C/C++，最终交出一个正确、可综合、PPA 尽可能好的 kernel。

可以把单道题抽象成：

```text
输入：task package + tool budget
输出：final kernel.cpp
```

## 三层信息必须分开

本仓库使用三类依据：

### 官方已经明确的规则

来源是 FPT 2026 Design Competition 官网。官网决定比赛目标、任务范围、评价维度和提交要求。

### 团队收到的补充提交指南

来源是团队于 2026-07-10 收到的英文 `Submission Guidelines for Track-A`。它新增了 U55C、Vitis 2025.2、最低 100 MHz、三种推荐模型、Docker、复现包和 5 分钟视频等要求。该通知目前尚未出现在公开官网，因此仓库同时保留原文翻译、公开核对结果和待确认问题。

### 当前 reference harness 的样例实现

来源是 `fpt26-harness`。它给出可运行的工具接口、样例 credits、样例评分器和参考 Agent，帮助我们理解和实验，但不等于最终正式评测细则。

因此：

- “预算有限、正确性优先、需要自动终止”是官方要求；
- “U55C、Vitis 2025.2、三类工具通过、至少 100 MHz、Docker 和 5 分钟视频”来自补充提交指南；
- `csim=1`、`synth=4`、`cosim=20` 是当前 reference harness 的配置；
- 样例评分公式和 `8x` acceleration cap 不能当作最终官方承诺。

## 官方任务会给什么

官方页面说明，每道任务会包含：

- 初始 C/C++ 或 HLS 源文件；
- public testbench 和构建脚本；
- 功能规格、接口、数据类型和数值误差要求；
- FPGA、HLS 工具版本、时钟和可选资源约束；
- 各工具最大调用次数，或统一 credit budget。

初始代码可能：

- 功能正确但性能很差；
- 编译或综合失败；
- `csim`、`cosim` 或 hidden test 失败；
- 存在 deadlock、无效 streaming 或严重资源浪费；
- 存在其他 HLS 编译问题。

所以参赛 Agent 不能只会“给正确代码加 pragma”，还必须能修功能错误、综合错误和结构性硬件问题。

## Agent 的完整闭环

```text
读取任务规格和初始 kernel
  ↓
分析当前阶段和失败类型
  ↓
生成或修改 HLS C/C++
  ↓
选择 csim / synth / cosim
  ↓
解析 ToolResult、日志和报告
  ↓
接受候选 / 回滚 / 反思 / 换策略
  ↓
判断继续还是停止
  ↓
返回 best_verified_code
```

官网要求成功方案展示：任务理解、代码生成或修改、工具调用、日志诊断、正确性优先、PPA 优化，以及在预算内终止。

## 三种工具

下表中的费用来自当前 reference harness，而不是官网固定价格。

| 工具 | 样例费用 | 主要回答的问题 | 不能替代什么 |
| --- | ---: | --- | --- |
| `csim` | 1 | C/C++ 在 public testbench 上是否功能正确 | 看不到真实 RTL deadlock，也不给出综合 PPA |
| `synth` | 4 | 能否综合，latency、II、时钟与资源如何 | 不等于功能验证，也不证明 RTL 行为一定正确 |
| `cosim` | 20 | C 与 RTL 是否一致，是否存在 stream/dataflow deadlock | 不适合频繁用于普通功能调试或搜索 PPA |

调用工具的目的不是“多跑几个测试更放心”，而是用最小成本获得足以改变下一步决策的信息。

## 当前目标平台和验收门槛

补充提交指南给出的当前要求是：

```text
FPGA:       AMD Alveo U55C
Software:   Vitis 2025.2
Validation: csim + synth + cosim 全部通过
Frequency:  >= 100 MHz，也就是 period <= 10 ns
Packaging:  Docker 环境 + 可复现源码/testbench/补充材料
```

reference harness 当前默认 part 是 `xcu55c-fsvh2892-2L-e`，样例目标为 200 MHz，也就是 5 ns。它比通知的最低 100 MHz 更严格，但正式任务仍应优先读取 task package 的目标约束。

## 评测如何看最终代码

官网目前明确的主要评价维度是：

- correctness；
- PPA；
- task difficulty。

补充提交指南还明确 token consumption 是最终评价的重要因素，但精确权重和统计口径尚未公布。

当前 reference scorer 会在 Agent 结束后、计费预算之外：

1. 用 hidden testbench 检查最终候选；
2. 对需要 RTL 验证的任务执行 hidden cosim；
3. 综合候选代码和原始 baseline；
4. 根据正确性、可综合性和 latency acceleration 生成 scorecard。

在该样例实现里，hidden functional test 失败会使分数为 0；正确后才计算综合与 PPA 收益。这个“正确性门槛”与官网强调的 correctness first 一致，但具体权重和 acceleration cap 仍应视为样例。

## Credits 为什么是核心

credits 不直接代表得分，却决定 Agent 能做多少实验。每次调用都有机会成本：

```text
一次 cosim = 20 次 csim
          = 5 次 synth
          = 4 轮 csim + synth 候选验证
```

以 reference harness 中预算为 80 的结构任务为例：

```text
初始 csim + cosim             21
修复后 csim + cosim           21
正确版本 baseline synth        4
预留最终 cosim                20
--------------------------------
可用于其他优化的预算           14
```

这说明真正的问题不是单独的“什么时候停止”，而是：

> 如何在修复、诊断、性能测量和最终验证之间动态分配预算，并在继续探索的预期价值不再高于机会成本时停止。

无论如何探索，Agent 都应始终保存 `best_verified_code`。预算耗尽、超时或新候选失败时，返回这个版本，而不是返回最后生成但未验证的代码。

## 什么由比赛提供，什么由我们开发

### 比赛或 harness 提供

- task package；
- public testbench；
- 工具调用接口；
- tool budget；
- hidden grader 和最终评价环境。

### 我们开发

- LLM 选择和调用层；
- task/failure classifier；
- HLS 代码生成与修改策略；
- 日志和报告解析；
- Tool-Grounded Reflection；
- 候选版本比较、接受和回滚；
- 预算控制、工具选择和最优停止；
- token、工具轨迹和实验记录。

最终材料还要包含 Docker 复现环境、实验报告、源码/testbench/补充材料和最长 5 分钟的实际运行视频。

底层 `csim/synth/cosim` 由 Vitis 和评测接口执行。我们可以增强封装、解析和调用策略，但不能靠改测试或绕过评测获得分数。

## 推荐的 Agent 形态

Track A 适合“确定性外层 + 动态智能内层”：

```text
LLM：分析、提出修改、解释假设、建议下一动作
规则层：预算、重复调用、最终验证预留、状态不变量
工具层：执行 csim/synth/cosim，返回真实证据
状态层：保存 best、verified 状态、失败历史和剩余预算
```

模型可以决定策略，但昂贵调用必须先经过确定性预算检查。Reflection 也应基于真实工具结果，而不是让模型凭感觉给自己打分。

## 提交与时间

官网当前要求：

- IEEE 双栏 PDF；
- 正文不超过 2 页，附录不限；
- Track A 论文说明 Agent workflow、运行过程和各阶段 token consumption；
- 入围后 10 分钟展示与 live demo，随后 5 分钟问答；
- 原则上线下参加，远程 demo 不具备获奖资格。

团队收到的补充提交指南进一步要求：

- 用 Vitis 2025.2 面向 Alveo U55C 完成验证；
- 最终候选通过 `csim`、`synth`、`cosim`，频率至少 100 MHz；
- 提供 experimental report；
- 推荐报告 DeepSeek V4 Pro、Qwen3.5 122B A10B AWQ-4bit 和 Qwen3.6 27B FP8 的对比结果；
- 项目在 Docker 中可构建、可运行，并建议附 Dockerfile；
- 提交源码、testbench 和复现材料；
- 提交最长 5 分钟的 target-platform demonstration video。

关键日期均为 23:59 AoE：

| 事项 | AoE 日期 | 北京时间约为 |
| --- | --- | --- |
| 注册截止 | 2026-07-07 | 2026-07-08 19:59，已过 |
| 技术材料截止 | 2026-08-07 | 2026-08-08 19:59 |
| 入围公布 | 2026-08-21 | 2026-08-22 19:59 |

截至 2026-07-10，官网仍表示 submission portal 和 FAQ 后续公布。补充指南已经增加技术与材料要求，但 `.zip` 结构、Docker/Vitis 接入方式、API 模型是否允许、token 评分和视频上传方式仍需组织者确认。

## 当前一句话目标

> 在每一道未知 HLS 任务上，用有限 credits、token 和时间，始终保住已验证的最好版本，并让每一次昂贵工具调用尽可能带来有效信息或最终得分提升。

## 依据与延伸阅读

- FPT 2026 Design Competition：<https://fpt2026.uark.edu/fpt26-design-competition/>
- Reference harness：<https://anonymous.4open.science/r/fpt26-harness/README.md>
- [官方规则快照](../materials/01_official/2026-07-10-track-a-official-summary.md)
- [Track-A Submission Guidelines 中文翻译与执行清单](../materials/01_official/2026-07-10-track-a-submission-guidelines-zh.md)
- [Reference harness 分析](../materials/02_harness/2026-07-10-reference-harness-analysis.md)
- [预算感知工具策略与最优停止](../materials/04_agent_basics/2026-07-10-budget-aware-tool-policy-and-optimal-stopping.md)
