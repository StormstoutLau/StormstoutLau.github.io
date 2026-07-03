---
title: "本地Agent部署实战：Hermes Agent + OpenClaw + QClaw"
date: 2026-06-08
tags:
  - Agent
  - Hermes
  - OpenClaw
  - Qwen
  - 飞书
  - 博客
aliases:
  - Agent部署
  - 本地Agent
---

# 本地 Agent 部署实战：Hermes Agent + OpenClaw + QClaw

> 在 AMD 395 平台上用本地大模型构建编程与研究 Agent

---

## 一、Hermes Agent — 本地编程 Agent 主力

### 1.1 安装

```bash
# 下载 AppImage
chmod +x hermes-desktop-0.5.0.AppImage
mkdir -p ~/Applications
mv hermes-desktop-0.5.0.AppImage ~/Applications/

# 创建桌面图标
nano ~/.local/share/applications/hermes.desktop
```

```ini
[Desktop Entry]
Name=Hermes Agent
Comment=AI Agent
Exec=/home/scott-lau/Applications/hermes-desktop-0.5.0.AppImage --no-sandbox
Icon=/home/scott-lau/.local/share/icons/hermes-icon.png
Terminal=false
Type=Application
Categories=Development;AI;
StartupWMClass=hermes
```

### 1.2 连接 LM Studio 本地模型

**config.yaml**（`~/.hermes/config.yaml`）：

```yaml
model:
  provider: "lmstudio"
  model_name: "qwen3-30b-a3b"
  api_key: "lm-studio"
  base_url: "http://localhost:1234/v1"
  temperature: 0.2
  max_tokens: 8192
```

### 1.3 常见报错：`401 Malformed LM Studio API token`

**根因**：Hermes 从 `~/.hermes/.env` 读取 `CUSTOM_API_KEY` 失败，发送了占位符 token。

**解决方案**（三选一）：

**方案一**：直接在 config.yaml 中写明 api_key：

```yaml
api_key: "lm-studio"
```

**方案二**：使用 CLI 设置：

```bash
hermes config set api_key "lm-studio"
```

**方案三**：环境变量：

```bash
export LM_API_KEY="lm-studio"
```

### 1.4 模型选择

| 场景 | 推荐模型 | 理由 |
|:---|:---|:---|
| 编程 | Qwen3-30B-A3B | Hermes 官方验证，工具调用准确 |
| 学术/证明 | Qwen3-30B-A3B + 专属 system prompt | AIME'25 准确率提升 67% |
| 轻量 | Qwen3.5-35B MoE | 速度快 (40+ t/s) |

### 1.5 System Prompt 设计

```
You are an AI programming agent specialized in quantitative finance.
You can write Python code, analyze data, and implement mathematical models.
When working with code:
- Use clear variable names and type hints
- Add docstrings for functions
- Prefer vectorized operations over loops
- Document any assumptions about data format

For mathematical work:
- Use LaTeX notation in explanations
- State assumptions clearly before deriving results
- Verify edge cases
```

温度设为 `0.2` 以提高工具调用准确性。中英双语提示在 Qwen3 上效果良好。

### 1.6 Sub-Agent 机制

Hermes 的子代理是同一模型在不同对话历史中的**逻辑分身**，共享模型计算资源但记忆不互通。**不需要独立 API 端口**——一个端口足够支持复杂多代理场景。

---

## 二、OpenClaw — 通用 Agent 平台

### 2.1 Docker Desktop + Docker Compose 部署

```bash
# 目录准备
mkdir -p ~/.openclaw

# docker-compose.yml
nano docker-compose.yml
```

```yaml
services:
  openclaw-gateway:
    image: ghcr.io/openclaw/openclaw:latest
    container_name: openclaw-gateway
    ports:
      - "18789:18789"
    volumes:
      - ~/.openclaw:/home/node/.openclaw
    extra_hosts:
      - "host.docker.internal:host-gateway"
    restart: unless-stopped
```

