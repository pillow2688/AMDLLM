# Track-A Submission Guidelines 中文翻译与执行清单

Status: team-received organizer guidance + partially cross-verified<br>
Owner: team<br>
Received: 2026-07-10<br>
Checked: 2026-07-10<br>
Source: 团队成员提供的英文 `Submission Guidelines for Track-A`；当前记录中未附原始发布页面、邮件或群公告地址<br>
Public page: https://fpt2026.uark.edu/fpt26-design-competition/<br>
Expires/Risk: high，提交入口、打包细节和评分权重仍可能更新

## 来源边界

本文件把团队收到的英文通知视为当前 Track-A 提交要求，并提供中文翻译和执行清单。

截至 2026-07-10，FPT 公开比赛页面尚未展示这 10 条 Submission Guidelines，搜索也未找到独立公开页面。因此团队应同时保存原始通知的截图、邮件、群公告或文件，记录发送方和发布时间。三个 Hugging Face 模型链接已核对存在，reference harness 也与 U55C、Vitis 2025.2、Docker 示例和 hidden grading 等要求一致。

## 十条要求逐条翻译

### 1. Co-simulation FPGA 平台

English:

> FPGA platform is targeted at the Alveo U55C for Co-simulation.

中文：

> 协同仿真使用的目标 FPGA 平台为 AMD Alveo U55C。

团队执行含义：至少要在 U55C 对应 part 上完成 co-simulation。reference harness 当前使用 `xcu55c-fsvh2892-2L-e`。

### 2. 软件版本

English:

> Software version is targeted at Vitis 2025.2.

中文：

> 目标软件版本为 AMD Vitis 2025.2。

团队执行含义：不能只在其他 Vitis 版本上通过后直接提交。最终实验应记录完整版本输出和环境构建方式。

### 3. 三类 HLS 流程和实验报告

English:

> Please pass csim, cosim and synth, and provide experimental report.

中文：

> 请确保 C simulation、C/RTL co-simulation 和 synthesis 均通过，并提供实验报告。

团队执行含义：最终候选不能只通过 `csim`。报告至少需要能追溯三类工具的结果、目标约束、关键日志、综合 latency/II/资源和失败处理过程。

### 4. 最低硬件频率

English:

> HLS generated hardware with at least 100Mhz.

中文：

> HLS 生成的硬件运行频率至少达到 100 MHz。

换算关系：

```text
100 MHz = 10 ns clock period
200 MHz = 5 ns clock period
```

最稳妥的报告方式是同时给出设定的 target clock 和 synthesis 报告中的 estimated/achieved clock，并证明不低于 100 MHz。reference harness 当前样例约束是 200 MHz，也就是 5 ns，比最低要求更严格。

### 5. Token 消耗

English:

> Token consumption is an important factor for final evaluation.

中文：

> Token 消耗是最终评测中的重要因素。

团队执行含义：实验不能只报告最终得分。每次运行至少记录输入 token、输出 token、总 token、模型调用次数、工具调用次数、停止原因和最终结果。

### 6. 推荐模型和扩展对比

English:

> Teams are recommended to evaluate their HLS agents using the following three models and report the experimental results. In addition to the recommended models, teams are encouraged to evaluate and compare their HLS agents using other publicly available open-weight foundation models or fine-tuned variants, and include the results in the technical report.

中文：

> 建议各团队使用下列三种模型评测自己的 HLS Agent，并报告实验结果。除推荐模型外，也鼓励团队使用其他公开可用的开放权重基础模型或微调版本进行评测和比较，并将结果写入技术报告。

关键词是 `recommended` 和 `encouraged`：三种模型是推荐对比组，不是唯一允许使用的模型。额外模型应是公开可用的 open-weight 模型或其微调版本，并准确报告模型 ID、revision、量化方式、provider 和推理设置。

### 7. Hidden benchmarks

English:

> Hidden test benchmarks exist for final evaluation.

中文：

> 最终评测包含隐藏测试基准。

团队执行含义：不能针对 public testbench 写特例。Agent 必须依据规格生成具有泛化能力的实现，并保护接口、数据类型、数值容差和边界条件。

### 8. Docker 环境

English:

> Submissions are expected to be built and run within a Docker environment. A Dockerfile is recommended to be included with the submission; a reference example is provided below: https://anonymous.4open.science/r/fpt26-harness

中文：

> 提交项目应能够在 Docker 环境中构建和运行。建议在提交材料中包含 Dockerfile；官方给出了 reference harness 作为示例。

措辞边界：Docker 构建/运行环境属于预期要求；随提交附带 Dockerfile 是推荐做法。reference harness 中的 `vitis.dockerfile` 主要安装 Ubuntu/Vitis 依赖，并没有把不可自由分发的 Vitis 2025.2 安装本体写入镜像，因此团队仍需明确 Vitis 安装、挂载、环境变量和 license 的复现方式。

### 9. 可复现材料

English:

> Source codes, testbenches and other supplementary materials that will be helpful for the reproduction, should be submitted for final evaluation (e.g., .Zip file).

中文：

> 最终评测时应提交有助于复现实验的源代码、测试平台和其他补充材料，例如使用 `.zip` 文件打包。

团队执行含义：除了 Agent 源码，还应提供依赖、配置示例、运行入口、测试说明、实验报告和不含秘密信息的复现材料。

### 10. 演示视频

English:

> Demonstration Video (max 5 min): must show project running on target platform with clear explanation.

中文：

