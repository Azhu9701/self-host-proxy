# 日常维护

## 定期检查（建议每月）

```bash
# SSH 登录
ssh root@VPS_IP

# 系统更新
apt update && apt upgrade -y

# 检查 Xray 状态
systemctl status xray-vless

# 查看端口
ss -tlnp | grep 443

# 查看日志
journalctl -u xray-vless -n 20 --no-pager
```

## 更新 Xray

```bash
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
systemctl restart xray-vless
```

## 换 UUID（安全更新）

```bash
# 生成新 UUID
/usr/local/bin/xray uuid

# 编辑配置文件
nano /usr/local/etc/xray/config.json
# 替换 clients[].id 字段

# 重启服务
systemctl restart xray-vless

# ⚠️ 记得同步更新所有客户端配置
```

## 换证书

```bash
# 证书过期或想更新
openssl req -x509 -newkey rsa:4096 \
  -keyout /etc/xray-key.pem \
  -out /etc/xray-cert.pem \
  -days 3650 -nodes \
  -subj "/CN=www.microsoft.com"

# 获取新 SHA256
openssl x509 -in /etc/xray-cert.pem -noout -fingerprint -sha256 | cut -d= -f2 | tr -d ':'

systemctl restart xray-vless
# ⚠️ 更新客户端 pinnedPeerCertSha256
```

## 监控流量

- **RackNerd**：登录控制面板 `my.racknerd.com` → 查看带宽使用
- **HostDare**：登录控制面板查看
- **通过 3X-UI 面板**：`http://VPS_IP:端口/路径` 查看流量

## 关闭 Ubuntu 自动更新（防止意外重启）

```bash
systemctl disable --now apt-daily.timer
systemctl disable --now apt-daily-upgrade.timer
systemctl disable --now unattended-upgrades.service
```

## 备份

只需备份配置文件：
```bash
# 备份
scp root@VPS_IP:/usr/local/etc/xray/config.json ./xray-config-backup.json

# 恢复
scp ./xray-config-backup.json root@VPS_IP:/usr/local/etc/xray/config.json
ssh root@VPS_IP "systemctl restart xray-vless"
```