**注意**：volumes 挂载路径必须是 `/home/node/.openclaw`，不能用 `/root/.openclaw`。

### 2.2 连接 LM Studio

**`~/.openclaw/openclaw.json`**：

```json
{
  "models": {
    "providers": {
      "lmstudio": {
        "baseUrl": "http://host.docker.internal:1234/v1",
        "apiKey": "any-value",
        "api": "openai-completions",
        "models": [{
          "id": "qwen3-30b-a3b",
          "name": "Qwen3 30B",
          "reasoning": false,
          "input": ["text"],
          "cost": { "input": 0, "output": 0 },
          "contextWindow": 32768,
          "maxTokens": 8192
        }]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": { "primary": "lmstudio/qwen3-30b-a3b" }
    }
  },
  "gateway": {
    "auth": { "mode": "none" },
    "bind": "127.0.0.1"
  }
}
```

### 2.3 常见故障

| 现象 | 根因 | 解决 |
|:---|:---|:---|
| 容器 Restarting 循环 | `Refusing to bind gateway to auto without auth` | 添加 `"bind": "127.0.0.1"` 或启用 token |
| 配置文件不生效 | 挂载路径错误 | volumes 改为 `/home/node/.openclaw` |
| JSON 解析错误 | 逗号冗余 | `python3 -m json.tool` 验证 |

### 2.4 飞书接入（远程遥控）

```bash
# 安装飞书插件
openclaw plugins install feishu-openclaw

# 配置
openclaw config set channels.feishu.appId "xxx"
openclaw config set channels.feishu.appSecret "xxx"
openclaw config set channels.feishu.enabled true

# 安装 gateway
openclaw gateway install
systemctl --user start openclaw-gateway
```

**飞书开放平台配置**：
1. 创建应用 → 添加机器人能力
2. 导入权限：
   ```json
   {
     "scopes": {
       "tenant": [
         "im:message",
         "im:message.group_at_msg:readonly",
         "im:message.p2p_msg:readonly",
         "im:message:send_as_bot",
         "im:resource"
       ]
     }
   }
   ```
3. 事件订阅使用 **长连接接收事件**（不用 Webhook）
4. 发布应用

### 2.5 LM Studio 后台启动（OpenClaw 依赖）

```bash
nohup /home/scott-lau/Applications/LM-Studio-*.AppImage \
    --server --port 1234 --host 0.0.0.0 \
    --no-sandbox > /tmp/lmstudio.log 2>&1 &
```

---

## 三、QClaw — 腾讯系 Agent

QClaw 支持兼容 OpenAI API 格式的本地服务。通过 Ollama 的 OpenAI 兼容端点接入：

```
Base URL: http://localhost:11434/v1
API Key: 任意非空值
Model: qwen3:30b
```

---

## 四、Windsurf Cascade 接入本地模型

在 Windsurf 设置中配置自定义 API Provider：

| 字段 | 值 |
|:---|:---|
| Provider | Custom (OpenAI Compatible) |
| Base URL | `http://localhost:11434/v1` |
| API Key | 任意非空值 |
| Model | `qwen3:30b` |

---

## 五、Agent 选择策略

| 我需要... | 选... | 原因 |
|:---|:---|:---|
| IDE 内编程 | Windsurf Cascade + Ollama | 直接集成，无需切换 |
| 复杂多文件工程 | Hermes Agent | 子代理并行 + 90 轮迭代 |
| 飞书远程操控 | OpenClaw | 原生飞书插件 |
| 轻量级日常 | QClaw | 腾讯系，中文友好 |
| 长期运行任务 | Hermes + LM Studio | 记忆管理 + 经验沉淀 (SKILL.md) |

---

> 本文素材提取自笔者与 DeepSeek 多轮 Agent 部署调优对话（2026.01-06），涵盖从安装到飞书接入的完整流程。