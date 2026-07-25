# 🚀 自建科学上网代理 — 全套开源方案

> ¥158/年起 | 双协议冗余 | Clash Verge Rev 客户端 | 一键部署

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Stars](https://img.shields.io/github/stars/huasanai/self-host-proxy?style=social)](https://github.com/huasanai/self-host-proxy)

> 🚀 **GitHub 加速**：[ghproxy 镜像](https://ghproxy.com/https://github.com/Azhu9701/self-host-proxy) | [下载 ZIP](https://ghproxy.com/https://github.com/Azhu9701/self-host-proxy/archive/refs/heads/main.zip)

## ✨ 特点

- 💰 **超低成本**：¥158/年，比商业 VPN 便宜一半
- ⚡ **双协议冗余**：VLESS + XTLS Vision（主力）+ Hysteria 2（备用）
- 🎯 **国内外分流**：GEOIP + 域名规则，国内直连、国外走代理
- 🖥️ **Clash Verge Rev**：GUI 客户端，可视化管理 + 流量监控
- 🔧 **全自动化**：systemd 自启、launchd 配置持久化
- 📚 **ZCode Skill**：内置 AI 技能文件，配新节点说句话就行

## 🏗️ 架构

```
用户设备 → Clash Verge Rev
              ├─ VLESS+XTLS (443)   ⭐ 主力
              └─ Hysteria2 (8443)    🔄 备用
                      ↓
             海外 VPS → 🌐 自由网络
```

## 🛒 推荐 VPS

| 档次 | 推荐 | 月费 | 链接 |
|------|------|:---:|------|
| 🥇 性价比 | **RackNerd** 1G/3TB | ¥13/月 | [👉 立即购买](https://my.racknerd.com/aff.php?aff=20743) |
| 🥈 中国优化 | HostDare CN2 GIA | ¥21/月 | [👉 立即购买](https://hostdare.com) |
| 🥉 顶级线路 | DMIT CN2 GIA | ¥29/月 | [👉 立即购买](https://www.dmit.io) |

> 🔗 RackNerd 购买时自动使用推荐链接，感谢支持！

## 🚀 快速开始

> ⚡ **国内加速**：所有 GitHub 链接可用 `https://ghproxy.com/` 前缀加速，例：`curl https://ghproxy.com/https://raw.githubusercontent.com/...`

### 1. 买 VPS

选 RackNerd $21.99/年套餐 → [点击购买](https://my.racknerd.com/aff.php?aff=20743)

### 2. 一键部署服务端

```bash
ssh root@你的VPS_IP

# 装 Xray
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install

# 生成证书
openssl req -x509 -newkey rsa:4096 -keyout /etc/xray-key.pem \
  -out /etc/xray-cert.pem -days 3650 -nodes -subj "/CN=www.microsoft.com"

# 用 references/configs/xray-server.json 模板写入配置
# 替换 UUID → 开 BBR → systemd 自启
```

详细步骤见 [references/server-setup.md](references/server-setup.md)

### 3. 客户端

下载 [Clash Verge Rev](https://github.com/clash-verge-rev/clash-verge-rev/releases) → 导入配置 → 完成！

配置模板见 [references/client-setup.md](references/client-setup.md)

## 📊 实测速度

| 网站 | VLESS | Hysteria 2 |
|------|:---:|:---:|
| Google | 1.3s | 1.5s |
| YouTube | 2.5s | 3.0s |
| GitHub | 1.9s | 3.4s |

> 测试环境：中国移动 → RackNerd 洛杉矶

## 🤖 AI 技能（ZCode Skill）

本仓库同时是一个 ZCode 技能文件。

你将看到的所有 Markdown 文件都遵循 ZCode 技能规范，供 AI 编程助手直接调用：

```
/self-host-proxy 帮我搭个新节点
```

## 📁 文件结构

```
├── SKILL.md                  # ZCode 技能入口
├── README.md                 # 你正在看的
└── references/
    ├── vps-guide.md          # VPS 选购指南
    ├── server-setup.md       # 服务端详细部署
    ├── hysteria2-setup.md    # Hysteria 2 备用节点
    ├── client-setup.md       # 客户端配置
    ├── clash-verge-fix.md    # Clash Verge Rev 持久化
    ├── troubleshooting.md    # 故障排查全解
    ├── maintenance.md        # 日常维护
    └── configs/
        └── xray-server.json  # 服务端配置模板
```

## 📝 License

MIT — 随意使用、修改、分发。

---

⭐ 如果这个项目对你有帮助，请点个 Star！
