# FPT26 Reference Harness 分析

Status: verified<br>
Owner: team<br>
Checked: 2026-07-10<br>
Source: https://anonymous.4open.science/r/fpt26-harness/README.md<br>
Local copy: `_external/fpt26-harness/`<br>
Expires/Risk: medium，正式评测接口可能更新

## 它是什么

该仓库是 Track A 的最小 reference agent 与 evaluation harness。README 明确说明它不是参赛提交，参赛者可以实现自己的 Agent 和 harness，只要满足比赛要求。

它最有价值的地方不是参考 Agent 本身，而是给出了一个可运行的端到端边界：

```text
task package
  → metered ToolServer
  → iterative Agent
  → final kernel
  → hidden grading and scorecard
```

## 当前固定的开发目标

reference harness 当前配置：

- AMD Vitis 2025.2；
- `vitis-run --mode hls`；
- Alveo U55C，part 为 `xcu55c-fsvh2892-2L-e`；
- 200 MHz，clock period 为 5 ns；
- LLM backend 支持 OpenRouter 与离线 scripted client。

这些配置应作为当前开发基线，但仍需等待正式 FAQ 确认最终环境是否完全一致。

## Task package

每个样例任务包含：

```text
task.toml
description.md
kernel.cpp
kernel.h
public testbench
hidden testbench
reference solution
```

`task.toml` 记录 top function、task type、difficulty、budget、part、clock 和是否需要 cosim。

Agent 只返回 kernel source。header 与 testbench 由 harness 组装，避免 Agent 修改接口或测试逻辑。

## ToolServer

`llm4hls/harness.py` 暴露三个计费接口：

```python
server.csim(kernel_code)
server.synth(kernel_code)
server.cosim(kernel_code)
```

每次调用先由 `Budget.charge()` 扣费，再运行工具，最后记录 transcript。当前费用为：

| 工具 | Credits | 典型作用 |
| --- | ---: | --- |
| `csim` | 1 | 编译并运行 public C testbench |
| `synth` | 4 | C synthesis，并解析 latency、II 与资源报告 |
| `cosim` | 20 | C/RTL co-simulation，检查 mismatch、deadlock 和 timeout |

如果余额不足，`BudgetExceeded` 会让参考 Agent 硬停止。

## ToolResult

工具结果包含结构化 phase，例如：

- `pass`；
- `compile_error`；
- `runtime_fail`；
- `synth_error`；
- `cosim_fail`；
- `timeout`。

此外还可能包含原始日志尾部、synthesis report 和 cosim report。这些字段正是 failure classifier 与 Tool-Grounded Reflection 的主要输入。

## 隐藏评分

`llm4hls/scoring.py` 在 Agent 计费预算之外执行：

1. hidden C test；
2. 对 `requires_cosim=true` 的任务执行 hidden cosim；
3. 综合最终候选；
4. 综合原始 baseline；
5. 比较 latency acceleration 并生成 scorecard。

当前样例公式：

```text
hidden functional fail → score = 0

otherwise:
  acceleration = baseline_latency / candidate_latency
  ppa_norm = min(acceleration, 8) / 8
  score = difficulty × (0.5 + 0.2 × synth_pass + 0.3 × ppa_norm)
```

资源使用会显示在 scorecard 中，但当前公式的 PPA 部分主要使用 latency acceleration。官网只承诺 correctness、PPA 和 difficulty，因此不能据此忽略资源与时序。

## 三个样例任务

| Task | 类型 | Budget | 主要问题 |
| --- | --- | ---: | --- |
| `projection_bugfix` | repair | 20 | 功能错误，public csim 失败 |
| `dotProduct_optimize` | optimize | 40 | 正确但未优化 |
| `residual_stream_deadlock` | structural | 80 | csim 通过，但 RTL bounded FIFO 在 cosim 中 deadlock |

第三个任务说明 `csim` 无法替代 `cosim`：C simulation 的 stream 行为可能没有真实 RTL FIFO 深度限制。

## ReferenceAgent 的策略

当前参考流程：

1. 用 csim 建立初始正确性状态；
2. 错误时把日志交给 LLM 修复；
3. structural task 在正确性阶段同时 cosim；
4. 正确后 synth，记录 baseline latency；
5. 每轮优化后执行 csim 与 synth；
6. latency 下降才接受；
7. 第一次无提升、达到轮数或预算上限时停止；
8. structural design 优化后，余额允许时再次 cosim，失败则回滚。

## ReferenceAgent 的局限

- 第一次无提升就停止，可能过早放弃其他优化方向；
- prompt 只强调降低 latency，未显式建模资源、时钟和 hidden-test 风险；
- 对昂贵 cosim 只有简单的 `can_afford` 判断，没有动态价值评估；
- 如果 structural task 优化后余额不足 20，最终候选可能没有重新 cosim；
- 失败日志只是截取尾部，没有完整的结构化诊断；
- 没有显式记录已尝试的优化假设与失败原因；
- 没有比较“继续探索”与“立即返回 best”的预期价值。

这些局限正是我们的主要改进空间。

## 对团队的直接结论

1. 不重写 Vitis 工具；先复用 ToolServer 边界。
2. 永远维护 `best_verified_code` 与 verification level。
3. structural task 从一开始就预留最终 cosim credits。
4. 每次修改都要记录 hypothesis、tool evidence、accept/reject reason。
5. 先在三个样例任务建立可重复 baseline，再扩展策略。
6. 所有针对样例评分器的优化，都要标记为 reference-only，避免过拟合。
