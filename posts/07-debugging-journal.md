---
title: "Ubuntu LLM 调优抓虫日记：崩溃/关机/内核升级实战"
date: 2026-06-08
tags:
  - 踩坑
  - LM Studio
  - GNOME
  - 内核
  - 调试
  - 博客
aliases:
  - 踩坑日记
  - 调试记录
---

# Ubuntu LLM 调优抓虫日记：崩溃、关机、内核升级

> 量化研究者在 AMD 395 + Ubuntu 24.04 上的真实排错记录

---

## 一、LM Studio 加载模型时崩溃

### 1.1 现象

LM Studio 用 Vulkan 后端加载模型时崩溃，GDB backtrace 显示：

```
ggml_abort
  ↑
ggml_backend_tensor_get              ← Vulkan 后端获取张量时崩溃
  ↑
llama_io_write_host::~llama_io_write_host()
  ↑
server_slot::prompt_save             ← 保存 Prompt Cache 时挂
  ↑
server_queue::start_loop
```

### 1.2 根因分析

Prompt Cache 保存时需要通过 Vulkan GPU 后端将缓存数据从 GPU 显存读回，但此操作触发了 `ggml_abort`。

三种可能：
1. **GPU 显存不足**：保存缓存时无法分配临时缓冲区
2. **Vulkan 驱动 bug**：特定操作触发断言失败
3. **Context Length 超出 GPU 能力**：分配 KV 缓存时爆显存

### 1.3 解决方案（按优先级）

1. **降低 Context Length**：8192 → 4096
2. **切换后端**：Vulkan → CPU AVX2 测试（确认是否 Vulkan 独有）
3. **更新驱动**：Vulkan 驱动和 LM Studio 更新到最新
4. **清除缓存**：删除模型目录下的 `*.cache` 文件

---

## 二、LM Studio 模型名解析错误（LM Link）

### 2.1 现象

LM Link 远程调用时报：

```
Cannot find model "qwen/qwen3-1.7b"
```

但：
- `/v1/models` 显示的是 `qwen3-coder-next`
- 被调用方没有 `qwen/qwen3-1.7b` 这个模型
- 单向错误：A→B 失败，B→A 正常

### 2.2 根因

GGUF 文件内部元数据的 `general.name` 字段是 `qwen/qwen3-1.7b`（权重的原始名称），而 LM Studio 显示的外部 ID 是 `qwen3-coder-next`。远程路由解析时**优先使用内部元数据名称**，导致找不到模型。

```
请求解析流程：
1. 客户端发送 model="qwen3-coder-next"
2. API 层查找注册表
3. 匹配失败/单模型模式 → 回退到已加载模型
4. 读取权重元数据 → 获取 "qwen/qwen3-1.7b"
5. 以内部 ID 生成路由 → 找不到模型 ❌
```

### 2.3 解决方案

**推荐方案**：CLI 强制指定标识符

```bash
lms load /path/to/model.gguf --identifier "qwen3-coder-next"
```

**备选方案**：修改 GGUF 元数据

```python
from gguf import GGUFReader, GGUFWriter

reader = GGUFReader("original.gguf")
writer = GGUFWriter("modified.gguf", "qwen3-coder-next")

for tensor in reader.tensors:
    writer.add_tensor(tensor.name, tensor.data)

for key, value in reader.metadata.items():
    if key == "general.name":
        value = "qwen3-coder-next"
    writer.add_metadata(key, value)

writer.close()
```

---

## 三、Ubuntu 登录后自动关机

### 3.1 现象

Ubuntu 24.04 登录后约 2 分钟自动关机，怀疑与内核或 GNOME 扩展有关。

### 3.2 排查过程

```bash
# 1. 禁用可疑 GNOME 扩展（ding@rastersoft.com）
gnome-extensions disable xxx
# 问题依旧 → 排除扩展

# 2. 检查内核崩溃
sudo dmesg | grep -i "panic\|oops\|mce"
# 无内核 panic → 排除内核崩溃

# 3. 分析上次启动日志
journalctl -b -1 --lines=500
```

**关键发现**：日志显示 `gnome-session` 在 21:18:59 退出，触发了用户级 `shutdown.target`。

### 3.3 结论与方案

**根因**：GNOME Session 异常退出 → systemd 用户实例触发关机，**非内核崩溃**。

**临时阻断**：

```bash
# 编辑 /etc/systemd/logind.conf
sudo nano /etc/systemd/logind.conf
# HandlePowerKey=ignore
# HandleSuspendKey=ignore
# HandleHibernateKey=ignore

# 创建新用户隔离测试
sudo useradd -m testuser && sudo passwd testuser
# 用 testuser 登录，确认是否为用户级配置问题
```

**建议**：排查 `~/.config/autostart/` 下的自启动项和 `~/.bashrc` 中的异常命令。

---

## 四、iGPU 显存识别不正确

### 4.1 现象

LM Studio 中 AMD iGPU 显存识别为远小于实际的 128GB。

### 4.2 解决

**BIOS**：iGPU Memory Size → **512MB**（不要超过）

**GRUB**：`/etc/default/grub` 添加：

```
GRUB_CMDLINE_LINUX_DEFAULT="... ttm.pages_limit=30720000 amdgpu.gttsize=120000"
```

```bash
sudo update-grub
sudo reboot
```

**验证**：

```bash
sudo dmesg | grep "amdgpu.*memory"
# 应显示: 120000M of GTT memory ready
```

---

## 五、Docker Desktop 依赖缺失

```bash
# 安装 .deb 包后
sudo dpkg -i docker-desktop-*.deb
# → 提示依赖缺失

# 修复
sudo apt --fix-broken install

# 如果仍缺失特定包
sudo apt install qemu-kvm libvirt-daemon-system
```

---

## 六、GNOME 自动关机后无法远程调试

```bash
# 查看可用内核列表
dpkg --list | grep linux-image

# 从 GRUB 选择旧内核启动（排查是否新内核问题）
# 启动时按住 Shift → 高级选项 → 选择旧内核

# 清理旧内核（确认新内核稳定后）
sudo apt autoremove --purge
```

---

## 七、排错方法论总结

经过 2026 年 1-6 月的密集踩坑，总结几条原则：

1. **先隔离再排查**：创建新用户 → 确认是系统级还是用户级问题
2. **日志是朋友**：`journalctl -b -1`、`dmesg`、GDB backtrace 依次查看
3. **降级验证**：新内核/新驱动出问题 → 切回旧版本确认
4. **不要同时改多个变量**：一次只改一个配置，确认效果后再改下一个
5. **记录一切**：每个排错步骤记录到 Obsidian，六个后你就是专家

---

> 本文所有案例均来自笔者在 AMD 395 + Ubuntu 24.04 上的实际排错经历，素材提取自与 DeepSeek 的多轮技术调试对话（2026.01-06）。