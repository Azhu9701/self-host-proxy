# Clash Verge Rev 配置修复

Clash Verge Rev 启动时会覆盖 `clash-verge.yaml` 为默认配置。需要自动重新注入。

## 原理

1. Clash Verge Rev 启动 → 覆盖 config 为默认空配置
2. 监控脚本检测到文件变化 → 8秒后写入完整 config
3. 通过 Unix Socket API 强制重载

## 部署步骤

### 1. 创建配置注入脚本

```bash
cat > ~/bin/clash-config-fix.sh << 'SCRIPT'
#!/bin/bash
sleep 8
cat > "$HOME/Library/Application Support/io.github.clash-verge-rev.clash-verge-rev/clash-verge.yaml" << 'FULL'
mode: rule
mixed-port: 7897
socks-port: 7891
allow-lan: false
log-level: info
ipv6: false
external-controller: ""
unified-delay: true
external-controller-unix: /tmp/verge/verge-mihomo.sock

proxies:
  - name: "VPS-VLESS"
    type: vless
    server: VPS_IP
    port: 443
    uuid: UUID
    network: tcp
    tls: true
    flow: xtls-rprx-vision
    servername: www.microsoft.com
    skip-cert-verify: true

proxy-groups:
  - name: "Proxy"
    type: select
    proxies:
      - "VPS-VLESS"
      - "DIRECT"

rules:
  - DOMAIN-SUFFIX,cn,DIRECT
  - DOMAIN-SUFFIX,baidu.com,DIRECT
  - DOMAIN-SUFFIX,bilibili.com,DIRECT
  - DOMAIN-SUFFIX,qq.com,DIRECT
  - DOMAIN-SUFFIX,taobao.com,DIRECT
  - DOMAIN-SUFFIX,jd.com,DIRECT
  - DOMAIN-SUFFIX,aliyun.com,DIRECT
  - DOMAIN-SUFFIX,douyin.com,DIRECT
  - DOMAIN-SUFFIX,weibo.com,DIRECT
  - GEOIP,CN,DIRECT
  - IP-CIDR,192.168.0.0/16,DIRECT
  - IP-CIDR,10.0.0.0/8,DIRECT
  - DOMAIN-SUFFIX,google.com,Proxy
  - DOMAIN-SUFFIX,youtube.com,Proxy
  - DOMAIN-SUFFIX,github.com,Proxy
  - DOMAIN-SUFFIX,openai.com,Proxy
  - DOMAIN-SUFFIX,chatgpt.com,Proxy
  - DOMAIN-SUFFIX,claude.ai,Proxy
  - DOMAIN-SUFFIX,twitter.com,Proxy
  - DOMAIN-SUFFIX,x.com,Proxy
  - MATCH,Proxy

tun:
  enable: false
FULL

curl -s --unix-socket /tmp/verge/verge-mihomo.sock -X PUT "http://localhost/configs?force=true" -H "Content-Type: application/json" -d "{\"path\":\"$HOME/Library/Application Support/io.github.clash-verge-rev.clash-verge-rev/clash-verge.yaml\"}" 2>/dev/null
SCRIPT

chmod +x ~/bin/clash-config-fix.sh
```

### 2. 创建 launchd 守护

```bash
cat > ~/Library/LaunchAgents/com.clash.config-fix.plist << 'PLIST'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.clash.config-fix</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>/Users/你的用户名/bin/clash-config-fix.sh</string>
    </array>
    <key>WatchPaths</key>
    <array>
        <string>/Users/你的用户名/Library/Application Support/io.github.clash-verge-rev.clash-verge-rev/clash-verge.yaml</string>
    </array>
    <key>ThrottleInterval</key>
    <integer>10</integer>
</dict>
</plist>
PLIST

launchctl load ~/Library/LaunchAgents/com.clash.config-fix.plist
```

### 3. GEOIP MMDB 文件

```bash
curl -sL "https://github.com/Loyalsoldier/geoip/raw/release/Country-only-cn-private.mmdb" \
  -o "$HOME/Library/Application Support/io.github.clash-verge-rev.clash-verge-rev/Country.mmdb"
```

### 验证

```bash
# 检查 launchd
launchctl list | grep clash

# 检查节点
curl -s --unix-socket /tmp/verge/verge-mihomo.sock http://localhost/proxies | python3 -c "
import sys,json
for k,v in json.load(sys.stdin)['proxies'].items():
    print(f'{k}: {v[\"type\"]}')
"
```
