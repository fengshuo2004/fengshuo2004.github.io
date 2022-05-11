---
layout: post
title: 在Debian系Linux下安装JRE8和OpenJFX8
---

注：本文适用于想要玩 <span style="color:red">1.12.2及更旧</span> 版本的 <span style="color:red">带模组</span> Minecraft ，并打算使用 <span style="color:red">HMCL</span> 作为游戏启动器的 <span style="color:red">Debian / Deepin</span> 用户

## 情景重现

在 Debian 10 Buster / Debian 11 Bullseye / Deepin 20.x 等新 Debian 系 Linux 的软件源中，安装 `default-jre` 包实际会安装 `openjdk-11-jre` ，即开源版的 Java 11 运行环境。

众所周知， Minecraft 1.12.2 及以下版本如果安装了 Forge ，就无法用 Java >8 运行。系统中必须装有 Oracle Java 8 或者 OpenJDK 8 才能运行 1.12.2 Forge。

安装 OpenJDK 8 并非难事，一条命令就能搞定。但是我使用的游戏启动器 HMCL 又依赖于 Java FX ，而 Debian 源里的 OpenJFX 只有 Java 11 版本的，不能用于 OpenJDK 8 。

## 解决方法

### 第一步 — 安装 OpenJDK 8

打开终端执行：

```sh
sudo apt install openjdk-8-jre
```

如果你之前安装了其他版本的 OpenJDK ，还需要将刚刚安装的 OpenJDK 8 设置为默认 Java 。执行下面的命令：

```sh
sudo update-alternatives --config java
```

### 第二步 — 下载旧版 OpenJFX DEB包

综上所述，在Debian系各发行版的软件源里， `openjfx` 包都是给 Java 11 用的。我们除了要下载安装 `openjfx` 本身，还要手动安装它的两个依赖：`libopenjfx-java` 和 `libopenjfx-jni` 。Debian Stretch 源包含的恰好是我们想要的 Java 8 版本。

[下载 openjfx_8u141-b14-3_deb9u1_amd64.deb](http://http.us.debian.org/debian/pool/main/o/openjfx/openjfx_8u141-b14-3~deb9u1_amd64.deb)

[下载 libopenjfx-java_8u141-b14-3_deb9u1_all.deb](http://http.us.debian.org/debian/pool/main/o/openjfx/libopenjfx-java_8u141-b14-3~deb9u1_all.deb)

[下载 libopenjfx-jni_8u141-b14-3_deb9u1_amd64.deb](http://http.us.debian.org/debian/pool/main/o/openjfx/libopenjfx-jni_8u141-b14-3~deb9u1_amd64.deb)

### 第三步 — 安装 OpenJFX

按道理到这一步直接用 dpkg 安装就完事了，可惜 dpkg 不争气，跟我抱怨说 `libopenjfx-jni` 缺少依赖！如果再手动解决这些依赖的依赖，鬼知道要搞到猴年马月去了...... 算了！大胆一搏，强行安装！在下载文件夹内右键 — “在终端打开” — 按顺序执行：

```sh
sudo dpkg --ignore-depends=libavcodec57,libavformat57,libicu57 -i ./libopenjfx-jni_8u141-b14-3_deb9u1_amd64.deb
sudo dpkg -i ./libopenjfx-java_8u141-b14-3_deb9u1_all.deb
sudo dpkg -i ./openjfx_8u141-b14-3_deb9u1_amd64.deb
```

### 第四步 — 禁止 OpenJFX 被 APT 更新

我们可不想让包管理器在下次更新时，把咱辛辛苦苦降级的这三个软件包又升级回 Java 11 的版本。执行以下命令会禁止它们自动更新：

```sh
sudo apt-mark hold openjfx libopenjfx-java libopenjfx-jni
```

到此为止！现在再打开 HMCL 应该不会闪退了，Minecraft 1.12.2 Forge 应该也能正常启动。
