---
name: self-host-proxy
description: 自建科学上网代理方案。当用户提到"翻墙"、"科学上网"、"自建VPN"、"VPS代理"、"搭建节点"、"自建梯子"、"代理加速"、"Hysteria"、"VLESS"或想从商业VPN迁移到自建时使用。涵盖VPS选购、VLESS+TLS+XTLS+BBR服务端、Hysteria2备用节点、Clash Verge Rev/Mihomo客户端、GEOIP分流、故障排查。成本¥158/年起。
---

# 自建科学上网代理

低成本、稳定、双协议冗余的自建代理方案。

## 最终架构

```
用户设备 ─→ Clash Verge Rev / Mihomo
                ├─ VLESS+XTLS (443)  ⭐ 主力
                └─ Hysteria2 (8443)   🔄 备用
                        ↓
               RackNerd VPS (¥158/年)
```

## ⚠️ 关键教训

1. **REALITY 跨平台不兼容**：macOS/Linux uTLS 库差异 → `handshake did not complete successfully`
2. **用 VLESS + 自签 TLS + XTLS Vision 替代**
3. **Hysteria 2 与 VLESS 速度持平**（RackNerd 线路丢包不高），留作备用节点
4. **Clash Verge Rev 启动覆盖配置** → 需 API 强制重载 + launchd 守护
5. **GEOIP MMDB 需手动下载**，Clash/Mihomo 无法自动获取

## 快速部署

### 1. VPS（RackNerd $21.99/年）

👉 references/vps-guide.md

### 2. 服务端（约 10 分钟）

👉 references/server-setup.md

```
1. 装 Xray → 2. 自签证书 → 3. VLESS+TLS+XTLS 配置 → 4. BBR+TFO → 5. systemd 自启
```

### 3. Hysteria 2 备用（可选，约 3 分钟）

👉 references/hysteria2-setup.md

```bash
curl -L https://github.com/apernet/hysteria/releases/latest/download/hysteria-linux-amd64 -o /usr/local/bin/hysteria
# 配置端口 8443/UDP → systemd 自启
```

### 4. 客户端

👉 references/client-setup.md  
👉 references/clash-verge-fix.md（配置持久化）

Clash Verge Rev 含双节点：
```yaml
proxies:
  - RackNerd-VLESS   # 主力 443
  - RackNerd-HY2     # 备用 8443
```

### 5. 验证

```bash
curl -x socks5://127.0.0.1:7891 https://www.google.com  # 应 200
```

## 故障诊断

👉 references/troubleshooting.md