> 演示视频最长 5 分钟，必须展示项目在目标平台上实际运行，并进行清晰讲解。

这与官网公布的 finalist 现场环节不是同一项：提交视频上限为 5 分钟；入围后的现场安排目前是 10 分钟口头展示和 live demo，加 5 分钟问答。

## 三种推荐模型

下面先忠实记录通知给出的参数，再单独列出模型卡核对结果。

| 推荐模型 | Model ID | 架构 | 总参数 / 激活参数 | Context | 通知所列量化 |
| --- | --- | --- | --- | ---: | --- |
| DeepSeek V4 Pro | `deepseek-ai/DeepSeek-V4-Pro` | MoE | 1.6T / 49B per token | 1,000,000 | FP8 official weights |
| Qwen3.5 122B A10B | `cyankiwi/Qwen3.5-122B-A10B-AWQ-4bit` | MoE | 122B / 10B per token | 262,144 | AWQ-4bit |
| Qwen3.6 27B | `Qwen/Qwen3.6-27B-FP8` | Dense | 27B / 27B per token | 262,144 | FP8 |

Links:

- https://huggingface.co/deepseek-ai/DeepSeek-V4-Pro
- https://huggingface.co/cyankiwi/Qwen3.5-122B-A10B-AWQ-4bit
- https://huggingface.co/Qwen/Qwen3.6-27B-FP8

## 模型卡交叉核对

### DeepSeek V4 Pro

- 仓库属于 `deepseek-ai` 官方组织，Hugging Face 标注 MIT license；
- 模型卡确认 1.6T 总参数、49B 激活参数和 1M context；
- 当前模型卡把 `DeepSeek-V4-Pro` 标为 FP4+FP8 mixed，而 `DeepSeek-V4-Pro-Base` 标为 FP8 mixed；
- 因此通知中的 `FP8 (Official Weights)` 与当前 Pro 模型卡存在表述差异，实验报告必须按实际下载 revision 或 API backend 记录，不能只写“FP8”。

### Qwen3.5 122B A10B AWQ-4bit

- 链接属于 `cyankiwi` 账号，是社区量化仓库，不是 `Qwen` 官方组织仓库；
- Hugging Face 标注 Apache-2.0 license；
- 模型卡记录默认 context 为 262,144；
- 报告中应同时记录基础模型与量化仓库的完整 ID、revision 和量化配置。

### Qwen3.6 27B FP8

- 链接属于 `Qwen` 官方组织；
- Hugging Face 标注 Apache-2.0 license；
- 模型卡确认 27B 参数、FP8 权重和 262,144 默认 context。

## 团队提交清单

### HLS 验证

- [ ] 使用 Vitis 2025.2；
- [ ] 目标 part 为 Alveo U55C；
- [ ] 每个最终候选通过 `csim`；
- [ ] 每个最终候选通过 `synth`；
- [ ] 每个最终候选通过 `cosim`；
- [ ] target 和 achieved clock 均有报告，最终频率不低于 100 MHz；
- [ ] 保存 latency、II、LUT、FF、DSP、BRAM、URAM 和关键日志。

### Agent 和模型实验

- [ ] 三个推荐模型均有可复现的实验结果，或在报告中明确说明无法完成的原因；
- [ ] 额外模型使用公开可用的 open-weight 权重或微调版本；
- [ ] 记录 exact model ID、revision、provider、量化、context 和 generation 参数；
- [ ] 记录每阶段和每次运行的 token consumption；
- [ ] 使用同一任务、预算、prompt 版本和重复次数做公平比较；
- [ ] 保留失败运行，不能只展示最好的一次。

### Docker 和复现包

- [ ] Docker 镜像可以从干净环境构建；
- [ ] 提供 Dockerfile、构建命令和运行命令；
- [ ] 写明 Vitis 2025.2 与 license 如何安装、挂载或接入；
- [ ] 提供 Agent 源码、配置模板、testbench 和必要脚本；
- [ ] 提供依赖锁定文件或明确版本列表；
- [ ] `.zip` 解压后有唯一清晰的入口 README；
- [ ] 不包含 API key、license server 密码、SSH 私钥和个人 token。

### 报告和视频

- [ ] 实验报告包含 csim、synth、cosim 结果；
- [ ] 报告包含三种推荐模型和其他候选的对比；
- [ ] 报告包含 token、tool calls、wall time、正确率和 PPA；
- [ ] 5 分钟视频展示真实运行，不只播放幻灯片；
- [ ] 视频显示目标平台、输入任务、Agent 过程和最终结果；
- [ ] 视频讲解清晰，并在时限内完成。

## 仍需向组织者确认

1. 这份 Submission Guidelines 的正式公开地址、发布日期和版本号；
2. 100 MHz 以 target clock、synthesis estimated clock 还是实现后实际频率判定；
3. Docker 是否必须包含完整 Vitis，还是允许挂载评测方提供的 Vitis 2025.2；
4. 是否允许通过 OpenRouter 或其他 API 调用推荐 open-weight 模型，还是要求本地部署；
5. token 的统计口径、上限和最终评分权重；
6. 三个推荐模型需要跑多少任务、多少随机种子和多少重复实验；
7. experimental report 是独立文件、论文 appendix，还是两者都要；
8. `.zip` 的目录结构、大小上限、提交入口和视频提交方式。

在这些问题得到正式答复前，团队应采用更容易复现和审计的实现，并在报告中明确披露实际环境与假设。
