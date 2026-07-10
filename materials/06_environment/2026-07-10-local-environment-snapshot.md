# 本机环境快照

Status: verified<br>
Owner: team<br>
Checked: 2026-07-10<br>
Sources: local read-only command checks, Track-A Submission Guidelines, reference harness<br>
Expires/Risk: high，本机安装状态随时可能变化

## 比赛目标环境

团队收到的提交指南要求：

```text
FPGA platform: AMD Alveo U55C for co-simulation
Software:      Vitis 2025.2
Validation:    csim + synth + cosim pass
Clock:         at least 100 MHz, period <= 10 ns
Packaging:     expected to build and run in Docker
```

reference harness 当前默认 part 为 `xcu55c-fsvh2892-2L-e`，目标为 200 MHz/5 ns。200 MHz 是当前 reference 默认值，100 MHz 是补充指南给出的最低门槛，实际运行仍应读取正式 task package 的 target 配置。

## 当前系统

工作目录：

```text
C:\Users\16234\Documents\LLM4HLS
```

当前开发主机为 Windows，reference harness 目标环境是 Linux 风格的 Vitis 2025.2 工具链。

## 已发现

- Docker CLI 存在，版本为 28.0.4；
- Docker daemon 当前不可连接，默认 pipe 不存在；
- WSL 命令存在，默认版本为 WSL 2；
- `wsl --list --quiet` 没有返回已安装发行版；
- Conda 存在，版本为 25.1.0；
- `vitis-run` 与 `vitis_hls` 不在当前 PATH；
- 未发现 `LLM4HLS_VITIS_HLS_ROOT`、`XILINX_VITIS` 或 `XILINX_HLS` 环境变量。

## 当前结论

这台 Windows 主机目前不能直接运行 reference harness 的 Vitis HLS 闭环。

提交指南预期项目在 Docker 中构建和运行，并建议附 Dockerfile。reference harness 的 `vitis.dockerfile` 安装了 Ubuntu 和 Vitis 所需系统依赖，但没有安装 Vitis 2025.2 本体，也没有解决 license。因此“Docker 能启动”不等于“可复现 csim/synth/cosim”。

可行环境路径：

1. 实验室或团队 Linux 服务器安装 Vitis 2025.2；
2. 配置 WSL2 Linux 发行版，并接入可用 Vitis 安装与 license；
3. 启动 Docker Desktop 后以 reference `vitis.dockerfile` 为基础，另外配置 Vitis 2025.2 安装或只读挂载、环境脚本和 license；
4. 远程连接已有 AMD/Vitis 环境。

## 下一次环境检查

环境变更后至少重新运行：

```powershell
Get-Command docker,wsl,conda,vitis-run,vitis_hls -ErrorAction SilentlyContinue
wsl --list --quiet
docker version
conda --version
```

并在 Linux/Vitis 环境中检查：

```bash
vitis-run --version
test -f "$LLM4HLS_VITIS_HLS_ROOT/settings64.sh"
python scripts/run_poc.py tasks/projection_bugfix
```

提交前还要保存以下证据：

- `vitis-run --version` 输出；
- U55C part 和 target clock；
- `csim`、`synth`、`cosim` 三类通过记录；
- synthesis estimated clock，证明最终频率不低于 100 MHz；
- 从干净 Docker 环境执行的完整构建和运行命令；
- Vitis/license 接入说明，但不包含任何秘密值。

不要在仓库中记录 license server 密码、SSH 私钥或 API key。
