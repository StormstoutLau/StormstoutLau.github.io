---
layout: default
title: Peng Liu · StormstoutLau
lang: en
description: Quantitative researcher building the verification layer for systematic finance and LLM agents.
---

# Peng Liu · StormstoutLau

[中文版 →](/zh/)

> Quantitative researcher building the verification layer for systematic finance and LLM agents.

One question runs through everything I build: **how much should we trust a factor, a model, or an agent — and how do we measure it?**

## Research Program

| Line | Question | Public repositories |
|---|---|---|
| **Factor diagnostics** | Which factors in the zoo genuinely fail — and how do we detect it before it hurts? | `factor_pipeline` + six `Factor_*` modules |
| **Auditable agent infrastructure** | When an LLM writes and runs a strategy, how do we know what it actually did? | `RPC` · `Spec_Workflow` · `audit-driven-development` |
| **Model fragility** | When does a pricing model stop being trustworthy? | `Cpp_Hub` + manuscripts under review |
| **Knowledge engineering** | Turning papers and textbooks into searchable, checkable structures. | `paper2kg` · `textbook2kg` |

## Selected Projects

### ① Factor diagnostics

**Flagship** — [factor_pipeline]: a unified orchestration layer spanning factor fingerprinting, semantic–statistical classification, per-type processing, migration monitoring and backtest integration. v3.1.0 passes 754 tests with zero regression; the v3.0.0 line passes 974 tests plus 11 subtests.

**Modules** — [Factor_Fingerprint] (fingerprints each factor from time-series and cross-sectional stability plus semantic understanding, so it gets an identity and a failure profile) · [Factor_Decoupler] (dual neutralisation that strips industry and size exposure while removing autocorrelation structure, isolating clean innovations) · [Factor_Imputer] (lookahead-free imputation, selected by factor type) · [Factor_Neutralizer] · [Factor_AdaptiveWinsor] · [Factor_DB] (DuckDB-backed local factor store) · [Factor_Trading] (backtest framework with a multi-level, multi-style agent decision layer)

### ② Auditable agent infrastructure

**Flagship** — [RPC]: a single-operator framework that runs a three-node local LLM inference cluster (AMD Ryzen AI Max+ 395, 128 GB unified memory per node, USB4 ring) and layers a spec-driven agent protocol on top — task cards → dispatch → collect → verify. Evidence is kept as an append-only chain with an off-machine anchor, so rewriting the archive is detectable; ~50 assertions run as pre-commit / pre-push gates. It is an operator's toolchain tuned for one specific ring, not a product.

**Support** — [Spec_Workflow]: the spec-driven development standard behind it, built for a solo researcher working with LLM agents (documentation-only; self-contained and portable across repositories since v1.2.1). [audit-driven-development]: multi-dimensional audit of code against design spec, producing scoring, issue severity, a fix-priority matrix and a fix baseline.

### ③ Model fragility & quantitative engineering

**Flagship** — [Cpp_Hub]: header-first C++20 pricing library (BS / Heston / PDE / Tree / MC / AAD Greeks / VaR / SVI, with Python bindings and optional CUDA MC). Extended in this period to **v1.4.3** with a high-frequency econometrics layer — realized measures and bipower variation, BNS / AJ / JO / Intraday / Rank jump tests, a realized kernel with 12 kernel functions plus noise-variance and bandwidth selection, multi-asset covariance estimators, liquidity measures, and HAR / HEAVY models — validated against the R `highfrequency` reference implementation at 1e-12. Cumulative cross-platform validation: **1412 / 1412 tests bit-exact across MSVC and GCC** under strict IEEE-754 mode.

This is the engineering side of the model-fragility research: the papers argue a model can break, the library shows where and how to detect it.

### ④ Knowledge engineering

[paper2kg] — classifies, renames and organises a local paper library into a structured, searchable knowledge base. [textbook2kg] — translates theorems, definitions and lemmas from mathematics textbooks into Lean 4 while building a searchable knowledge graph.

**Adaptations** — localised or extended forks of upstream projects: [Crucix-CN] · [deepseek-harness] · [ponytail] · [rea].

## Publications

| Paper | Status | Identifier |
|---|---|---|
| *Base Correlation Is Blind: Structural Model Risk in CDO Tranche Pricing* | Under review — *Quantitative Finance* | preprint v1.0 — [10.5281/zenodo.19493252](https://doi.org/10.5281/zenodo.19493252) |
| *Conduction Intensity and Structural Fragility of Heston Volatility Surfaces* | Preprint | [10.5281/zenodo.19826537](https://doi.org/10.5281/zenodo.19826537) |
| *A Joint Online Learning Framework for Time-Varying Parameter Correction and Multi-Scale Signal Decay* | Preprint | [10.2139/ssrn.6472825](https://doi.org/10.2139/ssrn.6472825) |
| *Mixed-Frequency Granger Causality in Higher Moments: Theory and Evidence* | Preprint | [10.2139/ssrn.6950078](https://doi.org/10.2139/ssrn.6950078) |

Full list: [ORCID 0009-0008-7493-6473](https://orcid.org/0009-0008-7493-6473)

## Writing

Long-form notes on local LLM infrastructure and workflows (mostly Chinese):

- [Local LLM deployment on an AMD Ryzen AI MAX+ 395 (128 GB) — 9-part series](/posts/)

## AI disclosure

Most code in these repositories was written with heavy LLM assistance under a spec-driven, audit-driven workflow. Research design, numerical verification, and the underlying formal claims remain my own work.

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
