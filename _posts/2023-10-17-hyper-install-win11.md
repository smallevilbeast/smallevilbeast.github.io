---
layout: post
title: Hyper-V 安装 Windows 11
categories: [Windows]
---

### vmware 的问题

vmware 在 17.0.0 之后，兼容了 `Hyper-V`, 但性能很差，因为要用 `wsl2` 的话，必须开启 `Hyper-V`，所以只能放弃 `vmware`，使用 `Hyper-V`。


### 开启 Hyper-V

```powershell
# 1. 开启 Hyper-V
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All

# 2. 开启 Hyper-V 管理器
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-Management-PowerShell
```

或在 `开启或关闭 Windows 功能` 中勾选 Hyper-V，然后重启。

确认 Hyper-V 已开启，用管理员权限打开 powershell：

```powershell
Get-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
```

### 安装 Windows 11

在 Hyper-V 管理器中新建，设置完后，需要注意的是

1. 在 `安全` 中，需要勾选 `启用安全启动`，否则会提示 `无法启动：未启用安全启动`。
2. 在 `安全` 中，勾选 `启用受信任的平台模块`，否则会提示 `无法启动：未启用受信任的平台模块`。

如果在启动时，提示没有可用的引导项目，在启动虚拟机时，按 `F2`  

### 跳过登录微软账户

在进入设置界面时，按 `shift + f10` 打开 `cmd`, 输入

```powershell
oobe\bypassnro
```

然后就可以跳过登录微软账户了，用本地账户，密码可以为空


### 激活 Windows 11

在 `cmd` 中输入

```powershell
slmgr /ipk W269N-WFGWX-YVC9B-4J6C9-T83GX
slmgr /skms kms.03k.org
slmgr /ato
```

