---
layout: post
title: Hyper-V 内部网络共享访问外网
categories: [Windows]
---

### 在 Hyper-V 中新建内部网络

在虚拟交换机管理器中，新建内部网络，类型选择 `内部`

### 给虚拟机分配内部网络

在虚拟机设置中，网络适配器，选择建立的内部网络

### 修改共享网络的 IP 段

在注册表中 `计算机 \HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters`

修改 `ScopeAddress` 和 `ScopeAddressBackup` 的值为内部网络的 IP 改为 `172.18.10.1`

### 将外网卡共享给内部网络

在网络连接中，右键外网卡，属性，共享，勾选 `允许其他网络用户通过此计算机的 Internet 连接来连接` 

### 设置重启后自动启动共享网络

打开注册表，然后在 `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\SharedAccess` 中
在空白处右击鼠标，新建 `DWORD（32 位）值（D）`，名称叫做 `EnableRebootPersistConnection`，将数值数据改为 1。
