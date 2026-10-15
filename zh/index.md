---
layout: default
title: Peng Liu · StormstoutLau
lang: zh
description: 为系统性金融与 LLM 智能体建造验证层的研究者。
---

# Peng Liu · StormstoutLau

[English →](/)

> 为系统性金融与 LLM 智能体建造验证层的研究者。

我做的一切都源于同一个问题：**一个因子、一个模型、一个智能体，究竟应当在多大程度上被信任——以及，如何测量这份信任？**

## 研究纲领

| 主线 | 追问的问题 | 公开仓库 |
|---|---|---|
| **因子诊断** | 因子动物园里哪些因子真的失效了——以及如何在它造成损失之前发现？ | `factor_pipeline` + 六个 `Factor_*` 模块 |
| **可审计智能体基础设施** | 当 LLM 写出并运行一个策略，我们凭什么知道它究竟做了什么？ | `RPC` · `Spec_Workflow` · `audit-driven-development` |
| **模型脆弱性** | 一个定价模型从哪一刻起不再可信？ | `Cpp_Hub` + 在审论文 |
| **知识工程** | 把论文与教材变成可检索、可核验的结构。 | `paper2kg` · `textbook2kg` |

## 代表项目

### ① 因子诊断

**旗舰** —— [factor_pipeline]：统一编排层，贯通因子指纹诊断、语义–统计融合分类、按类型差异化处理、迁移监测与回测集成。v3.1.0 通过 754 项测试且零回归；v3.0.0 线通过 974 项测试加 11 项子测试。

**模块** —— [Factor_Fingerprint]（由时序稳定性、截面稳定性与语义理解生成因子指纹，让每个因子获得身份与失效画像）· [Factor_Decoupler]（双重中性化：在剥离行业与市值暴露的同时去除时序自相关结构，提取纯净新息）· [Factor_Imputer]（无前瞻偏差插补，按因子类型自动择优）· [Factor_Neutralizer] · [Factor_AdaptiveWinsor] · [Factor_DB]（以 DuckDB 为引擎的本地因子存储）· [Factor_Trading]（回测框架，含多层级、多风格的 Agent 协作决策层）

### ② 可审计智能体基础设施

**旗舰** —— [RPC]：单操作者框架，在 USB4 环上运行三机本地 LLM 推理集群（AMD Ryzen AI Max+ 395，每节点 128 GB 统一内存），并在其上叠加 spec 驱动的 agent 协议——任务卡 → 派发 → 收集 → 核验。运行证据以追加式链条保存并带站外锚点，改写归档可被检出；约 50 条断言作为 pre-commit / pre-push 门禁运行。它是针对特定三机环调校的操作者工具链，不是产品。

**支撑** —— [Spec_Workflow]：其背后的 spec 驱动开发规范，面向「单人研究者 + LLM 智能体」工作流（纯文档型；自 v1.2.1 起自包含、可跨仓库迁移）。[audit-driven-development]：代码与设计规格的多维对齐审计，输出评分、问题严重度、修复优先级矩阵与修复基线。

### ③ 模型脆弱性与量化工程

**旗舰** —— [Cpp_Hub]：头文件优先的 C++20 定价库（解析解 / Heston / PDE / 树形 / 蒙特卡洛 / AAD Greeks / VaR / SVI，含 Python 绑定与可选 CUDA MC）。本期迭代至 **v1.4.3**，新增高频计量经济学层——已实现测度与双幂变差、BNS / AJ / JO / Intraday / Rank 跳跃检验、12 种核函数的 Realized Kernel 及噪声方差与最优带宽选择、多资产协方差估计、流动性度量、HAR / HEAVY 模型——以 1e-12 容差对标 R `highfrequency` 参考实现。累计跨平台验证：**1412 / 1412 项测试在 MSVC 与 GCC 下位精确一致**（严格 IEEE-754 模式）。

这是模型脆弱性研究的工程侧：论文论证模型会坏，代码库展示它在哪里坏、如何检测。

### ④ 知识工程

[paper2kg] —— 将本地论文库自动分类、重命名、整理为结构化可检索的知识库。[textbook2kg] —— 把数学教材中的定理、定义与引理翻译为 Lean 4，并构建可检索知识图谱。

**改编项目** —— 上游 fork 的本地化或扩展：[Crucix-CN] · [deepseek-harness] · [ponytail] · [rea]。

## 论文

| 论文 | 状态 | 公开标识 |
|---|---|---|
| *Model Fragility Beyond Base Correlation: An Operator-Theoretic Approach to CDO Risk Measurement* | 在审 —— *Quantitative Finance* | [10.5281/zenodo.19493252](https://doi.org/10.5281/zenodo.19493252) |
| *Conduction Intensity and Structural Fragility of Heston Volatility Surfaces* | 预印本 | [10.5281/zenodo.19826537](https://doi.org/10.5281/zenodo.19826537) |
| *A Joint Online Learning Framework for Time-Varying Parameter Correction and Multi-Scale Signal Decay* | 预印本 | [10.2139/ssrn.6472825](https://doi.org/10.2139/ssrn.6472825) |
| *Mixed-Frequency Granger Causality in Higher Moments: Theory and Evidence* | 预印本 | [10.2139/ssrn.6950078](https://doi.org/10.2139/ssrn.6950078) |

完整列表：[ORCID 0009-0008-7493-6473](https://orcid.org/0009-0008-7493-6473)

## 写作

关于本地 LLM 基础设施与工作流的长文（以中文为主）：

- [AMD Ryzen AI Max+ 395（128 GB）本地 LLM 部署 —— 九篇系列](/posts/)

## AI 使用披露

本仓库群中的大部分代码，均在 spec 驱动、审计驱动的工作流下由 LLM 大量协助写成。研究设计、数值验证与底层形式化主张仍为本人工作。

[factor_pipeline]: https://github.com/StormstoutLau/factor_pipeline
[Factor_Fingerprint]: https://github.com/StormstoutLau/Factor_Fingerprint
[Factor_Decoupler]: https://github.com/StormstoutLau/Factor_Decoupler
[Factor_Imputer]: https://github.com/StormstoutLau/Factor_Imputer
[Factor_Neutralizer]: https://github.com/StormstoutLau/Factor_Neutralizer
[Factor_AdaptiveWinsor]: https://github.com/StormstoutLau/Factor_AdaptiveWinsor
[Factor_DB]: https://github.com/StormstoutLau/Factor_DB
[Factor_Trading]: https://github.com/StormstoutLau/Factor_Trading
[RPC]: https://github.com/StormstoutLau/RPC
[Spec_Workflow]: https://github.com/StormstoutLau/Spec_Workflow
[audit-driven-development]: https://github.com/StormstoutLau/audit-driven-development
[Cpp_Hub]: https://github.com/StormstoutLau/Cpp_Hub
[paper2kg]: https://github.com/StormstoutLau/paper2kg
[textbook2kg]: https://github.com/StormstoutLau/textbook2kg
[Crucix-CN]: https://github.com/StormstoutLau/Crucix-CN
[deepseek-harness]: https://github.com/StormstoutLau/deepseek-harness
[ponytail]: https://github.com/StormstoutLau/ponytail
[rea]: https://github.com/StormstoutLau/rea
