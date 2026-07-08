# LLM4HLS Materials

这个目录用于存放团队学习资料、来源记录、阅读笔记、环境说明和实验记录。  
`docs/` 放整理后的成品文档；`materials/` 放资料库和过程性说明。

## 目录结构

```text
materials/
  00_index/            总索引、阅读顺序、资料地图
  01_official/         官方规则、比赛页面、FAQ、通知摘要
  02_harness/          fpt26-harness 阅读笔记、文件地图、运行记录
  03_hls_basics/       FPGA/HLS/csim/synth/cosim 基础学习资料
  04_agent_basics/     agent loop、tool use、budget policy、框架对比
  05_models_and_apis/  开源模型、OpenRouter、API、license、排行榜记录
  06_environment/      Vitis、Docker、WSL、服务器、环境检查记录
  07_experiments/      模型/agent/任务实验记录和 scorecard 摘要
  08_paper_and_demo/   论文、答辩、demo、token consumption 材料
  _templates/          统一模板
```

## 放置规则

1. 根目录只放本 README，不放散乱资料。
2. 每份资料必须放到对应编号目录；不确定放哪里时，先放 `00_index/triage.md` 记录待归类原因。
3. 文件名使用小写英文、数字和短横线，例如 `2026-07-07-track-a-rules.md`。
4. 外部资料不要只复制链接，必须写清楚：
   - source URL
   - checked date
   - 一句话用途
   - 是否会过期
5. 不把 API key、账号、license server 信息、私有服务器密码写进本目录。
6. 大型原始仓库、下载包、运行输出不要放这里；原始仓库放 `_external/`，实验运行产物放对应 harness 的 `runs/` 或另设输出目录。
7. `07_experiments/` 中每次实验必须能回答：模型是什么、agent 版本是什么、任务是什么、预算是多少、结果如何。
8. 如果一份资料已经整理成面向全队阅读的成品文档，再同步链接到 `docs/` 或 `00_index/README.md`。

## 当前成品文档

- [Track A 0 基础入门手册](../docs/track-a-zero-foundation-guide.md)

## 资料状态标记

在资料开头建议加：

```text
Status: raw | summarized | verified | outdated
Owner: <name>
Checked: YYYY-MM-DD
Source: <url or local path>
```

含义：

- `raw`：刚收集，还没读完。
- `summarized`：已经有人读过并做了摘要。
- `verified`：已和官方资料/本地实验交叉确认。
- `outdated`：可能过期，不再作为当前判断依据。
