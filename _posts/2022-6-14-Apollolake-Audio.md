---
layout: post
title: 在搭载Apollolake处理器的Chromebook上获得Linux声卡驱动
---

搭载英特尔 Apollolake 处理器的 Chromebook 在刷入完整 UEFI 固件后安装 Linux 会没有声音，这是因为 SOF 驱动不识别处理器内置声卡。目前没有完美的解决方法，但有一种非官方“补丁”至少能让笔记本出声。

## 步骤

1. 下载 [sof-apl-da7219.tplg]({{ site.baseurl }}/resources/sof-apl-da7219.tplg) 文件

2. 在下载文件夹中打开终端

3. 按顺序运行以下命令

    ```sh
    sudo apt install alsa-topology* alsa-ucm* linux-firmware
    sudo cp sof-apl-da7219.tplg /lib/firmware/intel/sof-tplg/
    sudo sh -c 'echo "options snd_intel_dspcfg dsp_driver=3" > /etc/modprobe.d/inteldsp.conf' 
    sudo sh -c 'echo "options snd-sof-pci fw_path=\"intel/sof\"" >>  /etc/modprobe.d/inteldsp.conf' 
    ```

4. 重启系统

重启后就应该有声音了，播放个[视频](https://www.bilibili.com/video/BV1uT4y1P7CX)试试！

## 备注

- 重启后初始音量可能特别高，注意！

- 耳机接口仍不能输出声音。插入耳机后还通过扬声器播放声音。

- 本教程基于 https://github.com/EMLommers/Apollolake_Audio 如有更新请关注这个仓库。
