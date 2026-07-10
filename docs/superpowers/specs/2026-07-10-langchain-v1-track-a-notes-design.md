# LangChain v1 Track A 资料整理设计

Status: approved
Owner: team
Checked: 2026-07-10

## 目标

把 LangChain v1 发布说明转换为适合 FPT 2026 Track A 团队使用的学习资料，回答三个问题：

1. LangChain v1 的变化分别是什么；
2. 哪些变化能直接帮助参赛 Agent；
3. LangChain `create_agent`、middleware、LangGraph 和 reference harness 应如何分工。

## 信息边界

- 中文页面用于降低阅读门槛；
- LangChain 官方英文文档用于确认当前 API 和框架定位；
- 团队笔记明确区分官方事实、比赛映射和团队建议；
- 不把页面示例直接复制成参赛实现；
- 本次不安装 LangChain，不修改 harness，不承诺采用某个框架。

## 内容结构

新增一篇独立专题，包含：

- v1 的三个核心变化；
- `create_agent` 的最小 Agent loop；
- middleware hook 到 Track A 职责的映射；
- structured output、standard content blocks 和简化 namespace 的优先级；
- `create_agent` 与 LangGraph 的分层方式；
- 对中文版示例时效性的提醒；
- 一个月赛程下的学习和验证顺序。

现有 LangGraph 架构笔记只补充框架分层和 middleware 放置建议，避免重复整篇 v1 发布说明。

## 文件范围

- 新增 `materials/04_agent_basics/2026-07-10-langchain-v1-track-a-mapping.md`
- 修改 `materials/04_agent_basics/2026-07-10-langgraph-track-a-architecture.md`
- 修改 `materials/04_agent_basics/README.md`
- 修改 `materials/00_index/README.md`
- 修改根目录 `README.md`

## 成功标准

- 初学者能解释 LangChain 和 LangGraph 不是二选一；
- 明确 credits、安全和最终验证不能只靠 prompt；
- 中文页面与官方页面都被记录，且官方页面是 API 权威来源；
- 所有新增相对链接可解析；
- 不引入代码、依赖、密钥或外部仓库内容。
