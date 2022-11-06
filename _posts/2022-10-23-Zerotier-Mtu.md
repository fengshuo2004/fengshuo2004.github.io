---
layout: post
title: ZeroTier 联机我的世界反复卡退的解决方法
---

ZeroTier是一个内网穿透/虚拟局域网工具，可以用来跟局域网以外的人联机打游戏，无需额外的服务器。

## 问题现象

多位玩家在同一个 ZeroTier 网络下玩 Minecraft。大家连入“对局域网开放”（服务端）电脑的IP地址后，可以建立连接，“服主”也能看见登陆消息，但当进入世界开始加载该玩家附近地形时，ping 会骤降导致玩家被卡掉线。

## 原因

ZeroTier 网络适配器指定的 MTU（最大传输单元）太大，导致大流量传输时 TCP 连接挂起。

参见：https://github.com/zerotier/ZeroTierOne/issues/593

## 解决方法

终端内执行：

```bash
curl -X POST "https://my.zerotier.com/api/network/${NETWORK_ID}" -H "Authorization: bearer ${TOKEN}" -d '{"config": {"mtu": 1280}}'
```

把 ZeroTier 网络的全局 MTU 大小调整为1280。