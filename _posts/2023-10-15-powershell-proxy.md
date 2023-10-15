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