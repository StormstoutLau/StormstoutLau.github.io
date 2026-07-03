---
title: "本地大模型运行时完全指南：LM Studio + Ollama + vLLM + llama.cpp RPC"
date: 2026-06-08
tags:
  - LLM运行
  - LM Studio
  - Ollama
  - vLLM
  - llama.cpp
  - 分布式推理
  - 博客
aliases:
  - LLM部署
  - 推理引擎
---

# 本地大模型运行时完全指南：LM Studio / Ollama / vLLM / llama.cpp RPC

> 四套本地推理方案的实际部署与选择策略

---

## 一、四种方案定位

| 方案 | 定位 | 适用场景 |
|:---|:---|:---|
| **LM Studio** | 桌面推理前端 | 日常对话、代码生成、Agent 后端 |
| **Ollama** | 轻量级推理服务 | API 接入、OpenAI 兼容端点、快速原型 |
| **vLLM** | 高性能推理引擎 | 批量推理、高并发、生产级部署 |
| **llama.cpp RPC** | 分布式推理 | 多机协同、126B+ 超大模型分片 |

---

## 二、LM Studio — 桌面主力

### 2.1 核心参数配置

| 参数 | 说明 | 推荐值 |
|:---|:---|:---|
| Context Length | KV 缓存长度 | 32768-131072（视模型支持） |
| GPU Offload | 卸载到 GPU 的层数 | AMD iGPU 用 **Vulkan** 后端 |
| Temperature | 输出随机性 | 代码 0.2，创意 0.8 |
| Top-P | 核采样阈值 | 0.9-0.95 |
| Repeat Penalty | 惩罚重复 | 1.0-1.1 |
| CPU Threads | CPU 推理线程数 | 物理核心数 |
| Keep Model in Memory | 常驻内存 | 频繁使用时 **开启** |

### 2.2 后端选择

| GPU | 推荐后端 |
|:---|:---|
| AMD iGPU (RDNA 3.5) | **Vulkan** |
| NVIDIA GPU | **CUDA** |
| 纯 CPU | **AVX2** |

### 2.3 投机解码

需要同时加载草稿模型（Draft Model）和主模型。草稿模型应比主模型小、参数接近。LM Studio 0.3.x+ 支持内置 MTP（Multi-Token Prediction）。

草稿接受率约 **65%** 时加速效果明显；低于 50% 可能反而变慢。

### 2.4 API 模型管理

```bash
# 加载模型
curl http://localhost:1234/api/v0/models/load \
  -H "Content-Type: application/json" \
  -d '{"model": "模型标识"}'

# 卸载模型
curl -X POST http://localhost:1234/api/v0/models/unload

# 列出已加载模型
curl http://localhost:1234/api/v0/models

# "重启"模型
curl -X POST http://localhost:1234/api/v0/models/unload
sleep 2
curl http://localhost:1234/api/v0/models/load \
  -H "Content-Type: application/json" \
  -d '{"model": "模型标识"}'
```

### 2.5 LM Link 远程调用

LM Studio 的 LM Link 支持设备间的推理请求路由，但**不支持真正的模型分片分布式推理**。

**常见陷阱**：LM Link 中某设备请求另一设备模型时报 `Cannot find model`——

**根因**：GGUF 文件内部元数据中的 `general.name` 字段（如 `qwen/qwen3-1.7b`）与 LM Studio 显示的外部 ID（如 `qwen3-coder-next`）不一致，远程路由解析时优先使用内部元数据名称。

**解决方案**：
1. 使用 CLI 强制指定标识符：`lms load /path/to/model.gguf --identifier "qwen3-coder-next"`
2. 使用 `lms import` 重新导入并指定 ID
3. 修改 GGUF 文件元数据（Python GGUFReader/GGUFWriter）

---

## 三、Ollama — API 端点

```bash
# 安装
curl -fsSL https://ollama.com/install.sh | sh

# 基础命令
ollama pull qwen3:30b
ollama list
ollama ps
ollama show qwen3:30b

# OpenAI 兼容端点
# Base URL: http://localhost:11434/v1
# API Key: 任意非空值即可
```

**Windsurf / Trae IDE 接入**：在 IDE 设置中配置自定义 API Provider，Base URL 填 `http://localhost:11434/v1`，API Key 任意值。

---

## 四、vLLM — 高性能分布式推理

### 4.1 Ray 集群配置

```bash
# 主节点
ray start --head --port=6379

# 工作节点（其他设备）
ray start --address='<主节点IP>:6379'
```

### 4.2 启动推理服务

```bash
python -m vllm.entrypoints.openai.api_server \
  --model <模型路径> \
  --tensor-parallel-size 2 \
  --distributed-executor-backend ray \
  --host 0.0.0.0 --port 8000
```

### 4.3 限制

- vLLM 原生对 AMD ROCm 的支持尚不完美（2026.06 时点）
- Tensor Parallel 要求所有节点的 GPU 型号和显存大小一致
- 对于研究者的双机 128GB 场景，**llama.cpp RPC 是更好的选择**

---

## 五、llama.cpp RPC — 双机分片推理

### 5.1 架构

```
[设备 A (主节点)] 
    ↓ 分发计算任务
[设备 B (工作节点)] ← USB4 40Gbps 互联
    ↓ 返回计算结果
[设备 A] 汇总输出
```

### 5.2 Docker 部署

```bash
# 工作节点
docker run -d --rm --network=host --name=rpc-server \
    -v /path/to/models:/models \
    ghcr.io/ggml-org/llama.cpp:full-rocm \
    rpc-server -h 0.0.0.0 -p 50052

# 主节点
docker run --rm --network=host \
    -v /path/to/models:/models \
    ghcr.io/ggml-org/llama.cpp:full-rocm \
    llama-cli -m /models/model.gguf -n 128 \
    --rpc 192.168.1.10:50052 --rpc 192.168.1.11:50052
```

### 5.3 适用场景

- 126B+ 模型 → 单机内存不够，分片到两台 128GB 设备
- USB4 40Gbps 互联速度足够支撑模型权重传输
- 相比 vLLM：AMD ROCm 兼容性更好

---

## 六、方案选择决策树

```
需要什么？
├── 日常 Chat / Agent → LM Studio（桌面应用，省心）
├── 写代码需要 IDE 集成 → Ollama（OpenAI 兼容 API）
├── 高并发 / 批量推理 → vLLM（需 CUDA）
├── 126B+ 超大模型 → llama.cpp RPC（双机分片）
└── 以上都需要 → LM Studio + Ollama 组合（前端 LM Studio，API Ollama）
```

---

> 本文素材提取自笔者与 DeepSeek 多轮部署排错对话（2026.01-06），涵盖从基础安装到分布式推理的全链路。