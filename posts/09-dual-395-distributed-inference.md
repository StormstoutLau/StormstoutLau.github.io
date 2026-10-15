# AMD Ryzen AI Max+ 395 双机分布式推理部署指南

> **适用环境**：Ubuntu 24.04 / 两台 AMD Ryzen AI Max+ 395（128GB 统一内存）通过 USB4 直连  
> **模型格式**：GGUF（来自 LM Studio 或其他源）  
> **推理后端**：Vulkan（备选 ROCm）  
> **分布式框架**：llama.cpp RPC 模式

---

## 1. 硬件与网络准备

### 1.1 硬件连接
- 使用 **USB4 线缆** 直连两台机器，系统会自动生成虚拟网卡（如 `enx...`）。
- 也可通过 **千兆/万兆交换机** 连接（推荐用于 3 台以上）。

### 1.2 配置静态 IP
在两台机器上分别执行（替换为实际网卡名）：
```bash
# 机器 A（主控）
sudo ip addr add 10.10.10.1/24 dev enx...
sudo ip link set enx... up

# 机器 B（计算节点）
sudo ip addr add 10.10.10.2/24 dev enx...
sudo ip link set enx... up
```
验证互通：
```bash
ping 10.10.10.2   # 在 A 上执行
ping 10.10.10.1   # 在 B 上执行
```

### 1.3 主机名与 SSH 免密登录（可选但推荐）
编辑 `/etc/hosts` 添加映射：
```
10.10.10.1  node1
10.10.10.2  node2
```
配置 SSH 免密（在 node1 上生成密钥并复制到 node2）：
```bash
ssh-keygen -t rsa
ssh-copy-id node2
```

---

## 2. 系统优化：扩展 GPU 显存

修改 GRUB 参数以突破 BIOS 96GB 限制，将每台机器的 GPU 可用显存提升至 **120GB**。

在两台机器上执行：
```bash
sudo nano /etc/default/grub
```
找到 `GRUB_CMDLINE_LINUX_DEFAULT`，在引号内追加：
```
ttm.pages_limit=30720000 amdgpu.gttsize=120000
```
保存后更新：
```bash
sudo update-grub
sudo reboot
```
重启后验证：
```bash
sudo dmesg | grep "amdgpu.*memory"
```
应看到类似 `120000M of GTT memory ready` 的输出。

---

## 3. 安装依赖与编译 llama.cpp（Vulkan 后端）

### 3.1 安装基础工具
```bash
sudo apt update
sudo apt install build-essential cmake git ccache curl
```

### 3.2 安装 Vulkan 驱动
```bash
sudo apt install mesa-vulkan-drivers vulkan-tools
vulkaninfo --summary   # 应能看到 Radeon 8060S
```

