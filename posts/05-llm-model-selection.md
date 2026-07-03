---
title: "量化金融场景的本地大模型选型指南"
date: 2026-06-08
tags:
  - 大模型选型
  - 量化金融
  - GPT-OSS
  - Qwen
  - Nemotron
  - 博客
aliases:
  - 模型推荐
  - 模型对比
---

# 量化金融场景的本地大模型选型指南

> 数学推理、代码生成、论文写作——不同任务用什么模型？

---

## 一、候选模型矩阵

| 模型 | 参数规模 | 推荐量化 | AMD 395 实测速度 | 特长 |
|:---|:---|:---|:---|:---|
| GPT-OSS 120B | 120B (5.1B激活) | MXFP4 | 24-40+ t/s | 编程/通用推理 |
| Nemotron Super 120B | 120B (12B激活) | NVFP4 | ~15 t/s | 数学/硬核推理 |
| Qwen3-235B MoE | 235B | Q4 | 14-18 t/s | 中文/超大容量 |
| Qwen3.5-35B MoE | 35B | Q4 | 40-45 t/s | 轻量/日常 |
| Qwen3 Coder Next | 30B A3B MoE | Q6_K | 30-40 t/s | 代码生成 |
| Qwen3.6-35B Claude蒸馏 | 35B (3B激活) | Q4 | 35-45 t/s | 编程+指令跟随 |
| Llama 4 Scout 109B | 109B MoE | Q4 | ~15 t/s | 多语言 |
| DeepSeek V4 Pro | 云端 | — | API | 综合最强(需联网) |

---

## 二、按任务选模型

### 2.1 数学推理 / LaTeX 论文

**首选：Nemotron 3 Super 120B**
- GPQA 80.0%, AIME 90.0%
- 硬核数理推理显著优于 GPT-OSS
- 缺点：AMD 平台上速度较慢（Mamba-2 架构非标准 Transformer）

**次选：GPT-OSS 120B**
- AIME 2024 96.6%
- 编程和通用推理也很强
- 四合一专家模型（Fin/Code/Math/Doc）

### 2.2 Python 代码生成

**首选：Qwen3 Coder Next (30B-A3B)**
- 30B 参数 / 3B 激活
- Q6_K 量化后 AMD 395 上 30-40 t/s
- 支持 FIM (Fill-in-Middle) 补全

**次选：Qwen3.6-35B Claude 蒸馏版**
- 蒸馏自 Claude 的输出，指令跟随能力强
- 适合 Agent 场景的代码生成

### 2.3 中文对话 / 日常 Agent

**首选：Qwen3-235B MoE**
- 中文能力最强的本地开源模型之一
- 235B 参数但 MoE 激活仅 ~22B
- AMD 395 上 14-18 t/s，可用但偏慢

**次选：Qwen3.5-35B MoE**
- 40-45 t/s 的流畅速度
- 日常对话完全够用

### 2.4 Agent 工具调用

**首选：Qwen3-30B-A3B（Hermes Agent 推荐）**
- Hermes Agent 官方验证过的最佳本地模型
- 工具调用准确率高

**次选：GPT-OSS 120B（需投机解码加速）**

---

## 三、投机解码加速策略

### 3.1 原理

用一个小的"草稿模型"快速生成候选 Token，主模型只需验证，可将吞吐量提升 1.5-2 倍。

### 3.2 推荐组合

| 主模型 | 草稿模型 | 接受率 | 加速比 |
|:---|:---|:---|:---|
| GPT-OSS 120B | P-EAGLE / EAGLE-3 | ~60-65% | 1.3-1.5x |
| Qwen3 Coder Next | 内置 MTP | ~70% | 1.5-2x |
| Nemotron 120B | Arctic Speculator | ~55% | 1.2-1.3x |

### 3.3 在 LM Studio 中配置

1. 加载主模型
2. 设置 → 投机解码 → 选择草稿模型
3. 观察接受率——低于 50% 时关闭（反而拖慢）

---

## 四、Qwen3.6-35B Claude 蒸馏版使用技巧

**来历**：通过 Claude API 输出蒸馏训练的 Qwen 模型，免费使用。

**系统提示**（英文）：

```
You are an AI assistant specialized in quantitative finance, 
mathematical proofs, and Python programming. Be precise and 
rigorous in mathematical derivations. When writing code, use 
clear variable names and add type hints.
```

**中英双语提示**在 Qwen3 上表现良好。温度设为 `0.2` 提高工具调用准确性。

---

## 五、我的实际使用组合

```
场景分层：
├── 编程 Agent (Hermes) → Qwen3-30B-A3B (快、准)
├── 数学推理 (论文)     → Nemotron 120B (硬核)
├── 通用问答              → Qwen3.5-35B MoE (快)
├── 中文深度讨论          → Qwen3-235B MoE (偶尔用，14 t/s)
├── 代码大活儿            → GPT-OSS 120B + 投机解码
└── 联网检索 / 最强能力    → DeepSeek V4 Pro (云端 API)
```

**存盘策略**：常驻加载 Qwen3.5-35B（40 t/s 流畅），需要时再切换到 Nemotron 或 235B。

---

> 本文素材提取自笔者与 DeepSeek 数十轮模型对比讨论（2026.01-06），所有性能数据均在 AMD Ryzen AI MAX+ 395 平台实测。