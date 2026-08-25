# Peng Liu · StormstoutLau

> Quantitative researcher building the verification layer for systematic finance and LLM agents.

One question runs through everything I build: **how much should we trust a factor, a model, or an agent — and how do we measure it?**

## Research Program

| Line | What | Flagship repos |
|---|---|---|
| **Factor diagnostics** | Which factors in the zoo genuinely fail — and how do we detect it before it hurts? | [Factor_Fingerprint], [Factor_Decoupler] |
| **LLM-agent verification** | When an LLM writes a trading strategy, how do we falsify it? | [deepseek-harness], [Spec_Workflow] |
| **Model fragility** | When does a pricing model stop being trustworthy? | [Cpp_Hub] + paper under review at *Quantitative Finance* |

## Selected Projects

### ① Factor diagnostics

**Flagship** — [Factor_Fingerprint]: fingerprints factors by time-series/cross-sectional stability plus semantic understanding, so each factor gets an identity and a failure profile. [Factor_Decoupler]: recovers clean innovations from dynamic factors via time-series decoupling.

Support: [factor_pipeline] · [Factor_Imputer] · [Factor_Neutralizer] · [Factor_AdaptiveWinsor] · [Factor_DB] · [Factor_Trading]

### ② LLM-agent verification

**Flagship** — [deepseek-harness]: an agent harness ("everything is a plugin") that constrains what LLM-generated code may do. [Spec_Workflow]: the spec-driven development workflow behind it, built for a solo developer working with LLM agents.

Support: [ponytail] · [Crucix-CN]

### ③ Model fragility & quantitative engineering

**Flagship** — [Cpp_Hub]: header-first C++20 pricing library (BS / Heston / PDE / Tree / MC / AAD Greeks / VaR / SVI, Python bindings), verified bit-identical across three platforms with 286+320 tests (v1.0). This is the engineering side of my model-fragility research: the paper proves a model can break, the library shows where and how to detect it.

Support: [audit-driven-development] · [paper2kg] · [textbook2kg] (Lean4 formal verification)

## Publications ↔ Code

| Paper | Status | Corresponding code |
|---|---|---|
| Derivative-operator framework for detecting model fragility | Under review — *Quantitative Finance* | [Cpp_Hub] (Heston / SABR / calibration) |
| Higher-moment (un)predictability (MIDAS line) | Preprint (SSRN) | [Factor_Decoupler] · [textbook2kg] (Lean4) |

Full list: [ORCID 0009-0008-7493-6473](https://orcid.org/0009-0008-7493-6473)

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

[Factor_Fingerprint]: https://github.com/StormstoutLau/Factor_Fingerprint
[Factor_Decoupler]: https://github.com/StormstoutLau/Factor_Decoupler
[factor_pipeline]: https://github.com/StormstoutLau/factor_pipeline
[Factor_Imputer]: https://github.com/StormstoutLau/Factor_Imputer
[Factor_Neutralizer]: https://github.com/StormstoutLau/Factor_Neutralizer
[Factor_AdaptiveWinsor]: https://github.com/StormstoutLau/Factor_AdaptiveWinsor
[Factor_DB]: https://github.com/StormstoutLau/Factor_DB
[Factor_Trading]: https://github.com/StormstoutLau/Factor_Trading
[deepseek-harness]: https://github.com/StormstoutLau/deepseek-harness
[Spec_Workflow]: https://github.com/StormstoutLau/Spec_Workflow
[ponytail]: https://github.com/StormstoutLau/ponytail
[Crucix-CN]: https://github.com/StormstoutLau/Crucix-CN
[Cpp_Hub]: https://github.com/StormstoutLau/Cpp_Hub
[audit-driven-development]: https://github.com/StormstoutLau/audit-driven-development
[paper2kg]: https://github.com/StormstoutLau/paper2kg
[textbook2kg]: https://github.com/StormstoutLau/textbook2kg