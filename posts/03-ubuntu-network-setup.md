---
title: "Ubuntu网络配置实战：Clash代理 + Samba共享 + USB4双机互联"
date: 2026-06-08
tags:
  - Ubuntu
  - 网络配置
  - Clash
  - Samba
  - USB4
  - 博客
aliases:
  - 网络配置
  - 代理
  - 双机互联
---

# Ubuntu 网络配置实战：代理 + 共享 + 双机互联

> 量化研究者多台设备间的网络基础设施搭建

---

## 一、Clash 代理配置

### 1.1 安装

```bash
# 从 GitHub Releases 下载 clash-linux-amd64
wget https://github.com/.../clash-linux-amd64-xxx.gz
gunzip clash-linux-amd64-xxx.gz
chmod +x clash-linux-amd64-xxx
sudo mv clash-linux-amd64-xxx /usr/local/bin/clash
```

### 1.2 配置

```bash
mkdir -p ~/.config/clash
# 将订阅配置或手动配置放入 ~/.config/clash/config.yaml
```

### 1.3 系统代理设置

GNOME 设置 → 网络 → 代理 → 手动：

| 协议 | 主机 | 端口 |
|:---|:---|:---|
| SOCKS5 | 127.0.0.1 | 7890 |
| HTTP | 127.0.0.1 | 7890 |

### 1.4 开机自启（systemd）

```bash
sudo nano /etc/systemd/system/clash.service
```

```ini
[Unit]
Description=Clash Service
After=network.target

[Service]
Type=simple
User=scott-lau
ExecStart=/usr/local/bin/clash -d /home/scott-lau/.config/clash
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable clash --now
```

### 1.5 Web 控制面板

浏览器访问：`http://clash.razord.top`（Clash Premium 内核）

推荐图形化客户端 Clash Verge（支持订阅管理 + 系统代理一键切换）。

---

## 二、Samba 局域网共享

### 2.1 安装与基础配置

```bash
sudo apt install samba
sudo nano /etc/samba/smb.conf
```

文件末尾添加：

```ini
[Research]
path = /home/scott-lau/Research
browseable = yes
read only = no
guest ok = yes
create mask = 0755

[Models]
path = /home/scott-lau/.cache/lm-studio/models
browseable = yes
read only = yes
guest ok = yes
```

### 2.2 设置密码与启动

```bash
sudo smbpasswd -a scott-lau
sudo systemctl restart smbd

# 防火墙放行
sudo ufw allow samba
```

### 2.3 从其他设备访问

- **Windows**：文件资源管理器 → `\\192.168.1.10\Research`
- **macOS**：Finder → 前往 → 连接服务器 → `smb://192.168.1.10`
- **另一台 Ubuntu**：文件管理器 → `smb://192.168.1.10/Research`

---

## 三、USB4 双机互联

### 3.1 物理连接

两台 AMD 395 设备通过 USB4 线缆直连：

```
[设备 A (NEX)] ←── USB4 40Gbps ──→ [设备 B (GTR-Pro)]
```

### 3.2 IP 配置

**设备 A**：
```bash
sudo ip addr add 192.168.100.10/24 dev eno1
```

**设备 B**：
```bash
sudo ip addr add 192.168.100.20/24 dev eno1
```

### 3.3 连通性验证

```bash
# Ping 测试
ping 192.168.100.20

# 找到 USB4 对应的网卡接口名
ip route get 192.168.100.20
# 输出中 dev 后面的名称就是实际网卡

# 带宽测试
iperf3 -s                          # B 设备上启动服务端
iperf3 -c 192.168.100.20 -t 30    # A 设备上测试（实测约 10-11 Gbps）
```

### 3.4 常见坑：Ping 通但 TCP 不通

**现象**：`ping` 正常但 `ssh`/`scp`/`nc` 全部超时。

**排查步骤**：

```bash
# 1. 检查防火墙
sudo iptables -L -n
sudo ufw status

# 2. 临时清空所有防火墙规则测试
sudo iptables -F

# 3. 检查路由
ip route get 192.168.100.20

# 4. 关闭硬件卸载（如有网卡接口名）
sudo ethtool -K eno1 tx off rx off tso off gso off gro off lro off

# 5. 确认 IP 配在了正确的网卡上
ip addr show | grep 192.168.100
```

### 3.5 文件传输

```bash
# SCP（修复后可用）
scp large_file.bin scott-lau@192.168.100.20:/path/to/dest/

# rsync（同步文件夹）
rsync -avz --progress /source/ scott-lau@192.168.100.20:/dest/

# Python HTTP 服务器（简单传文件）
python3 -m http.server 8000
# 另一台设备浏览器访问 http://192.168.100.10:8000
```

---

## 四、多机工作流协作

### 4.1 Syncthing（P2P 实时同步）

```bash
sudo apt install syncthing
systemctl --user enable syncthing --now
# 浏览器打开 http://localhost:8384
```

### 4.2 Git + SSH 代码同步

```bash
# 在设备 A 上创建 bare repo
git init --bare ~/code/repo.git

# 设备 B 克隆
git clone ssh://scott-lau@192.168.100.10/home/scott-lau/code/repo.git
```

---

## 五、关键踩坑

| 问题 | 最终方案 |
|------|----------|
| USB4 直连后只有 Ping 通，TCP 不通 | 检查防火墙 / 确认 IP 配在正确的网卡上 / 关闭硬件卸载 |
| Samba 共享后 Windows 看不到 | `sudo smbpasswd -a 用户名` 设置密码 |
| Clash 代理不生效 | 确认系统代理设置中端口匹配（默认 7890） |
| USB4 实际速度不到 40Gbps | 正常——协议开销 + 线缆质量，实测 10-11 Gbps 属于正常范围 |

---

> 本文素材提取自笔者与 DeepSeek 多轮网络配置排错对话（2026.01-06）。