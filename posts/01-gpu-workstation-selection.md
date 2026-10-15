---
title: "GPU工作站选购指南：从Nvidia DGX Spark到AMD 395"
date: 2026-06-08
tags:
  - GPU
  - 硬件选型
  - 本地大模型部署
  - AMD 395
  - Nvidia
  - 博客
aliases:
  - GPU选购
  - 工作站选型
---

# GPU 工作站选购指南：从 Nvidia DGX Spark 到 AMD 395

> 量化研究者部署本地大模型，硬件怎么选？

---

## 一、核心场景：量化研究者需要什么

作为一名独立量化研究员，我的日常 LLM 使用场景包括：

- LaTeX 论文推理（Heston/SABR公式推导）
- 编程 Agent（Python 回测代码生成）
- 文献整理与形式化证明（Lean4/Mathlib 探索）
- 长期运行的 Agent 任务（Hermes Agent 工作流）

**核心需求**：128GB 统一内存（能跑 100B+ 参数模型）、合理推理速度、本地部署无隐私顾虑。

---

## 二、候选方案横评

| 维度 | **Nvidia DGX Spark** | **AMD 395（天钡NEX）** | **Apple M5 Max** |
|:---|:---|:---|:---|
| CPU | GB10 Grace Blackwell (ARM) | Ryzen AI MAX+ 395 (Zen 5, x86) | 18核 (6P+12E) |
| GPU | Blackwell (1000 TOPS FP4) | 40 RDNA 3.5 CU (126 TOPS INT8) | 最高40核 |
| 统一内存 | 128GB LPDDR5X | 128GB LPDDR5X-8533 | 最高128GB |
| 内存带宽 | 273 GB/s | ~273 GB/s | 614 GB/s |
| 互联 | NVLink-C2C + ConnectX-7 (200Gbps) | USB4 40Gbps × 2 | Thunderbolt 5 |
| 软件生态 | DGX OS (仅Linux), CUDA | Windows/Linux, ROCm/Vulkan | macOS, MLX |
| 参考价格 | ~¥33,999 | ¥14,999 (128GB+2TB) | ¥2.5-6.5万 |
| 扩展槽 | 有限 | 3×M.2 | 无 |

---

## 三、关键决策因素

### 3.1 CUDA vs ROCm：生态护城河

Nvidia 的 CUDA 生态是客观优势。绝大多数开源 LLM 框架（vLLM、llama.cpp GPU backend）优先适配 CUDA。AMD 的 ROCm 在追赶，但兼容性仍有差距。

**但**：对于推理场景（非训练），Vulkan backend 在 AMD iGPU 上已稳定可用。LM Studio 的 Vulkan 后端在 AMD 395 上实测：

| 模型 | 速度 | 量化 |
|:---|:---|:---|
| GPT-OSS 120B | 24-40+ t/s | MXFP4 |
| Qwen3-235B MoE | 14-18 t/s | Q4 |
| Llama 4 Scout 109B | ~15 t/s | Q4 |
| Qwen3.5-35B MoE | 40-45 t/s | Q4 |
| Phi-3.5 3.8B | ~61 t/s | Q4 |

### 3.2 内存带宽才是推理瓶颈

统一内存架构的推理性能，瓶颈在内存带宽而非算力。三款方案中 M5 Max 带宽最高（614 GB/s），但 macOS 的 LLM 生态不如 Linux 成熟。AMD 395 和 DGX Spark 带宽接近（~273 GB/s），理论上 128GB 容量的推理吞吐量差异不大。

### 3.3 x86 vs ARM：软件兼容性

这是 AMD 395 的关键优势。作为 x86 平台，所有 Linux 软件无需转译。DGX Spark 的 ARM 架构可能存在兼容性问题（某些 Python 库、Docker 镜像无 arm64 版本）。

---

## 四、我的选择与推荐

**我的实际配置**：2 台 AMD 395 设备（天钡 NEX）+ 计划购入 1 台 DGX Spark。

**推荐逻辑**：

- **预算 ≤ 2 万**：AMD 395（性价比之王，128GB 统一内存能跑 120B 模型）
- **预算 2-3 万**：AMD 395 × 2（双机 llama.cpp RPC 分布式推理）
- **预算 ≥ 3.5 万**：DGX Spark × 1 + AMD 395 × 1（混合架构：Spark 负责 CUDA 训练/微调，395 负责大容量推理）
- **预算充足且主力 macOS**：M5 Max 128GB（MLX 生态日渐成熟）

**不推荐单台 DGX Spark** 的原因：¥33,999 的价格可以买 2 台 AMD 395 + 还剩 ¥5,000，双机分布式推理能覆盖更大的模型。

---

## 五、关键教训

1. **显存 > 算力**：70B 模型推理，128GB 内存的重要性远大于 GPU 的 TOPS 数字
2. **生态兼容性 > 纸面参数**：AMD 395 的 x86 兼容性让 Ubuntu 配置少踩很多坑
3. **多机互联是性价比捷径**：USB4 40Gbps 直连 + llama.cpp RPC，2 台设备可用成本接近 1 台高端设备
4. **BIOS 配置别忘了**：iGPU 专用显存 512MB + GRUB 参数 `ttm.pages_limit=30720000 amdgpu.gttsize=120000`

---

> 本文素材提取自笔者与 DeepSeek 的多轮技术对话（2026.01-06），涵盖硬件选型、性能对比、部署实战全过程。