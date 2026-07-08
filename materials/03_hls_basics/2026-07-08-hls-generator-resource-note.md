# hls-generator 资源笔记

Status: summarized  
Owner: team  
Checked: 2026-07-08  
Source: https://github.com/Eriemon/hls-generator  
Expires/Risk: medium

## Why This Matters

`Eriemon/hls-generator` 是一个面向 AI coding agent 的 Vitis HLS 技能/资料仓库。它不是 FPT Track A 官方 harness，也不能直接替代比赛系统，但很适合帮助我们补 HLS 工程知识、优化模式和验证意识。

一句话：

> fpt26-harness 是比赛主循环和评测接口；hls-generator 更像 HLS 知识库、prompt 规则和工程规范参考。

## What It Is

这个仓库定位是 Codex-ready HLS agent skill。它提供：

- HLS 生成和修改流程规则
- Vitis HLS 验证意识
- HLS 优化模式参考
- 示例 specs 和 HLS 工程模式
- runtime/helper 相关思路

它强调：

```text
先确认接口和需求
先得到能过 csim 的 baseline
再根据 synth report 做优化
不要乱堆 pragma
没有真实运行 Vitis 就不要声称验证通过
```

## What We Can Borrow

### 1. HLS 优化知识

重点看它的 references：

- `hls-optimization-patterns.md`
- `hls-report-driven-optimization.md`
- `hls-memory-burst-and-layout.md`
- `hls-task-parallel-strategy.md`

这些可以帮助我们建立：

- pipeline / unroll / partition 的使用规则
- 如何读 synth report
- 什么时候是 memory bandwidth 问题
- 什么时候是 dataflow/task parallel 问题
- prompt 里该怎么约束 HLS 优化

### 2. Agent Prompt 设计

它的思路适合迁移到 Track A：

```text
固定接口
固定 top function signature
先正确，再优化
输出完整 kernel
每次修改后必须验证
```

这些正好对应 `fpt26-harness` 中的 reference agent。

### 3. 示例任务/模式

它的 examples 里包含：

- matmul
- FIR
- FFT
- stencil
- dataflow
- AXIS
- array partition
- multi m_axi

这些适合 0 基础队友学习 HLS 常见结构，不建议一开始直接搬到比赛 agent。

### 4. 验证纪律

它强调：没有真正跑 `vitis-run` / `vitis_hls`，就不能声称 HLS 验证通过。

这和 Track A 的核心一致：

```text
不能相信 LLM 说“我修好了”
必须用 csim / synth / cosim 验证
```

## What It Cannot Replace

它不能替代：

- `fpt26-harness`
- Track A 的 budget 机制
- hidden grading
- `ToolServer`
- `scoring.py`
- 比赛任务格式

所以不要把路线搞反。

正确关系是：

```text
主线：
  fpt26-harness → 改强 agent.py → 用 scoring 评估

辅助：
  hls-generator → 借鉴 HLS 知识、优化模式、prompt 约束
```

## Mapping To Track A

| hls-generator 价值 | Track A 对应落点 |
| --- | --- |
| HLS 需求确认 | 读 task description / header / testbench |
| 优化模式 | 优化 prompt 和 HLS action library |
| report-driven optimization | 根据 synth report 决定下一步 |
| 验证纪律 | csim/synth/cosim gate |
| 示例 specs | 训练队友理解 HLS 结构 |
| 不乱堆 pragma | 避免 synth/resource/routing 崩掉 |

## Recommended Use

当前阶段建议这样用：

```text
第 1 步：队友读这个笔记，知道它不是官方 harness
第 2 步：读 hls-optimization-patterns.md
第 3 步：读 hls-report-driven-optimization.md
第 4 步：挑 1-2 个 examples 理解 HLS 写法
第 5 步：把有用规则整理成我们的 prompt templates
```

暂时不要：

```text
不要直接基于它重写比赛系统
不要把它当作评分 harness
不要一上来搬复杂 examples
```

## Open Questions

- 哪些 hls-generator 规则适合直接写进 repair prompt？
- 哪些规则适合写成 synth report classifier？
- 是否需要把它的 examples 转成我们自己的小型学习任务？
