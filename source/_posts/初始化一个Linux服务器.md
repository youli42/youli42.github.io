---
title: 初始化一个Linux服务器
date: 2026-03-09 23:51:00
tags: 
- Linux
- 服务器
- Docker
categories: 技术笔记
excerpt: 记录我初始化Linux服务器的完整流程，包含换源、Docker安装和常用容器部署
---

拿到一台全新的 Linux 服务器后，我习惯按照固定流程进行初始化，以确保后续使用稳定且高效。以下是我个人的初始化流程，记录下来方便查阅。

## 一、换源与系统更新

新服务器到手后，第一件事就是更换 apt 镜像源，这样可以大幅提升后续软件安装速度，避免因官方源在国外导致的下载缓慢问题。

### 国内服务器

```bash
# 主地址
bash <(curl -sSL https://linuxmirrors.cn/main.sh)

# GitHub 地址
bash <(curl -sSL https://raw.githubusercontent.com/SuperManito/LinuxMirrors/main/ChangeMirrors.sh)

# Gitee 地址
bash <(curl -sSL https://gitee.com/SuperManito/LinuxMirrors/raw/main/ChangeMirrors.sh)
```

### 海外服务器

海外服务器可以使用 `--abroad` 参数切换为国际镜像源，或者不执行换源操作（因为默认源通常就在当地，速度尚可）：

```bash
# 使用 --abroad 参数切换为国际镜像源
bash <(curl -sSL https://linuxmirrors.cn/main.sh) --abroad

# GitHub 地址
bash <(curl -sSL https://raw.githubusercontent.com/SuperManito/LinuxMirrors/main/ChangeMirrors.sh) --abroad

# Gitee 地址
bash <(curl -sSL https://gitee.com/SuperManito/LinuxMirrors/raw/main/ChangeMirrors.sh) --abroad

# 或者不执行换源，保持默认源
```

> 该脚本会自动检测服务器所在地区并选择合适的镜像源，整个过程交互友好，按提示操作即可。更换完成后，系统会自动执行 `apt update` 更新软件包列表。

---

## 二、安装 Docker 环境

换源完成后，接下来安装 Docker。我通常有两种方式：

### 方式一：在线安装（推荐）

如果换源成功，建议使用国内 Docker 镜像源（如阿里云、清华镜像等）进行安装，可以显著提升下载速度：

```bash
# 安装依赖
apt install -y apt-transport-https ca-certificates curl gnupg lsb-release

# 国内服务器 - 以阿里云为例
## 添加 Docker GPG 密钥
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

## 添加 Docker APT 源（以 Debian 为例）
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://mirrors.aliyun.com/docker-ce/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

# 海外服务器 - 使用官方源
# curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
# echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

# 安装 Docker
apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-buildx-plugin
```

常用的国内 Docker 镜像源：
- 阿里云：`https://mirrors.aliyun.com/docker-ce/linux/debian`
- 清华镜像：`https://mirrors.tuna.tsinghua.edu.cn/docker-ce`

> 注意：根据你的服务器发行版（Debian/Ubuntu）调整命令中的 `debian` 关键字。

### 方式二：手动安装（换源不可用时）

有时国内服务器换源可能失败，或者需要特定版本的 Docker 时，我会选择手动下载 deb 包进行安装。我通常从清华镜像站或 Docker 官方下载以下四个包：

| 包名 | 说明 |
|------|------|
| `containerd.io_xxx_amd64.deb` | containerd 是 Docker 的底层容器运行时，负责管理容器的生命周期（如创建、启动、停止等），是 Docker 运行的核心依赖 |
| `docker-ce_xxx_amd64.deb` | Docker 引擎的核心包，包含 Docker 守护进程（dockerd）等核心功能，是运行 Docker 容器的基础 |
| `docker-ce-cli_xxx_amd64.deb` | Docker 命令行工具，提供 docker 命令（如 docker run、docker ps），用于与 Docker 引擎交互 |
| `docker-compose-plugin_xxx_amd64.deb` | Docker Compose 插件，支持通过 docker compose 命令管理多容器应用 |

```bash
# 下载后使用 dpkg 安装
dpkg -i containerd.io_xxx_amd64.deb docker-ce_xxx_amd64.deb docker-ce-cli_xxx_amd64.deb docker-compose-plugin_xxx_amd64.deb

# 如果有依赖缺失，执行
apt-get install -f
```

安装完成后，验证 Docker 是否正常运行：

```bash
docker --version
docker compose version
```

---

## 三、安装常用 Docker 容器

Docker 环境就绪后，我会使用 Docker Compose 部署几个常用的容器服务。以下是基础的 `docker-compose.yml` 配置：

```yaml
services:
  # ===== easytier 虚拟组网 =====
  # 01
  easytier:
    image: easytier/easytier:v2.5.0 # 国内用户可以使用 m.daocloud.io/docker.io/easytier/easytier:latest
    hostname: easytier
    container_name: easytier
    # labels: # 配置自动更新
    #   com.centurylinklabs.watchtower.enable: 'true'
    restart: always
    network_mode: host
    cap_add:
      - NET_ADMIN # 允许进程执行各种网络管理和配置操作
      - NET_RAW   # 允许进程创建和使用"原始"套接字，以及绑定到透明代理的套接字
    environment:
      - TZ=Asia/Shanghai # 设置时区
    devices:
      - /dev/net/tun:/dev/net/tun
    volumes:
      - ./easytier/config.yaml:/config/config.yaml # 映射配置文件
      # - /etc/machine-id:/etc/machine-id:ro # 映射宿主机机器码
    command: -c /config/config.yaml
    deploy: # 资源限制
      resources:
        limits:  # 硬限制
          memory: '128M'

  # ====== Lucky 主服务 ======
  # 用作反向代理和网络流量控制
  # 03
  lucky:
    image: gdy666/lucky:2.27.2
    container_name: lucky
    user: "1000:1000"
    group_add:
      - "989"
      - "101"
    cap_add: # 新增：添加端口绑定权限
      - NET_BIND_SERVICE
    volumes:
      - ./LuckyConfig/config:/app/conf      # 映射配置文件，新版变化
      - /var/run/docker.sock:/var/run/docker.sock # 映射 docker
      - ./:/home/youli/dockerApp/           # 项目文件目录
    restart: always
    privileged: true
    network_mode: host # 使用主机网络模式
    deploy: # 资源限制
      resources:
        limits:  # 硬限制
          memory: '256M'
```

