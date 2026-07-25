# 故障排查

## 诊断流程

按顺序执行，找出问题所在：

### 1. VPS 可达性

```bash
# 从本机测试
nc -z VPS_IP 443
# 期望：Connection succeeded
```

如果失败 → VPS 防火墙问题或 IP 被墙

### 2. Xray 服务状态

```bash
# SSH 登录 VPS 后
systemctl status xray-vless
# 期望：active (running)
```

如果 inactive → `systemctl start xray-vless`

### 3. 端口监听

```bash
ss -tlnp | grep 443
# 期望：xray 监听 0.0.0.0:443
```

如果没有 → 检查 `/usr/local/etc/xray/config.json` 是否正确

### 4. 客户端日志

Xray 客户端：
```bash
tail /tmp/xray-client.log
```

Mihomo：
```bash
tail /tmp/mihomo.log
```

### 5. 服务端日志

```bash
# SSH VPS
tail /tmp/xray.log
```

## 常见问题

### REALITY 握手失败

**现象：** 客户端报 `EOF`，服务端报 `handshake did not complete successfully`

**原因：** REALITY 协议在 macOS 与 Linux 跨平台时 uTLS 库不兼容。

**解决：** 不用 REALITY，改用自签 TLS + XTLS Vision。见 [server-setup.md](./server-setup.md)。

### 连接超时

**现象：** HTTP 000，curl 卡在连接阶段

**排查：**
1. 确认 VPS 可达（nc 测试）
2. 检查是否被本地 VPN 拦截（查看路由：`route get VPS_IP`）
3. 尝试非常规端口

### Mihomo GEOIP 规则导致启动失败

**现象：** Mihomo 卡在 "Can't find MMDB, start download" 不启动

**解决：** 去掉 `GEOIP,CN,DIRECT` 规则，用域名匹配替代。或手动下载：
```bash
curl -L https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geoip.dat -o ~/.config/mihomo/geoip.dat
```

### XTLS only supports TLS and REALITY

**现象：** 客户端报 `XTLS only supports TLS and REALITY directly for now`

**原因：** 使用了 `flow: xtls-rprx-vision` 但 `security: none`

**解决：** 必须设置 `security: tls` 并配置证书，或移除 `flow` 字段。

### allowInsecure 已移除

**现象：** `The feature "allowInsecure" has been removed`

**解决：** Xray 26.x 中需改用：
1. `skip-cert-verify: true`（Mihomo 客户端）
2. `pinnedPeerCertSha256`（Xray 客户端，需要证书 SHA256 指纹）

### 系统代理不生效

**现象：** 浏览器不走代理

**排查：**
1. 确认 Mihomo 运行：`lsof -i :7891`
2. 确认系统代理已设：`networksetup -getsocksfirewallproxy Wi-Fi`
3. 部分浏览器需手动设置代理
4. 重启浏览器

### 速度慢

1. 确认已启用 BBR + TFO：`sysctl net.ipv4.tcp_congestion_control`（应为 bbr）；`sysctl net.ipv4.tcp_fastopen`（应为 3）
2. 确认使用了 XTLS Vision（`flow: xtls-rprx-vision`）
3. ⚠️ 不要调 TCP 缓冲区参数 —— 实测 `rmem/wmem` 会反而变慢
4. 晚高峰切 Hysteria 2（QUIC 抗丢包）
5. 考虑换 VPS 线路（RackNerd → HostDare CN2 GIA）

### Hysteria 2 不生效

1. 确认 UDP 端口：`ss -ulnp | grep 8443`
2. 确认防火墙开了 UDP：`ufw status | grep 8443/udp`
3. Mihomo 配置中 `type: hysteria2`（不是 `hysteria`）
4. 不需要 `up` 和 `down` 带宽参数（自动协商）

### 国内网站走代理了

1. 确认规则有 `GEOIP,CN,DIRECT`
2. 确认 `Country.mmdb` 文件存在
3. 手动加域名规则：`DOMAIN-SUFFIX,xxx.com,DIRECT`
