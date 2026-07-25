# Hysteria 2 备用节点部署

Hysteria 2 基于 QUIC（UDP），在丢包严重的网络环境下比 TCP 快 30-50%。与 VLESS 同时运行，互为备份。

## 安装

```bash
# 下载二进制
curl -L https://github.com/apernet/hysteria/releases/latest/download/hysteria-linux-amd64 \
  -o /usr/local/bin/hysteria
chmod +x /usr/local/bin/hysteria

# 生成密码
HY2_PASS=$(openssl rand -base64 16 | tr -d /=+)
echo "密码: $HY2_PASS"
```

## 配置 /etc/hysteria/config.yaml

```yaml
listen: :8443

tls:
  cert: /etc/xray-cert.pem    # 复用 VLESS 的自签证书
  key: /etc/xray-key.pem

auth:
  type: password
  password: 你的密码

masquerade:
  type: proxy
  proxy:
    url: https://www.microsoft.com
    rewriteHost: true
```

## 启动和自启

```bash
# 防火墙
ufw allow 8443/udp

# 手动运行
nohup /usr/local/bin/hysteria server -c /etc/hysteria/config.yaml > /tmp/hysteria.log 2>&1 &

# systemd 自启
cat > /etc/systemd/system/hysteria.service << 'EOF'
[Unit]
Description=Hysteria 2 Service
After=network.target

[Service]
ExecStart=/usr/local/bin/hysteria server -c /etc/hysteria/config.yaml
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable hysteria
systemctl start hysteria
```

## 客户端配置（Mihomo / Clash Verge Rev）

```yaml
proxies:
  - name: "VPS-HY2"
    type: hysteria2
    server: VPS_IP
    port: 8443
    password: 你的密码
    sni: www.microsoft.com
    skip-cert-verify: true
```

添加到 proxy-groups 作为备用节点。

## 验证

```bash
ss -ulnp | grep 8443  # 应看到 hysteria 监听
```

## 速度对比

| 场景 | VLESS (TCP) | Hysteria 2 (QUIC) |
|------|:---:|:---:|
| 正常网络 | ⭐⭐⭐ | ⭐⭐⭐ |
| 丢包 > 5% | ⭐⭐ | ⭐⭐⭐⭐ |
| 晚高峰 | ⭐⭐ | ⭐⭐⭐ |

日常用 VLESS，网络不稳定时切到 Hysteria 2。
