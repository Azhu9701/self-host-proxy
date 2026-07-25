# 服务端部署 (Xray VLESS + TLS + XTLS Vision)

## 前置条件

- 已购买 VPS（推荐 Ubuntu 22.04 LTS）
- 已通过 SSH 登录：`ssh root@VPS_IP`

## 步骤 1：一键安装 3X-UI（可选但有 Web 管理面板）

```bash
# 停止 SSH 密码复用（首次需要先修改 root 密码或使用密钥）

# 非交互安装（推荐）
XUI_NONINTERACTIVE=1 XUI_USERNAME=admin XUI_PASSWORD=你的密码 XUI_PANEL_PORT=2053 bash <(curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh)
```

面板地址：`http://VPS_IP:2053/随机路径`

## 步骤 2：生成自签 TLS 证书

REALITY 在 macOS 与 Linux 间有 uTLS 兼容问题，改用自签 TLS + XTLS Vision：

```bash
openssl req -x509 -newkey rsa:4096 \
  -keyout /etc/xray-key.pem \
  -out /etc/xray-cert.pem \
  -days 3650 -nodes \
  -subj "/CN=www.microsoft.com"
```

## 步骤 3：部署 Xray 服务端配置

```bash
# 安装 Xray（如果没有）
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install

# 写入配置
cat > /usr/local/etc/xray/config.json << 'EOF'
{
  "log": {"loglevel": "warning"},
  "inbounds": [{
    "port": 443,
    "protocol": "vless",
    "settings": {
      "clients": [{"id": "替换为你的UUID", "flow": "xtls-rprx-vision", "email": "user@example.com"}],
      "decryption": "none"
    },
    "streamSettings": {
      "network": "tcp",
      "security": "tls",
      "tlsSettings": {
        "certificates": [{"certificateFile": "/etc/xray-cert.pem", "keyFile": "/etc/xray-key.pem"}]
      }
    },
    "sniffing": {"enabled": true, "destOverride": ["http", "tls"]}
  }],
  "outbounds": [{"protocol": "freedom", "tag": "direct"}]
}
EOF
```

生成 UUID：`/usr/local/bin/xray uuid`

## 步骤 4：启用 BBR 加速

```bash
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p

# 验证
sysctl net.ipv4.tcp_congestion_control  # 应输出 bbr
```

## 步骤 5：设置 systemd 开机自启

```bash
cat > /etc/systemd/system/xray-vless.service << 'EOF'
[Unit]
Description=Xray VLESS Service
After=network.target

[Service]
ExecStart=/usr/local/bin/xray run -c /usr/local/etc/xray/config.json
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable xray-vless
systemctl start xray-vless
systemctl status xray-vless  # 验证 active
```

## 步骤 6：配置防火墙

```bash
ufw allow 22/tcp
ufw allow 443/tcp
ufw allow 443/udp
ufw --force enable
ufw status verbose
```

## 验证部署

```bash
ss -tlnp | grep 443  # 应看到 xray 监听
systemctl status xray-vless  # 应显示 active
```

## 获取证书 SHA256（客户端需要）

```bash
openssl x509 -in /etc/xray-cert.pem -noout -fingerprint -sha256 | cut -d= -f2 | tr -d ':'
```
