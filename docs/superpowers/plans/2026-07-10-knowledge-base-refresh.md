# LLM4HLS 资料库整理实现计划

> **面向 AI 代理的工作者：** 在当前会话中按任务顺序执行，完成每项后更新进度；最终使用 verification-before-completion 进行验证。

**目标：** 更新团队资料库，使比赛规则、reference harness、Agent 架构和预算策略准确、分层、可维护，并推送到 GitHub。

**架构：** 保留现有 `docs/` 成品文档与 `materials/` 过程资料边界。新增一份比赛总览和若干带日期的专题笔记，只对零基础指南进行针对性校准，避免全量重写。

**技术栈：** Markdown、Git、PowerShell 链接检查、GitHub HTTPS remote。

---

### 任务 1：补齐权威说明与 harness 专题

**文件：**

- 创建：`docs/track-a-competition-overview.md`
- 创建：`materials/01_official/2026-07-10-track-a-official-summary.md`
- 创建：`materials/02_harness/2026-07-10-reference-harness-analysis.md`

- [x] 写清官方输入、工作流、评价维度、提交物和日期。
- [x] 明确官网规则与 reference harness 实现的边界。
- [x] 说明 task、ToolServer、Budget、工具结果和隐藏评分的数据流。

### 任务 2：补齐 Agent、模型和环境专题

**文件：**

- 创建：`materials/04_agent_basics/2026-07-10-langgraph-track-a-architecture.md`
- 创建：`materials/04_agent_basics/2026-07-10-budget-aware-tool-policy-and-optimal-stopping.md`
- 创建：`materials/05_models_and_apis/2026-07-10-open-model-selection-snapshot.md`
- 创建：`materials/06_environment/2026-07-10-local-environment-snapshot.md`

- [x] 记录 LangChain 与 LangGraph 当前官方分工。
- [x] 定义模型提议、规则校验、工具执行、Reflection、回滚和停止闭环。
- [x] 把模型与本机环境信息标记为有日期的快照。

### 任务 3：校准零基础指南

**文件：**

- 修改：`docs/track-a-zero-foundation-guide.md`
- 修改：`materials/04_agent_basics/2026-07-09-datawhale-hello-agents-to-llm4hls-mapping.md`

- [x] 增加资料时效和官方/参考边界说明。
- [x] 修正 LangChain/LangGraph 章节。
- [x] 强化 `best_verified_code`、预算预留和最优停止。
- [x] 将模型与环境详情改成专题链接。

### 任务 4：更新导航

**文件：**

- 修改：`README.md`
- 修改：`docs/README.md`
- 修改：`materials/README.md`
- 修改：`materials/00_index/README.md`
- 修改：`materials/01_official/README.md`
- 修改：`materials/02_harness/README.md`
- 修改：`materials/04_agent_basics/README.md`
- 修改：`materials/05_models_and_apis/README.md`
- 修改：`materials/06_environment/README.md`

- [x] 把比赛总览设为新队友第一入口。
- [x] 在各专题目录列出当前笔记。
- [x] 统一 Hello-Agents canonical GitHub URL。

### 任务 5：验证并发布

- [x] 用 `rg` 检查 TODO、旧链接和过强表述。
- [x] 用 PowerShell 检查所有本地 Markdown 相对链接是否存在。
- [x] 检查 `git diff --check`、`git diff --stat` 和工作区状态。
- [ ] 提交为一个聚焦的文档更新 commit。
- [ ] 推送 `main` 到 `origin`。
