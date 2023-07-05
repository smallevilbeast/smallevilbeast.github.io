---
layout: post
title: iPhone 15.0.2 越狱
categories: [jailbreak]
---

### palera1n

目前 palera1n 适用于 iOS 15.0-16.5 上的 checkm8 设备 (A8-A11), 我手上这台是`iPhone 8 Plus`, CPU 是`A11`, OS 是 `15.0.2` 刚好满足， 不过由于 iOS 15 之后引入的 Signed System Volume (SSV) 安全措施， 无论是基于 checkm8 的 palera1n 还是基于 Fugu 系列的 Dopamine 都不再挂载 rootfs ， 叫 rootless 模式

在 [palera1n releases](https://github.com/palera1n/palera1n/releases) 页面下载 `palera1n-macos-x86_64`, 后在终端执行

```bash
chmod +x ./palera1n-macos-x86_64
./palera1n-macos-x86_64
```

过程如下， 注意要先按 `音量减键` + `电源键`， 然后看提示只放开`电源键`


```bash
➜  Downloads ./palera1n-macos-x86_64
# == palera1n-c ==
#
# Made by: Nick Chan, Ploosh, Samara, Nebula, staturnz, kok3shidoll
#
# Thanks to: pythonplayer123, llsc12, Mineek, tihmstar, nikias
# (libimobiledevice), checkra1n team (Siguza, axi0mx, littlelailo
# et al.), Procursus Team (Hayden Seay, Cameron Katri, Keto et.al)

 - [07/05/23 12:51:14] <Info>: Waiting for devices
 - [07/05/23 12:51:14] <Info>: Telling device with udid e7daef1c6385943f3b17664008b45134a636aa0e to enter recovery mode immediately
 - [07/05/23 12:51:26] <Info>: Press Enter when ready for DFU mode

Get ready (0)
Hold volume down + side button (0) 
Hold volume down button (3)
 - [07/05/23 12:51:48] <Info>: Device entered DFU mode successfully
 - [07/05/23 12:51:48] <Info>: About to execute checkra1n
#
# Checkra1n 0.1337.1
#
# Proudly written in nano
# (c) 2019-2023 Kim Jong Cracks
#
#========  Made by  =======
# argp, axi0mx, danyl931, jaywalker, kirb, littlelailo, nitoTV
# never_released, nullpixel, pimskeks, qwertyoruiop, sbingner, siguza
#======== Thanks to =======
# haifisch, jndok, jonseals, xerub, lilstevie, psychotea, sferrini
# Cellebrite (ih8sn0w, cjori, ronyrus et al.)
#==========================

 - [07/05/23 12:51:48] <Verbose>: Starting thread for Apple TV 4K Advanced board
 - [07/05/23 12:51:48] <Info>: Waiting for DFU mode devices
 - [07/05/23 12:51:48] <Verbose>: DFU mode device found
 - [07/05/23 12:51:48] <Info>: Checking if device is ready
 - [07/05/23 12:51:48] <Verbose>: Attempting to perform checkm8 on 8015 11
 - [07/05/23 12:51:48] <Info>: Setting up the exploit
 - [07/05/23 12:51:48] <Verbose>: == checkm8 setup stage ==
 - [07/05/23 12:51:48] <Verbose>: Entered initial checkm8 state after 1 steps
 - [07/05/23 12:51:48] <Verbose>: Stalled input endpoint after 8 steps
 - [07/05/23 12:51:48] <Verbose>: DFU mode device disconnected
 - [07/05/23 12:51:49] <Verbose>: DFU mode device found
 - [07/05/23 12:51:49] <Verbose>: == checkm8 trigger stage ==
 - [07/05/23 12:51:53] <Info>: Checkmate!
 - [07/05/23 12:51:53] <Verbose>: Device should now reconnect in download mode
 - [07/05/23 12:51:53] <Verbose>: DFU mode device disconnected
 - [07/05/23 12:52:01] <Info>: Entered download mode
 - [07/05/23 12:52:01] <Verbose>: Download mode device found
 - [07/05/23 12:52:01] <Info>: Booting PongoOS...
 - [07/05/23 12:52:03] <Info>: Found PongoOS USB Device
 - [07/05/23 12:52:03] <Info>: Booting Kernel...
 ```

### 安装 Sileo

进入系统后会看到 `palera1n` 的应用， 首次会提示设置`mobile`用户的密码， 后面可以连 ssh 用， 打开后安装 `Sileo` 软件商店工具， 添加 `https://cydia.akemi.ai/` 源安装

1. appinst
2. AppSync Unified
3. xz-utils

### 通过 usb 连接 ssh

改端口
```bash
alias usbproxy='nohup iproxy 2222 22 > /dev/null 2>&1 &'
```

先使用`mobile`账号连接手机 ssh

```bash
ssh -p 2222 mobile@localhost
```
改 root 密码 (旧密码: alpine)

```bash
sudo passwd root
```

设置 zshrc

```bash
alias usbproxy='nohup iproxy 2222 22 > /dev/null 2>&1 &'
alias usbssh='ssh -p 2222 root@localhost'
```

### 安装 frida

在 sileo 中添加源

`https://miticollo.github.io/repos/my` 

安装即可， 如果失效可以用下面的方式


### 手动安装并运行 frida

目前 frida 还没有适配 rootless 模式， 不能通过软件源安装， 只能手动安装， 从 [frida releases](https://github.com/frida/frida/releases) 下载 `frida_xx.x.x_iphoneos-arm.deb` 使用 `ar -x frida_xx.x.x_iphoneos-arm.deb` 解压出 `data.tar.xz`, 将`data.tar.xz`复制到手机的`/tmp`目录

```bash
scp -P 2222 data.tar.xz root@127.0.0.1:/tmp/
```

解压`data.tar.xz` 

```bash
mkdir frida
tar -xf data.tar.xz -C frida/
CRYPTEX_MOUNT_PATH=/tmp/frida /tmp/frida/usr/sbin/frida-server
```
最后在本机使用 `frida-ps -U` 验证

测试没问题将 `frida` 目录移动到 `/var/jb/var/root/` 目录下

start_frida.sh

```bash
#!/bin/sh
CRYPTEX_MOUNT_PATH=/var/jb/var/root/frida /var/jb/var/root/frida/usr/sbin/frida-server > /dev/null 2>&1 &
```