### 3.1 EasyTier 虚拟组网

EasyTier 是一个轻量级的 WireGuard 替代方案，用于搭建内网穿透或异地组网。

**配置说明：**

| 配置项 | 说明 |
|--------|------|
| `network_mode: host` | 使用主机网络模式，减少网络性能损耗 |
| `cap_add: NET_ADMIN` | 允许进程执行各种网络管理和配置操作 |
| `cap_add: NET_RAW` | 允许进程创建原始套接字，直接操作网络层数据包 |
| `devices: /dev/net/tun` | 映射 TUN 设备，这是创建虚拟网卡所必需的 |
| `volumes` | 映射配置文件，需要在同级目录下创建 `easytier/config.yaml` |

**配置文件示例** (`easytier/config.yaml`)：

```yaml
hostname = "youliLaptop"
instance_name = "youli"
ipv4 = "192.168.2.7/24"
dhcp = false
listeners = [
    "tcp://0.0.0.0:11010",
    "udp://0.0.0.0:11010",
    "wg://0.0.0.0:11011",
    "ws://0.0.0.0:11011/",
    "wss://0.0.0.0:11012/",
]

[network_identity]
network_name = "userName"
network_secret = "PassWD"

[[peer]]
uri = "tcp://public.easytier.top:11010" # 官方服务器，但已经无法使用


[flags]


```

### 3.2 Lucky 主服务

Lucky 是一个多功能网关工具，支持反向代理、动态域名解析、端口转发、流量统计等功能。

**配置说明：**

| 配置项 | 说明 |
|--------|------|
| `user: "1000:1000"` | 以指定用户身份运行，确保文件权限正确 |
| `group_add` | 添加额外的组，确保访问特定资源时有足够权限 |
| `cap_add: NET_BIND_SERVICE` | 允许绑定到低端口号（如 80、443） |
| `network_mode: host` | 使用主机网络模式，便于端口监听和服务访问 |
| `/var/run/docker.sock` | 映射 Docker socket，可以从容器内管理宿主机上的其他容器 |
| `./:/home/youli/dockerApp/` | 将整个项目目录映射到容器内，方便文件管理和配置 |

---

## 四、启动服务

将上述 compose 配置保存为 `docker-compose.yml`，在目录下执行：

```bash
docker compose up -d
```

查看容器运行状态：

```bash
docker compose ps
docker compose logs -f
```

---

## 总结

我的 Linux 服务器初始化流程相对固定：

1. **换源** - 使用 `linuxmirrors.cn` 脚本自动更换 apt 镜像源
2. **安装 Docker** - 在线安装或手动 deb 包安装
3. **部署应用** - 通过 Docker Compose 运行 EasyTier、Lucky 等服务

---

## 使用命令总结

### 1. 换源命令

```bash
# 国内服务器换源（主地址）
bash <(curl -sSL https://linuxmirrors.cn/main.sh)

# 国内服务器换源（GitHub 地址）
bash <(curl -sSL https://raw.githubusercontent.com/SuperManito/LinuxMirrors/main/ChangeMirrors.sh)

# 国内服务器换源（Gitee 地址）
bash <(curl -sSL https://gitee.com/SuperManito/LinuxMirrors/raw/main/ChangeMirrors.sh)

# 海外服务器换源（主地址）
bash <(curl -sSL https://linuxmirrors.cn/main.sh) --abroad

# 海外服务器换源（GitHub 地址）
bash <(curl -sSL https://raw.githubusercontent.com/SuperManito/LinuxMirrors/main/ChangeMirrors.sh) --abroad

# 海外服务器换源（Gitee 地址）
bash <(curl -sSL https://gitee.com/SuperManito/LinuxMirrors/raw/main/ChangeMirrors.sh) --abroad
```

### 2. Docker 安装命令

```bash
# 安装依赖
apt install -y apt-transport-https ca-certificates curl gnupg lsb-release

# 添加 Docker GPG 密钥（国内 - 阿里云）
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 添加 Docker APT 源（国内 - 阿里云）
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://mirrors.aliyun.com/docker-ce/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

# 添加 Docker GPG 密钥（海外 - 官方）
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 添加 Docker APT 源（海外 - 官方）
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

# 更新并安装 Docker
apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-buildx-plugin

# 手动安装（下载 deb 包后）
dpkg -i containerd.io_xxx_amd64.deb docker-ce_xxx_amd64.deb docker-ce-cli_xxx_amd64.deb docker-compose-plugin_xxx_amd64.deb
# 修复依赖
apt-get install -f
```

### 3. Docker 验证命令

```bash
# 查看 Docker 版本
docker --version

# 查看 Docker Compose 版本
docker compose version
```

### 4. Docker Compose 命令

```bash
# 启动容器服务
docker compose up -d

# 查看容器运行状态
docker compose ps

# 查看容器日志（实时）
docker compose logs -f
```

### 5. 获取系统信息命令

```bash
# 获取系统发行版信息
lsb_release -cs
```
