---
layout: post
title: PowerShell 代理
categories: [Windows]
---

## 设置代理

```powershell  
$proxy = "http://127.0.0.1:10807"
[system.net.webrequest]::defaultwebproxy = new-object system.net.webproxy($proxy)
[system.net.webrequest]::defaultwebproxy.credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials

# and 
$env:HTTPS_PROXY="http://127.0.0.1:10807"
$env:HTTP_PROXY="http://127.0.0.1:10807"

```

## 取消代理

```powershell
[system.net.webrequest]::defaultwebproxy = new-object system.net.webproxy
[system.net.webrequest]::defaultwebproxy.credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials

# or 
[system.net.webrequest]::defaultwebproxy = $null
[system.net.webrequest]::defaultwebproxy.credentials = $null

# and 
$env:HTTPS_PROXY=""
$env:HTTP_PROXY=""
```

### 函数定义
以下是可以添加到你的 PowerShell 配置文件的函数定义：

```PowerShell
function Enable-Proxy {
    $proxy = "http://127.0.0.1:10807"
    [system.net.webrequest]::defaultwebproxy = new-object system.net.webproxy($proxy)
    [system.net.webrequest]::defaultwebproxy.credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials

    $env:HTTPS_PROXY=$proxy
    $env:HTTP_PROXY=$proxy

    Write-Host "Proxy enabled: $proxy" -ForegroundColor Green
}

function Disable-Proxy {
    [system.net.webrequest]::defaultwebproxy = new-object system.net.webproxy
    [system.net.webrequest]::defaultwebproxy.credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials

    $env:HTTPS_PROXY=""
    $env:HTTP_PROXY=""

    Write-Host "Proxy disabled" -ForegroundColor Yellow
}
```

### 添加函数到 PowerShell 配置文件
1. 打开你的 PowerShell 配置文件 `$PROFILE` 以进行编辑。你可以使用任何文本编辑器来做这个。以下是如何使用 Notepad 来做的命令：
    ```PowerShell
    notepad.exe $PROFILE
    ```
    如果 `$PROFILE` 文件不存在，你需要创建它：
    ```PowerShell
    if (!(Test-Path $PROFILE)) {
        New-Item -Type File -Path $PROFILE -Force
    }
    notepad.exe $PROFILE
    ```
   
2. 将上述的函数定义粘贴到配置文件中，并保存文件。

3. 关闭并重新打开 PowerShell。

### 使用函数
- 要开启代理，只需在 PowerShell 中键入 `Enable-Proxy` 并按 Enter。
  
- 要关闭代理，键入 `Disable-Proxy` 并按 Enter。
