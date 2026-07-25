# VPS 运维手册

## 快速诊断（出问题先跑这个）

```bash
ssh root@VPS_IP

# 一键检查
echo "=== Xray ===" && systemctl is-active xray-vless
echo "=== Hysteria ===" && systemctl is-active hysteria
echo "=== 443端口 ===" && ss -tlnp | grep 443
echo "=== 8443端口 ===" && ss -ulnp | grep 8443
echo "=== 内存 ===" && free -h | grep Mem
echo "=== 磁盘 ===" && df -h / | tail -1
echo "=== 运行时间 ===" && uptime
echo "=== 最新日志 ===" && journalctl -u xray-vless -n 5 --no-pager
```

## 日常巡检（每周一次，5 分钟）

```bash
ssh root@VPS_IP

# 服务状态
systemctl status xray-vless hysteria --no-pager

# 是否有入侵尝试
grep "Failed password" /var/log/auth.log | tail -5

# 系统负载
top -bn1 | head -5
```

## 查看流量（RackNerd）

登录 https://my.racknerd.com → 查看 Bandwidth Usage

## 日志排查

```bash
# Xray 服务日志（最近 50 行）
journalctl -u xray-vless -n 50 --no-pager

# 实时跟踪日志
journalctl -u xray-vless -f

# 按时间筛选
journalctl -u xray-vless --since "2026-07-25 10:00" --no-pager

# Hysteria 日志
tail -50 /tmp/hysteria.log
```

## 系统更新（每月一次）

```bash
ssh root@VPS_IP
apt update && apt upgrade -y

# 如果更新了内核或核心库，重启
reboot
# 等 2 分钟后验证服务恢复
ssh root@VPS_IP "systemctl status xray-vless hysteria"
```

## 升级 Xray / Hysteria

```bash
# Xray
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
systemctl restart xray-vless

# Hysteria 2
curl -L https://github.com/apernet/hysteria/releases/latest/download/hysteria-linux-amd64 -o /usr/local/bin/hysteria
chmod +x /usr/local/bin/hysteria
systemctl restart hysteria
```

## 换 UUID / 密码（安全轮换）

```bash
# 生成新 UUID
/usr/local/bin/xray uuid
# 输出类似: d0b8b7e0-6c1a-4c2a-8f0a-9a1b2c3d4e5f

# 编辑配置
nano /usr/local/etc/xray/config.json
# 替换 clients[].id 字段

# 重启
systemctl restart xray-vless

# ⚠️ 记得同步更新所有客户端（Mac/手机/路由器）的 UUID
```

## 换 TLS 证书（3650 天有效期，不用担心）

```bash
# 仅当证书泄露或想更新时
openssl req -x509 -newkey rsa:4096 \
  -keyout /etc/xray-key.pem \
  -out /etc/xray-cert.pem \
  -days 3650 -nodes \
  -subj "/CN=www.microsoft.com"

# 获取新指纹（客户端需要）
openssl x509 -in /etc/xray-cert.pem -noout -fingerprint -sha256 | cut -d= -f2 | tr -d ':'

# 重启两个服务
systemctl restart xray-vless
systemctl restart hysteria
# ⚠️ 更新客户端 pinnedPeerCertSha256 或换 skip-cert-verify
```

## 备份

```bash
# 完整备份 - 把以下文件存到本地
scp root@VPS_IP:/usr/local/etc/xray/config.json ./backup/xray-config-$(date +%Y%m%d).json
scp root@VPS_IP:/etc/hysteria/config.yaml ./backup/hysteria-config-$(date +%Y%m%d).yaml
scp root@VPS_IP:/etc/xray-cert.pem ./backup/
scp root@VPS_IP:/etc/xray-key.pem ./backup/

# 恢复
scp ./backup/xray-config-*.json root@VPS_IP:/usr/local/etc/xray/config.json
ssh root@VPS_IP "systemctl restart xray-vless"
```

## 安全加固

```bash
# 1. 禁用 root 密码登录，用 SSH 密钥
ssh-copy-id root@VPS_IP
# 然后编辑 /etc/ssh/sshd_config
# PasswordAuthentication no
# systemctl restart sshd

# 2. 改 SSH 端口（减少扫描）
echo "Port 2222" >> /etc/ssh/sshd_config
ufw allow 2222/tcp
systemctl restart sshd
# 之后登录: ssh root@VPS_IP -p 2222

# 3. 安装 fail2ban
apt install -y fail2ban
systemctl enable fail2ban
```

## 关闭 Ubuntu 自动更新（防止意外重启）

```bash
systemctl disable --now apt-daily.timer
systemctl disable --now apt-daily-upgrade.timer
systemctl disable --now unattended-upgrades.service

# 验证
systemctl list-timers | grep apt
```

## 各服务端口对照表

| 端口 | 协议 | 服务 | 用途 |
|:---:|------|------|------|
| 22 | TCP | sshd | SSH 管理 |
| 443 | TCP | xray | VLESS 主力代理 |
| 8443 | UDP | hysteria | Hysteria 2 备用 |
| 2053 | TCP | x-ui | 3X-UI 面板（可选） |

## 告警信号

| 现象 | 可能原因 | 操作 |
|------|------|------|
| 443 端口掉了 | Xray 挂了 | `systemctl restart xray-vless` |
| 8443 UDP 掉了 | Hysteria 挂了 | `systemctl restart hysteria` |
| SSH 连不上 | VPS 重启/网络故障 | 等 2 分钟重试 |
| 网速突然变慢 | 晚高峰/邻居滥用 | 切 HY2，或换 VPS |
| 流量突然暴涨 | 被扫/泄露 | 立刻换 UUID |
| auth.log 大量失败登录 | 被暴力破解 | 改 SSH 端口 + fail2ban |
| 磁盘满了 | 日志堆积 | `journalctl --vacuum-size=200M` |
