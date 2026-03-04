---
title: 白嫖 Cloudflare：家庭服务器的 DDNS 与内网穿透一站式方案
created_time: 2026-03-05T00:00:00.000Z
last_edited_time: 2026-03-05T00:00:00.000Z
tags:
  - Homelab
  - Cloudflare
  - Docker
  - Self-hosted
date: '2026-03-05'
layout: single
---

家里有一台 NAS 或者旧电脑想跑点服务？国内家宽两座大山：动态 IP 和端口封锁。这篇文章记录我是怎么用 Cloudflare 的免费套餐，绕开这两个问题，把家里的服务暴露到公网的。

<!--more-->

## 背景：家宽建站的经典困境

在 Bilibili 看到过一篇[家用服务器建站指南](https://www.bilibili.com/read/cv15787877/)，作者买了台 DELL R620 想自建服务，然后踩遍了国内家宽的所有坑：

1. **动态公网 IP**：运营商会不定期变更你的公网 IP，域名解析随时失效
2. **端口封锁**：80、443、8080 这些常用端口全部封禁，用户必须在域名后面加端口号才能访问（`example.com:5555`）
3. **搜索引擎不友好**：非标准端口域名对 SEO 不友好

作者最后的结论是："追求稳定、速度的站长，去买云服务器即可"——维护成本、电费、噪音、停电风险算下来，云服务器反而更划算。

这个结论放在 2022 年是对的。但现在有了更好的路：**Cloudflare 免费套餐**。

---

## 核心思路：不依赖公网 IP，不依赖端口转发

传统方案的链路是：

```
用户 → DNS 解析到家里公网 IP → 路由器端口转发 → 内网服务
```

这条路上有三个卡点：动态 IP 需要 DDNS 维护、端口转发要配路由器、443/80 可能被封。

Cloudflare Tunnel 的链路完全不同：

```
用户 → Cloudflare Edge → Tunnel → 家里的 cloudflared → 内网服务
```

关键区别：**连接是从家里主动发起到 Cloudflare 的**，不需要开放任何入站端口，路由器不需要任何配置，公网 IP 变了也没关系。

---

## 能解决的问题

### 1. DDNS（动态域名解析）

Cloudflare 提供免费域名（`*.dpdns.org`），同时提供 DNS API。用 `favonia/cloudflare-ddns` 这个 Docker 镜像，每 5 分钟自动检测公网 IP 并更新 A 记录。

适用场景：SSH 远程登录家里机器、远程桌面、需要直连的服务。

### 2. 内网穿透（Cloudflare Tunnel）

不需要公网 IP，不需要端口转发，**甚至不需要路由器支持**。

`cloudflared` 容器在家里启动后，向 Cloudflare 建立持久的出站连接。Cloudflare 的全球边缘节点接受用户请求，通过这条隧道转发到家里的具体服务。

用户访问的是标准 443 端口的 HTTPS，证书由 Cloudflare 签发和管理，家里完全不需要 Certbot 或者 Let's Encrypt。

---

## 实现路径

### 前置条件

- Cloudflare 账号（免费）
- 一个 Cloudflare 管理的域名（可以用 CF 赠送的 `*.dpdns.org`）
- 家里有台能跑 Docker 的机器（NAS、树莓派、旧电脑都行）

### 步骤概览

**Step 1：申请 Cloudflare 免费域名**

注册 Cloudflare 账号，在 DNS 页面可以领一个 `*.dpdns.org` 的免费子域名。

**Step 2：创建 Cloudflare Tunnel**

进入 Zero Trust → Networks → Tunnels → Create a tunnel，复制生成的 Token。

**Step 3：配置 Public Hostname**

在 Tunnel 配置里添加路由规则，把子域名指向内网服务：

| Subdomain | 内网地址 |
|-----------|---------|
| `vault.your-domain.dpdns.org` | `http://vaultwarden:80` |
| `jellyfin.your-domain.dpdns.org` | `http://jellyfin:8096` |

**Step 4：Docker Compose 一键启动**

所有服务统一在一个 `docker-compose.yml` 里管理。

---

## 项目结构

```
homelab/
├── docker-compose.yml      # 所有服务定义
├── .env                    # 密钥（不进 git）
├── .env.example            # 变量模板（进 git）
├── cloudflare/
│   └── README.md           # Tunnel 路由说明
└── data/                   # 容器数据目录（不进 git）
    ├── vaultwarden/
    └── alist/
```

`docker-compose.yml` 包含以下服务：

```yaml
services:
  cloudflare-ddns:    # 自动更新 A 记录，解决动态 IP
  cloudflared:        # Tunnel 连接，解决内网穿透
  vaultwarden:        # 密码管理器（Bitwarden 兼容）
  jellyfin:           # 媒体服务器
  alist:              # 网盘聚合
```

---

## 不需要公网 IP 能做到什么

这是这套方案最值得强调的地方：

- ✅ 外网 HTTPS 访问家里服务，URL 干净无端口号
- ✅ 证书自动管理，浏览器无警告
- ✅ 不开路由器端口，内网安全边界完整
- ✅ 运营商换 IP、封端口，对 Tunnel 完全没影响
- ✅ 流量走 Cloudflare 全球 CDN，国内访问延迟还不错

不能做到的：

- ❌ 大流量视频流（CF ToS 禁止代理视频），Jellyfin 建议走直连或单独处理
- ❌ 超低延迟场景（毕竟多了一跳 CF 节点）

---

## 总结

两年前那篇 Bilibili 文章的问题，现在用 Cloudflare 免费套餐全部解决，而且更简单：

| 问题 | 传统方案 | Cloudflare 方案 |
|------|---------|----------------|
| 动态 IP | DDNS 服务（NoIP 等，需定期续期）| CF DDNS，API 驱动，永久免费 |
| 端口封锁 | 换端口 + CDN 隐性解析 | Tunnel 根本不用端口 |
| HTTPS 证书 | Certbot + Let's Encrypt + Nginx | CF 自动签发，无需 Nginx |
| 路由器配置 | 端口转发 | 不需要 |

整套服务跑在 NAS 上，`docker compose up -d` 一键启动，配置文件全部在 Git 里版本控制。Nginx、Certbot 一行都不需要写。
