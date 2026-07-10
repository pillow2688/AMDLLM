# 预算感知工具策略与最优停止

Status: team design<br>
Owner: team<br>
Checked: 2026-07-10<br>
Evidence: FPT 2026 Track A official page + current reference harness<br>
Expires/Risk: medium，正式预算与评分细则可能变化

## 核心问题

Track A 可以写成一个受约束的序贯决策问题：

```text
maximize expected final score(final_code)

subject to:
  sum(tool_cost) <= budget
  token_usage <= token_limit
  elapsed_time <= time_limit
```

credits 不是直接扣分项，但它们有限。调用一个低价值工具会减少未来修复、测量或验证的机会，因此每次调用都有机会成本。

## Agent 每轮真正选择什么

可选动作不只是“继续”或“停止”：

```text
修改代码
调用 csim
调用 synth
调用 cosim
接受候选
拒绝并回滚
换一个优化假设
返回 best_verified_code
```

停止是动作集合中的一个动作，而不是预算异常后的被动结果。

## 必须维护的状态

- `remaining_budget`；
- 每种工具的费用或剩余次数；
- `best_verified_code`；
- best 的 latency、资源和 verification level；
- 当前 candidate 及其代码 hash；
- 已运行工具与结果；
- 已尝试的优化假设；
- 连续无改善次数；
- structural/cosim risk；
- token 与时间余量。

## 第一原则：正确性门槛

如果还没有正确候选，PPA 优化没有意义。参考 scorer 中 hidden functional fail 直接为 0，官网也要求 correctness before PPA。

预算分配优先级应为：

```text
获得至少一个正确候选
  > 证明它可综合
  > 保留必要的最终验证预算
  > 使用剩余预算优化 PPA
```

## 第二原则：永远保存 best

`best_verified_code` 只能被验证等级足够、且目标指标更好的候选替换。

建议接受规则：

```text
candidate csim fail        → reject
candidate synth fail       → reject
required cosim fail        → reject
latency/resource/timing 更差 → reject 或保留为备选
指标改善且验证充分          → accept as new best
```

LLM 生成的最后一份代码不等于最终代码。发生预算耗尽、超时或异常时，应返回 best。

## 第三原则：先预留，再探索

定义：

```text
spendable_budget = remaining_budget - validation_reserve
```

`validation_reserve` 取决于任务和当前 verification level：

- 普通任务：候选必须在接受前完成 `csim + synth`，通常无需额外重复；
- structural task：优化后可能重新引入 deadlock，应预留最终 cosim；
- 尚未综合的正确候选：至少预留一次 synth；
- 正式规则若按工具次数限制，则分别预留对应调用次数。

任何新动作如果使 `spendable_budget < 0`，默认拒绝，除非它本身就是最终验证。

## 工具选择的价值

### csim

适合：

- 新候选的第一道过滤；
- 功能修复迭代；
- 语法和 public testbench 快速反馈。

不要把 csim pass 当成结构任务最终正确。

### synth

适合：

- csim pass 后测量 latency、II、时钟和资源；
- 修复 synthesis error；
- 判断某项 HLS 优化是否真实生效。

不要对明显功能错误或完全相同代码重复 synth。

### cosim

适合：

- `requires_cosim=true`；
- 使用 stream、DATAFLOW、bounded FIFO 或有 RTL/C mismatch 风险；
- structural candidate 的最终确认。

一次 cosim 在当前样例中等价于 20 次 csim 或 5 次 synth，因此调用前必须有具体风险证据。

## 硬停止条件

满足任何一项就停止探索并返回 best：

- 没有足够预算完成下一个“修改 + 必要验证”闭环；
- 继续会侵占最终验证预留；
- token 或时间即将达到上限；
- 已达到当前已知评分收益上限；
- 模型无法提出合法且不同的新动作；
- 所有可用工具均已达到正式次数限制。

如果尚无正确候选，停止意味着“无法继续修复”，而不是正常进入 PPA 提交阶段。

## 软停止条件

可在满足下列多个信号时停止：

- 连续 2 至 3 个不同假设没有改善；
- 模型重复相同 pragma、相同代码或同一失败原因；
- synth 报告瓶颈未变，且没有新的报告驱动假设；
- 预计收益很小，但 hidden correctness、资源或 timing 风险明显上升；
- 剩余预算更适合保留给一次高价值验证。

参考 Agent 第一次无改善就停止，适合作为 baseline，但可能过早。我们的策略应允许有限的“耐心”，同时要求假设具有多样性。

## LLM 与规则层的分工

LLM 可以看到：

- 剩余预算；
- 所有工具结果；
- best 与 candidate 差异；
- 历史失败假设；
- 剩余 token 和时间。

LLM 输出下一动作及理由。规则层只做硬校验：

```text
LLM proposes action
  ↓
cost and reserve check
  ↓
duplicate and verification check
  ↓
allow tool call OR reject with reason
  ↓
reason returned to LLM
```

这保留了 Agent 的灵活性，同时避免概率模型破坏预算不变量。

## 当前样例预算的直观含义

### Budget 20：repair 样例

应优先用便宜 csim 修正确性，再 synth。通常不应在无 structural 证据时花光预算跑 cosim。

### Budget 40：optimize 样例

一次初始 csim 与 baseline synth 约花 5 credits；每个完整候选约花 5 credits。最大轮数仍应由优化假设质量而不是简单除法决定。

### Budget 80：structural 样例

两轮 `csim + cosim` 已花 42 credits，baseline synth 后为 46。如果预留 20 做最终 cosim，只剩 14，完整 `csim + synth` 优化最多约两轮。

## 如何做实验

至少比较：

1. reference stop-on-first-no-improvement；
2. 固定轮数 workflow；
3. LLM 完全自由选择工具；
4. 我们的 model-proposal + deterministic-gate 策略。

记录：

- hidden/public correctness rate；
- synth/cosim pass rate；
- final latency 和资源；
- 最终 score；
- credits 构成；
- token 与时间；
- 无效重复调用数；
- 最后代码是否等于 best verified；
- stop reason。

这组消融可以证明提升来自预算策略，而不只是换了更强模型。
