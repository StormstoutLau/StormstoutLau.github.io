# AMD 395 本地大模型部署实战

> 在 AMD Ryzen AI MAX+ 395 (128GB) + Ubuntu 24.04 上部署本地大模型的完整踩坑记录。

## 硬件配置

| 组件 | 规格 |
|:---|:---|
| CPU | AMD Ryzen AI MAX+ 395 (Zen 5, 16C/32T) |
| GPU | Radeon 8060S (40 RDNA 3.5 CU) |
| 内存 | 128GB LPDDR5X-8533 统一内存 |
| OS | Ubuntu 24.04 LTS (HWE-edge kernel 6.17) |

## 系列文章

| # | 标题 | 主题 |
|:---:|:---|:---|
| 1 | [选购篇](posts/01-gpu-workstation-selection.md) | Nvidia DGX Spark vs AMD 395 横评 |
| 2 | [环境篇](posts/02-ubuntu-dev-environment.md) | Ubuntu 24.04 AI 开发环境从零搭建 |
| 3 | [网络篇](posts/03-ubuntu-network-setup.md) | Clash 代理 + Samba 共享 + USB4 双机互联 |
| 4 | [运行时篇](posts/04-llm-runtime-guide.md) | LM Studio / Ollama / vLLM / llama.cpp RPC |
| 5 | [选型篇](posts/05-llm-model-selection.md) | 量化金融场景的本地大模型选择 |
| 6 | [Agent篇](posts/06-local-agent-deployment.md) | Hermes Agent + OpenClaw + 飞书接入 |
| 7 | [踩坑篇](posts/07-debugging-journal.md) | LM Studio 崩溃 / GNOME 关机 / 内核升级 |
| 8 | [方法篇](posts/08-vibe-proving.md) | LLM 辅助数学证明 (Vibe Proving) |

## 实测性能数据 (AMD 395, Vulkan backend)

| 模型 | 量化 | 推理速度 |
|:---|:---|:---|
| GPT-OSS 120B | MXFP4 | 24-40+ t/s |
| Qwen3-235B MoE | Q4 | 14-18 t/s |
| Qwen3.5-35B MoE | Q4 | 40-45 t/s |
| Llama 3.3 70B | Q6_K | ~5 t/s |

## License

MIT
