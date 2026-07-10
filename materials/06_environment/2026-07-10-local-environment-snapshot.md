# 本机环境快照

Status: verified<br>
Owner: team<br>
Checked: 2026-07-10<br>
Source: local read-only command checks<br>
Expires/Risk: high，本机安装状态随时可能变化

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

可行环境路径：

1. 实验室或团队 Linux 服务器安装 Vitis 2025.2；
2. 配置 WSL2 Linux 发行版，并接入可用 Vitis 安装与 license；
3. 启动 Docker Desktop 后使用官方示例 Dockerfile，但 Vitis 安装与许可仍需单独解决；
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

不要在仓库中记录 license server 密码、SSH 私钥或 API key。
