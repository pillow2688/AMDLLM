# LLM4HLS Track A Team Knowledge Base

这个仓库用于团队共享 FPT 2026 Track A / LLM4HLS 的学习资料、阅读笔记、环境说明和实验记录。

## Start Here

新队友先读：

1. [Track A 0 基础入门手册](docs/track-a-zero-foundation-guide.md)
2. [资料库目录规则](materials/README.md)
3. [Agent 基础资料](materials/04_agent_basics/README.md)

## Repository Layout

```text
docs/       面向全队阅读的成品文档
materials/ 资料库、阅读笔记、模型记录、环境记录、实验记录
```

不提交到 Git：

```text
_external/  下载的外部仓库，例如 fpt26-harness
runs/       HLS/agent 运行输出
.env        API key、账号、私密环境变量
.agents/    本地 agent 状态
.codex/     本地 Codex 状态
```

## Team Rules

1. 正式资料只认本仓库。
2. 群聊只发链接，不发散乱文件。
3. 新资料先按 `materials/README.md` 分类。
4. 模型/API 相关资料必须写清楚来源、检查日期和 license 风险。
5. 实验记录必须使用 `materials/_templates/experiment-log-template.md`。
6. 不提交 API key、license server 信息、私有服务器密码。

## Upstream References

- FPT 2026 Design Competition: <https://fpt2026.uark.edu/fpt26-design-competition/>
- FPT26 harness: <https://anonymous.4open.science/r/fpt26-harness/README.md>
- Hello Agents: <https://helloagents.pro/#home>
