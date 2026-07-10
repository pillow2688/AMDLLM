# LangChain v1 Track A 资料整理实施计划

> **面向 AI 代理的工作者：** 使用 `superpowers-zh:executing-plans` 在当前会话逐项执行。

**目标：** 新增一篇经官方资料核对、面向 Track A 的 LangChain v1 映射笔记，并把它接入现有资料导航。

**架构：** 中文页面作为辅助阅读入口，官方文档作为事实和 API 依据。新增专题负责解释 v1，现有 LangGraph 笔记负责表达团队架构，README 只承担导航职责。

**技术栈：** Markdown、PowerShell、Git

---

### 任务 1：建立专题笔记

**文件：**

- 创建：`materials/04_agent_basics/2026-07-10-langchain-v1-track-a-mapping.md`

- [x] 写明来源、检查日期和时效风险
- [x] 总结 `create_agent`、middleware、structured output、content blocks、namespace
- [x] 把 middleware hooks 映射到 Track A 的预算和工具治理
- [x] 给出推荐分层与一个月内学习优先级
- [x] 标注中文版示例不能替代当前官方 API

### 任务 2：消除现有资料歧义

**文件：**

- 修改：`materials/04_agent_basics/2026-07-10-langgraph-track-a-architecture.md`

- [x] 补充 `create_agent` 是 LangGraph 上层 Agent harness
- [x] 区分 middleware 局部拦截与外层 StateGraph 全局状态管理
- [x] 保留确定性 budget gate 和 best-code invariant

### 任务 3：接入导航

**文件：**

- 修改：`materials/04_agent_basics/README.md`
- 修改：`materials/00_index/README.md`
- 修改：`README.md`

- [x] 增加专题链接
- [x] 调整 Agent 资料推荐阅读顺序
- [x] 保持根目录入口简洁

### 任务 4：验证并发布

**文件：**

- 检查：全部受影响的 Markdown 文件

- [x] 检查相对链接
- [x] 检查单一 H1、代码围栏和禁止占位符
- [x] 运行 `git diff --check`
- [x] 检查暂存区不含 `_external`、密钥或运行产物
- [ ] 提交并推送到 `origin/main`
