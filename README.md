# Peng Liu · StormstoutLau

> Quantitative researcher building the verification layer for systematic finance and LLM agents.

Independent researcher working at the intersection of quantitative finance, statistics, and AI. One question runs through everything I build: **how much should we trust a factor, a model, or an agent — and how do we measure it?**

## Research Interests

- **Factor diagnostics** — detecting when a factor genuinely fails, from the factor zoo to live strategies
- **LLM-agent verification** — falsifiable, statistically grounded constraints for financial AI agents
- **Derivative pricing & model fragility** — operator-based detection of when a model stops being trustworthy
- **Formal verification** — proving econometric and mathematical claims in Lean4

## Selected Projects

### LLM-agent tooling & verification

| Project | What it does |
|---|---|
| [deepseek-harness](https://github.com/StormstoutLau/deepseek-harness) | Agent harness — "everything is a plugin" |
| [Spec_Workflow](https://github.com/StormstoutLau/Spec_Workflow) | Spec-driven development workflow for a solo developer + LLM agents |
| [ponytail](https://github.com/StormstoutLau/ponytail) | Prompt wrapper that keeps an agent thinking like the laziest senior dev |
| [Crucix-CN](https://github.com/StormstoutLau/Crucix-CN) | Global intelligence radar, re-adapted for Chinese data sources and UI |

### Quantitative engineering (C++)

| Project | What it does |
|---|---|
| [Cpp_Hub](https://github.com/StormstoutLau/Cpp_Hub) | Header-first C++20 pricing library: BS/Heston/PDE/Tree/MC/AAD Greeks/VaR/SVI + Python bindings, verified across three platforms (v1.0) |

### Factor toolchain

| Project | What it does |
|---|---|
| [factor_pipeline](https://github.com/StormstoutLau/factor_pipeline) | Unified factor-processing orchestration |
| [Factor_Fingerprint](https://github.com/StormstoutLau/Factor_Fingerprint) | Factor fingerprinting and adaptive classification (time-series / cross-sectional stability) |
| [Factor_Decoupler](https://github.com/StormstoutLau/Factor_Decoupler) | Time-series decoupling — recovers clean innovations from dynamic factors |
| [Factor_Imputer](https://github.com/StormstoutLau/Factor_Imputer) | Lookahead-free factor imputation for A-share data |
| [Factor_Neutralizer](https://github.com/StormstoutLau/Factor_Neutralizer) | Industry / market-cap neutralization |
| [Factor_AdaptiveWinsor](https://github.com/StormstoutLau/Factor_AdaptiveWinsor) | Adaptive winsorization, distribution transform, standardization |
| [Factor_DB](https://github.com/StormstoutLau/Factor_DB) | DuckDB-backed local factor storage and querying |
| [Factor_Trading](https://github.com/StormstoutLau/Factor_Trading) | Backtesting framework — orders, factor management, market constraints, portfolio optimization, multi-style agent decision |

### Methodology

| Project | What it does |
|---|---|
| [audit-driven-development](https://github.com/StormstoutLau/audit-driven-development) | Multidimensional audit of code vs. design-spec alignment |
| [paper2kg](https://github.com/StormstoutLau/paper2kg) | Papers to knowledge graphs |
| [textbook2kg](https://github.com/StormstoutLau/textbook2kg) | Math textbooks to Lean4 formal verification code + searchable knowledge graph |

## Publications

- **A derivative-operator framework for detecting model fragility** — under review at *Quantitative Finance* (applied to CDO pricing and the Heston volatility surface)
- **Higher-moment (un)predictability** — MIDAS line, boundary results on predictability (SSRN)
- Full list: [ORCID 0009-0008-7493-6473](https://orcid.org/0009-0008-7493-6473)

## Writing

Long-form notes on infrastructure, workflows, and observations (mostly Chinese):

- [Local LLM deployment on an AMD Ryzen AI MAX+ 395 (128 GB) — 9-part series](posts/README.md)
- [Notes from a Potemkin Village (EN)](posts/10-potemkin-village-notes.md) · [草台班子观察笔记 (中文)](posts/10-草台班子观察笔记.md)

## Contact

- Email: peng.liu.john@gmail.com
- GitHub: [StormstoutLau](https://github.com/StormstoutLau)
- ORCID: [0009-0008-7493-6473](https://orcid.org/0009-0008-7493-6473)

## AI disclosure

Most code in these repositories was written with heavy LLM assistance under a spec-driven, audit-driven workflow. Research design, numerical verification, and the underlying formal claims remain my own work.