# 客户端配置

## ⚠️ 重要

- **不用 REALITY**：macOS/Linux uTLS 不兼容
- **Clash Verge Rev 会覆盖配置**：需用 launchd 自动修复（见 `references/clash-verge-fix.md`）
- **GEOIP 需要 Country.mmdb**：手动下载到 clash 配置目录

## 获取证书 SHA256 指纹

```bash
# SSH 登录 VPS 后执行
openssl x509 -in /etc/xray-cert.pem -noout -fingerprint -sha256 | cut -d= -f2 | tr -d ':'
```

记录这个 64 位十六进制字符串，用于客户端 `pinnedPeerCertSha256`。

## 方案 1：Mihomo（命令行，推荐）

安装：`brew install mihomo`

配置模板 `~/.config/mihomo/config.yaml`：

```yaml
mixed-port: 7897
port: 7890
socks-port: 7891
allow-lan: false
mode: Rule
log-level: info
ipv6: false

proxies:
  - name: "VPS-XTLS"
    type: vless
    server: 你的VPS_IP
    port: 443
    uuid: 你的UUID
    network: tcp
    tls: true
    flow: xtls-rprx-vision
    servername: www.microsoft.com
    skip-cert-verify: true

proxy-groups:
  - name: "Proxy"
    type: select
    proxies:
      - "VPS-XTLS"
      - "DIRECT"

rules:
  # 国外网站 → 代理
  - DOMAIN-SUFFIX,google.com,Proxy
  - DOMAIN-SUFFIX,youtube.com,Proxy
  - DOMAIN-SUFFIX,github.com,Proxy
  - DOMAIN-SUFFIX,openai.com,Proxy
  - DOMAIN-SUFFIX,chatgpt.com,Proxy
  - DOMAIN-SUFFIX,claude.ai,Proxy
  - DOMAIN-SUFFIX,anthropic.com,Proxy
  - DOMAIN-SUFFIX,twitter.com,Proxy
  - DOMAIN-SUFFIX,x.com,Proxy
  - DOMAIN-SUFFIX,facebook.com,Proxy
  - DOMAIN-SUFFIX,instagram.com,Proxy
  - DOMAIN-SUFFIX,telegram.org,Proxy
  # 国内网站 → 直连
  - DOMAIN-SUFFIX,cn,DIRECT
  - DOMAIN-SUFFIX,baidu.com,DIRECT
  - DOMAIN-SUFFIX,bilibili.com,DIRECT
  - DOMAIN-SUFFIX,qq.com,DIRECT
  - DOMAIN-SUFFIX,taobao.com,DIRECT
  - DOMAIN-SUFFIX,jd.com,DIRECT
  - DOMAIN-SUFFIX,aliyun.com,DIRECT
  - DOMAIN-SUFFIX,weibo.com,DIRECT
  - DOMAIN-SUFFIX,douyin.com,DIRECT
  # 默认走代理
  - MATCH,Proxy
```

启动：
```bash
nohup mihomo -d ~/.config/mihomo > /tmp/mihomo.log 2>&1 &
```

## 方案 2：Xray 客户端（进阶）

配置模板 `~/.xray/config.json`：

```json
{
  "log": {"loglevel": "warning"},
  "inbounds": [{"port": 10808, "protocol": "socks", "settings": {"auth": "noaccounts", "udp": true}}],
  "outbounds": [{
    "protocol": "vless",
    "settings": {
      "vnext": [{"address": "VPS_IP", "port": 443, "users": [{"id": "UUID", "flow": "xtls-rprx-vision", "encryption": "none"}]}]
    },
    "streamSettings": {
      "network": "tcp", "security": "tls",
      "tlsSettings": {"serverName": "www.microsoft.com", "pinnedPeerCertSha256": "证书SHA256指纹"}
    }
  }]
}
```

启动：`nohup xray run -c ~/.xray/config.json > /tmp/xray-client.log 2>&1 &`

## 方案 3：Clash Verge Rev（GUI）

1. 下载：https://github.com/clash-verge-rev/clash-verge-rev/releases
2. 安装 macOS dmg
3. 将 Mihomo 的 YAML 配置复制到：
   `~/Library/Application Support/io.github.clash-verge-rev.clash-verge-rev/clash-verge.yaml`
4. 重启 Clash Verge Rev

## macOS 系统代理设置

```bash
# 启用 SOCKS5 代理
networksetup -setsocksfirewallproxy Wi-Fi 127.0.0.1 7891
networksetup -setsocksfirewallproxystate Wi-Fi on

# 关闭（恢复）
networksetup -setsocksfirewallproxystate Wi-Fi off
```

## GEOIP 支持

```bash
# 下载中国 IP 数据库
curl -sL "https://github.com/Loyalsoldier/geoip/raw/release/Country-only-cn-private.mmdb" \
  -o ~/.config/mihomo/Country.mmdb
```

然后在规则中加 `- GEOIP,CN,DIRECT` 即可。

## 验证

```bash
curl -x socks5://127.0.0.1:7891 -s -o /dev/null -w "HTTP %{http_code} %{time_total}s\n" https://www.google.com
# 期望：HTTP 200

# 国内走直连应很快
curl -x socks5://127.0.0.1:7891 -s -o /dev/null -w "HTTP %{http_code} %{time_total}s\n" https://www.baidu.com
# 期望：HTTP 200 + 速度快
```
