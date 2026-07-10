# FPT 2026 Track A 官方规则快照

Status: verified<br>
Owner: team<br>
Checked: 2026-07-10<br>
Source: https://fpt2026.uark.edu/fpt26-design-competition/<br>
Expires/Risk: high，官网 FAQ 与提交入口尚未发布

## 文档边界

这份笔记只记录 FPT 2026 Design Competition 官网已经明确的内容，不把 reference harness 的样例费用、样例任务和评分公式写成正式规则。

## Track A 定位

Track A 名称为 **Budgeted End-to-End LLM4HLS Agent**。

参赛者需要开发自主 Agent，在受限工具调用预算下处理多类 HLS 任务。目标是获得功能正确且 PPA 质量高的 HLS 实现。

## 可能的初始状态

- 功能正确但未优化的 baseline C/C++；
- 编译或综合失败的 HLS 设计；
- 编译成功但 C simulation、co-simulation 或 hidden test 失败；
- deadlock、无效 streaming 或严重资源低效；
- 其他 HLS 编译相关问题。

## 成功方案需要展示的闭环

1. 理解任务规格与初始代码；
2. 生成或修改 HLS C/C++，包括 pragma；
3. 调用工具反馈接口；
4. 解析日志和报告并诊断问题；
5. 优先解决正确性，再优化 PPA；
6. 在分配的预算内终止。

## 每道任务提供的材料

- baseline C/C++ 源文件；
- public testbench 和构建脚本；
- 接口、数据类型、数值误差和设计约束；
- FPGA、HLS 工具版本、时钟与可选资源限制；
- `csim/cosim/synth` 最大调用次数，或统一 credit budget。

## 官方评价维度

- correctness；
- PPA metrics；
- problem difficulty。

官网尚未公布 Track A 最终的精确评分公式。因此，reference harness 中的权重、credits 和 acceleration cap 只能用于开发实验。

## 参赛和平台规则

- 全球高校、研究机构和企业研发团队均可参加；
- 每队 1 至 4 名成员，最多 2 名指导教师；
- 每人只能参加一个队；
- 只接受 AMD 支持的 FPGA/AIE 平台；
- 允许 AMD 或开源工具链；
- 必须是原创工作并包含新贡献。

## 提交要求

- IEEE conference 双栏 PDF；
- 主体内容不超过 2 页；
- appendix 不限页数；
- Track A 必须分析 workflow、Agent 运行过程和各阶段 token consumption；
- preliminary stage 对技术文档和补充材料做双盲评审。

## Final stage

- 15 分钟总时长；
- 10 分钟口头展示和 live demo；
- 5 分钟问答；
- 原则上需要 Full Registration 并线下参加；
- 特殊情况可申请线上，但 remote demo 不具备获奖资格。

## 关键日期

官网所有日期均为 23:59 AoE。北京时间比 AoE 快 20 小时。

| 事项 | AoE | 北京时间约为 |
| --- | --- | --- |
| 注册截止 | 2026-07-07 | 2026-07-08 19:59，已过 |
| 技术材料截止 | 2026-08-07 | 2026-08-08 19:59 |
| 入围公布 | 2026-08-21 | 2026-08-22 19:59 |

## 尚未确认

截至本次检查，以下信息仍应等待 FAQ、submission portal 或组织者通知：

- 正式隐藏任务数量与分布；
- 最终每类工具调用上限或统一 credit 数值；
- Track A 精确评分公式；
- 最终提交代码和运行环境的打包格式；
- 是否会锁定 reference harness 的全部版本与接口；
- 模型合规要求的最终证明材料形式。

官网联系人：`erie@seu.edu.cn`、`icfpt2026@gmail.com`。
