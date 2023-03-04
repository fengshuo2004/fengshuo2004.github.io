---
layout: post
title: ZeroTier 联机《我的世界》反复卡退的解决方法
---

ZeroTier是一个内网穿透/虚拟局域网工具，可以用来跟局域网以外的人联机打游戏，无需额外的服务器。

## 问题现象

多位玩家在同一个 ZeroTier 网络下玩 Minecraft。大家连入“对局域网开放”（服务端）电脑的IP地址后，可以建立连接，“服主”也能看见登陆消息，但当进入世界开始加载该玩家附近地形时，ping 会骤降导致玩家被卡掉线。

## 原因

ZeroTier 网络适配器指定的 MTU（最大传输单元）太大，导致大流量传输时 TCP 连接挂起。

参见：https://github.com/zerotier/ZeroTierOne/issues/593

## 解决方法

### 获取 Network ID 和 API Token

一个 ZeroTier 虚拟网络的 MTU 设定是无法通过网络管理界面更改的，只有调用 API 才可以改。而使用 API 需要你知道自己的 API Token（令牌）：

1. 浏览器访问 https://my.zerotier.com/ ，登录你的账号
2. 把你的 Network ID 复制出去保存起来
3. 访问 https://my.zerotier.com/account 往下滚动到 API Access Tokens 这一行
4. 点击 `New Token`，Token Label 下的文本框允许你给令牌起个名字，默认不用修改，最后点击 `Generate`
5. 浅绿色框里出现的这一长串字母+数字组合就是你的 API Token，复制出去保存起来

### 调用 API

macOS、Linux用户可以在终端内执行：

```bash
curl -X POST "https://my.zerotier.com/api/network/${NETWORK_ID}" -H "Authorization: bearer ${TOKEN}" -d '{"config": {"mtu": 1280}}'
```

把 ZeroTier 网络的全局 MTU 大小调整为1280。

记得把命令里的 `${NETWORK_ID}` 替换成你的 Network ID，`${TOKEN}` 替换成你的 API Token。