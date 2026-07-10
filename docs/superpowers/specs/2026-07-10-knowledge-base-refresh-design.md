# LLM4HLS 资料库整理设计

日期：2026-07-10<br>
状态：已批准<br>
范围：比赛说明、参考 harness、Agent 架构、预算策略、模型与环境快照、仓库导航

## 目标

把当前对话中形成的比赛理解同步到团队 Git 仓库，同时修正现有资料中容易误导或快速过期的内容。

完成后，新队友应当能够明确区分：

- 官方已经明确的比赛规则；
- reference harness 当前采用的样例实现；
- 团队自己的架构判断和待验证假设。

## 现状问题

现有目录结构和模板质量良好，但核心入门指南承担了过多职责：

- 稳定的 HLS/Agent 基础与易变化的模型、环境信息混在一起；
- 个别段落把 reference harness 的 credits 和评分公式说得过于接近正式规则；
- LangChain/LangGraph 判断未反映当前官方文档关系；
- `01_official/` 和 `02_harness/` 缺少真正的内容笔记；
- 预算分配、候选回滚和最优停止尚未成为独立专题。

## 内容分层

### 成品文档

`docs/track-a-competition-overview.md` 作为面向全队的比赛总览，讲清输入、Agent 闭环、工具、评分、预算和提交物。

`docs/track-a-zero-foundation-guide.md` 保留零基础解释，但将易变化的信息改成摘要和专题链接，并修正官方/参考边界。

### 证据和专题笔记

- `materials/01_official/`：带检查日期的官方规则快照；
- `materials/02_harness/`：reference harness 文件、工具和评分实现分析；
- `materials/04_agent_basics/`：LangGraph 混合架构，以及预算感知工具策略和最优停止；
- `materials/05_models_and_apis/`：带日期的模型选择快照，不在长期指南里固化排行榜；
- `materials/06_environment/`：当前机器环境快照，不把设备状态写成长期事实。

## 准确性原则

1. 官网明确的信息标记为“官方规则”。
2. `fpt26-harness` 中的 `1/4/20`、样例预算、评分公式标记为“当前 reference harness”。
3. 模型、provider、license 和环境信息必须带检查日期。
4. 未经最终 FAQ 确认的内容不写成正式比赛承诺。
5. 始终保存并返回 `best_verified_code`，作为预算策略的核心不变量。

## 非目标

- 不彻底拆分整份零基础指南；
- 不实现 Agent 代码；
- 不提交本地 `_external/` harness；
- 不把尚未实测的模型结论写成推荐排名。

## 验证标准

- 所有新增和修改的相对 Markdown 链接均指向存在的文件；
- 核心入口能从根 README 在两次点击内到达；
- 文档中不存在把样例 credits/评分误称为最终规则的表述；
- Git diff 只包含本次资料整理；
- 提交成功推送到 `origin/main`。
