---
title: "Ubuntu 24.04 AI开发环境从零搭建"
date: 2026-06-08
tags:
  - Ubuntu
  - 开发环境
  - LM Studio
  - Anaconda
  - Docker
  - 博客
aliases:
  - Ubuntu部署
  - 环境搭建
---

# Ubuntu 24.04 AI 开发环境从零搭建

> 量化研究者 Ubuntu 工作站的完整初始化流程

---

## 一、系统初始化

### 1.1 内核升级（AMD 395 必备）

AMD Ryzen AI MAX+ 395 需要较新的内核才能完整支持 iGPU 和 ROCm。

```bash
# 查看当前内核
uname -r

# Ubuntu 24.04 推荐 HWE-edge 内核
sudo apt install linux-generic-hwe-24.04-edge
sudo apt install linux-headers-$(uname -r)

# 重启后验证
uname -r  # 应显示 6.17.0-xx 或更高

# 查看已安装的所有内核
dpkg --list | grep linux-image
```

### 1.2 iGPU 显存分配

**BIOS 设置**：Advanced → iGPU Memory Size → **512MB**

**GRUB 参数**（让系统分配 120GB 给 GPU）：

```bash
sudo nano /etc/default/grub
```

找到 `GRUB_CMDLINE_LINUX_DEFAULT` 行，添加：

```
ttm.pages_limit=30720000 amdgpu.gttsize=120000
```

更新 GRUB 并重启：

```bash
sudo update-grub
sudo reboot
```

验证：

```bash
sudo dmesg | grep "amdgpu.*memory"
# 应显示: 120000M of GTT memory ready
```

### 1.3 ROCm 安装（AMD GPU 推理加速）

```bash
ROCM_VERSION=7.1
wget https://repo.radeon.com/amdgpu-install/${ROCM_VERSION}/ubuntu/$(lsb_release -cs)/amdgpu-install_${ROCM_VERSION}.70100-1_all.deb
sudo apt install ./amdgpu-install_${ROCM_VERSION}.70100-1_all.deb
sudo amdgpu-install -y --usecase=rocm --no-dkms
sudo usermod -a -G render,video $USER
# 重新登录生效
```

---

## 二、基础开发工具

### 2.1 Anaconda

```bash
wget https://repo.anaconda.com/archive/Anaconda3-2025.06-0-Linux-x86_64.sh
bash Anaconda3-2025.06-0-Linux-x86_64.sh
# 安装过程中选 yes 初始化 conda

# 常用命令
conda create -n quant python=3.12
conda activate quant
conda install numpy pandas scipy matplotlib
```

### 2.2 Git

```bash
sudo apt install git

# 批量提交一个项目的多个文件
git add file1.py file2.py
git commit -m "feat: add factor pipeline"

# 克隆失败排查
git clone --depth 1 <url>            # 浅克隆，加速
GIT_CURL_VERBOSE=1 git clone <url>   # 调试网络问题
```

### 2.3 Docker

```bash
# 安装前置依赖
sudo apt install ca-certificates curl gnupg lsb-release

# 添加 Docker 官方源后安装
sudo apt install docker-ce docker-ce-cli containerd.io

# 如果 .deb 包提示依赖缺失
sudo apt --fix-broken install

# 确认 KVM 支持
kvm-ok
```

### 2.4 中断安装的修复

```bash
# 清理缓存
sudo apt clean && sudo apt autoclean

# 修复中断的安装
sudo dpkg --configure -a
sudo apt --fix-broken install

# 清理孤儿依赖
sudo apt autoremove --purge
```

---

## 三、核心推理工具

### 3.1 LM Studio（主力推理前端）

```bash
# 下载 AppImage
chmod +x LM-Studio-*.AppImage

# 移动到标准目录
mkdir -p ~/Applications
mv LM-Studio-*.AppImage ~/Applications/

# 后台启动 API 服务
nohup ~/Applications/LM-Studio-*.AppImage \
    --server --port 1234 --host 0.0.0.0 \
    --no-sandbox > /tmp/lmstudio.log 2>&1 &

# 验证
curl http://localhost:1234/v1/models
```

**桌面图标修复**（图标不显示时）：

```bash
# 找到 .desktop 文件
find ~/.local/share/applications -name "*.desktop" | xargs grep -l "LM Studio"

# 编辑 Icon 路径为绝对路径
nano ~/.local/share/applications/lm-studio.desktop

# 刷新图标缓存
gtk-update-icon-cache -f -t ~/.local/share/icons/hicolor/
# Alt+F2 → r → 回车（重启 GNOME Shell）
```

### 3.2 Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh

# 查看配置
ollama list          # 已安装模型
ollama ps            # 运行中的模型
ollama show <模型名>  # 模型详情
```

### 3.3 llama.cpp RPC（分布式推理）

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

---

## 四、Trae IDE 更新与维护

Trae 基于 VS Code，更新时直接覆盖安装即可保留所有设置：

```bash
# 下载最新 .deb → 覆盖安装
sudo dpkg -i trae-latest.deb

# 配置保存在这里（卸载前备份）
~/.config/Trae/           # 用户设置
~/.trae/extensions/       # 扩展

# 卸载
sudo dpkg -r trae
```

---

## 五、踩坑记录

| 问题 | 根因 | 解决 |
|------|------|------|
| LM Studio 图标不显示 | AppImage 的 Icon 路径无效 | 手动指定绝对路径 + 刷新图标缓存 |
| Docker Desktop 依赖缺失 | 未安装前置包 | `sudo apt --fix-broken install` |
| apt 安装中断后无法继续 | 缓存锁定 | `sudo dpkg --configure -a` |
| 内核升级后 ROCm 不识别 | 缺少 headers | `sudo apt install linux-headers-$(uname -r)` |
| GNOME 自动关机 | session 退出触发 | `systemctl --user mask shutdown.target`（临时） |

---

> 本文素材提取自笔者与 DeepSeek 的多轮技术对话（2026.01-06），涵盖 Ubuntu 24.04 环境搭建的实际操作过程。