### 3.3 获取源码（离线包方式更稳）
从 [GitHub Releases](https://github.com/ggml-org/llama.cpp/releases) 下载 `Source code (tar.gz)`，上传到 `/tmp`，然后：
```bash
sudo tar -xzf /tmp/llama.cpp-*.tar.gz -C /opt/
sudo mv /opt/llama.cpp-* /opt/llama.cpp
sudo chown -R $USER:$USER /opt/llama.cpp
```

### 3.4 编译（RPC + Vulkan）
```bash
cd /opt/llama.cpp
mkdir -p build && cd build
cmake .. -DGGML_VULKAN=1 -DGGML_RPC=ON
cmake --build . --config Release -j $(nproc)
```
编译完成后，二进制文件位于 `/opt/llama.cpp/build/bin/`。  
*注*：某些环境下编译产物可能直接输出到 `/opt/llama.cpp/` 根目录，请确认 `llama-cli` 和 `ggml-rpc-server` 的实际位置。

### 3.5 验证编译成功
```bash
ls -l /opt/llama.cpp/build/bin/llama-cli
ls -l /opt/llama.cpp/build/bin/ggml-rpc-server
```
如果找不到，检查 `/opt/llama.cpp/` 根目录是否存在这些文件。

---

## 4. 模型准备：合并 GGUF 分片

如果下载的模型是多个分片（如 `model-00001-of-00002.gguf`），需先合并为一个完整文件。

使用 `llama-gguf-split` 工具（位于编译目录的 `bin/` 或根目录）：
```bash
# 进入模型存放目录
cd /path/to/model/dir

# 合并（第一个分片作为参数，输出指定合并后的文件名）
/opt/llama.cpp/build/bin/llama-gguf-split --merge model-00001-of-00002.gguf model-merged.gguf
```
验证：
```bash
file model-merged.gguf
```
应显示 `GGUF model data`。

---

## 5. 分布式推理配置

### 5.1 创建目录与配置文件
在 **主控节点**（如 node1）的 home 下创建 `llama-distributed` 目录：
```bash
mkdir -p ~/llama-distributed/logs
cd ~/llama-distributed
```

### 5.2 配置文件 `inference.conf`
```ini
# inference.conf - 推理参数配置
# 修改后保存，运行 run_inference.sh 即可

# ---------- 模型路径 ----------
MODEL_PATH="/home/user/.lmstudio/models/.../model-merged.gguf"

# ---------- RPC Worker 地址 ----------
RPC_ADDR="10.10.10.2:50052"   # Worker 节点的 IP 和端口

# ---------- 推理参数 ----------
PROMPT="你好，请介绍一下自己"
MAX_TOKENS=1024
TEMPERATURE=0.7
TOP_P=0.95
TOP_K=40
REPEAT_PENALTY=1.1
REPEAT_LAST_N=64
CTX_SIZE=2048
GPU_LAYERS=999                # -1 表示全部，999 表示尽可能多
THREADS=8
BATCH_SIZE=512

# ---------- 其他 ----------
VERBOSE=false
```

### 5.3 RPC 服务启动脚本 `start_rpc.sh`（在 Worker 节点执行）
```bash
#!/bin/bash
# Worker 节点启动 RPC 服务（后台运行）

RPC_BIN="/opt/llama.cpp/build/bin/ggml-rpc-server"
# 如果编译产物在根目录，则改为 /opt/llama.cpp/ggml-rpc-server
RPC_HOST="0.0.0.0"
RPC_PORT="50052"
LOG_DIR="$HOME/llama-distributed/logs"
LOG_FILE="$LOG_DIR/rpc_$(date +%Y%m%d_%H%M%S).log"

mkdir -p "$LOG_DIR"

if pgrep -f "ggml-rpc-server.*$RPC_PORT" > /dev/null; then
    echo "⚠️ RPC 服务已在端口 $RPC_PORT 运行，PID: $(pgrep -f "ggml-rpc-server.*$RPC_PORT")"
    exit 1
fi

echo "🚀 启动 RPC 服务，监听 $RPC_HOST:$RPC_PORT"
nohup "$RPC_BIN" -H "$RPC_HOST" -p "$RPC_PORT" > "$LOG_FILE" 2>&1 &

sleep 2
if pgrep -f "ggml-rpc-server.*$RPC_PORT" > /dev/null; then
    echo "✅ RPC 服务启动成功，PID: $(pgrep -f "ggml-rpc-server.*$RPC_PORT")"
else
    echo "❌ 启动失败，请检查日志: $LOG_FILE"
    exit 1
fi
```

### 5.4 推理执行脚本 `run_inference.sh`（在 Master 节点执行）
```bash
#!/bin/bash
# Master 节点执行推理

CONFIG_FILE="$(dirname "$0")/inference.conf"
if [ ! -f "$CONFIG_FILE" ]; then
    echo "❌ 配置文件不存在: $CONFIG_FILE"
    exit 1
fi
source "$CONFIG_FILE"

if [ -z "$MODEL_PATH" ] || [ ! -f "$MODEL_PATH" ]; then
    echo "❌ 模型文件不存在或未指定: $MODEL_PATH"
    exit 1
fi

if [ -z "$RPC_ADDR" ]; then
    echo "❌ 未指定 RPC_ADDR"
    exit 1
fi

# 检测 RPC 服务是否在线
RPC_HOST=$(echo $RPC_ADDR | cut -d':' -f1)
RPC_PORT=$(echo $RPC_ADDR | cut -d':' -f2)
if ! nc -z "$RPC_HOST" "$RPC_PORT" 2>/dev/null; then
    echo "⚠️ 无法连接到 RPC 服务 $RPC_ADDR，请确保 Worker 已启动 RPC。"
    exit 1
fi

LLAMA_CLI="/opt/llama.cpp/build/bin/llama-cli"
# 如果编译产物在根目录，则改为 /opt/llama.cpp/llama-cli

CMD="$LLAMA_CLI -m \"$MODEL_PATH\""
CMD="$CMD --rpc $RPC_ADDR"
CMD="$CMD -ngl $GPU_LAYERS"
CMD="$CMD -p \"$PROMPT\""
CMD="$CMD -n $MAX_TOKENS"
CMD="$CMD --temp $TEMPERATURE"
CMD="$CMD --top-p $TOP_P"
CMD="$CMD --top-k $TOP_K"
CMD="$CMD --repeat-penalty $REPEAT_PENALTY"
CMD="$CMD --repeat-last-n $REPEAT_LAST_N"
CMD="$CMD -c $CTX_SIZE"
CMD="$CMD -t $THREADS"
CMD="$CMD -b $BATCH_SIZE"

if [ "$VERBOSE" = "true" ]; then
    CMD="$CMD --verbose-prompt"
fi

echo "🚀 执行推理命令："
echo "$CMD"
echo ""
echo "---------- 开始推理 ----------"
eval "$CMD"
```

### 5.5 赋予执行权限
```bash
chmod +x ~/llama-distributed/start_rpc.sh
chmod +x ~/llama-distributed/run_inference.sh
```

---

## 6. 运行分布式推理

### 6.1 在 Worker 节点（如 node2）启动 RPC
```bash
cd ~/llama-distributed
./start_rpc.sh
```
或前台运行（便于调试）：
```bash
/opt/llama.cpp/build/bin/ggml-rpc-server -H 0.0.0.0 -p 50052
```

### 6.2 在 Master 节点（如 node1）执行推理
编辑 `~/llama-distributed/inference.conf` 确保模型路径和 RPC 地址正确，然后：
```bash
cd ~/llama-distributed
./run_inference.sh
```

### 6.3 结果示例
```
Prompt: 9.3 t/s | Generation: 40.7 t/s
```
表示成功。

---

## 7. 多节点扩展（3台及以上）

- 在新增节点上重复 **第2、3、6.1** 步骤。
- 在 Master 的 `inference.conf` 中，将 `RPC_ADDR` 改为以逗号分隔的列表（或使用多个 `--rpc` 参数）。
- 修改 `run_inference.sh` 支持多个 RPC 地址，或手动执行命令。

示例命令（3节点）：
```bash
/opt/llama.cpp/build/bin/llama-cli -m model.gguf \
  --rpc 10.10.10.2:50052 \
  --rpc 10.10.10.3:50052 \
  -ngl 999 -p "Hello" -n 256
```

---

## 8. 常见问题排查

### 8.1 RPC 连接失败
- 检查 Worker 的 RPC 是否运行：`ps aux | grep rpc-server`
- 检查防火墙：`sudo ufw disable`（临时）
- 使用 IP 代替主机名。

### 8.2 模型加载失败 "invalid split file name"
- 使用 `llama-gguf-split --merge` 正确合并。

### 8.3 Vulkan 报错
- 确保 `mesa-vulkan-drivers` 已安装。
- 设置环境变量：`export GGML_VK_PREFER_HOST_MEMORY=1`

### 8.4 显存不足（OOM）
- 减少 `-ngl` 层数。
- 减小上下文窗口 `-c`。
- 检查是否已将 TTM 参数调大（第2章）。

### 8.5 终端输出被截断
- 增大 `MAX_TOKENS` 参数。
- 重定向输出到文件查看完整内容。

---

## 9. 补充：使用 Beowulf AI Cluster 自动化（可选）

若不想手动每台编译，可使用 Ansible 自动化部署，但需确保：
- 所有机器 SSH 免密登录。
- 配置文件 `config.yml` 中指定 `llama_build_opts: "-DGGML_VULKAN=1 -DGGML_RPC=ON"`。
- 运行 `ansible-playbook main.yml --tags llama-setup`。

---

## 10. 总结

本指南提供了一个从零开始、可复现的双机分布式推理方案。核心要点：
- **硬件优化**：通过 TTM 参数释放最大显存。
- **软件选型**：Vulkan 后端 + RPC 模式，兼容性好，已验证。
- **脚本化**：通过 `start_rpc.sh` 和 `run_inference.sh` 简化日常操作。

---

**文档版本**：v1.0  
**最后更新**：2026-07-03  
**适用范围**：AMD Ryzen AI Max+ 395 双机（及以上）集群

---

> 如需完整脚本文件，可复制上述内容保存为 `deploy_guide.md` 或 `.sh` 文件使用。