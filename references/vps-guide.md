# VPS 选购指南

## 推荐方案

| 档次 | 推荐 | 价格/年 | 配置 | 线路 | 适合 |
|------|------|:---:|------|------|------|
| 🥇 性价比 | **RackNerd** | $21.99 (¥158) | 1C1G / 20G SSD / 3TB | 普通 Cogent/HE | 日常浏览、视频 |
| 🥈 稳定 | **HostDare CN2 GIA** | $35.99 (¥258) | 1C768M / 10G / 500G | CN2 GIA 优化 | 稳定性要求高 |
| 🥉 顶配 | **DMIT** | $48.88 (¥350) | 1C1G / 10G / 1.5TB | CN2 GIA 顶级 | 追求极致 |

## RackNerd 购买

1. 打开 https://www.racknerd.com/specials/
2. 选 1 GB KVM VPS ($21.99/year)
3. 机房选 **Los Angeles**（对中国延迟最低）
4. 年付（Annually）
5. 支持支付宝

## HostDare 购买

1. 打开 https://hostdare.com
2. 选 CN2 GIA 系列（CSSD/CKVM，标有 "China Optimized"）
3. 年付

## 购买后检查

```bash
# SSH 登录
ssh root@VPS_IP

# 确认系统版本
lsb_release -a  # 应为 Ubuntu 22.04 或 24.04

# 测试基本连通性
ping -c 3 google.com
```
