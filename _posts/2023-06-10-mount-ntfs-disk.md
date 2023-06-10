---
title: NTFS 文件系统挂载时的权限问题
published: true
---

用新的外星人笔记本安装了双系统， Linux 与 Windows 共用了一个数据分区 （ntfs 格式）， 凭着过往的经验， 在/etc/fstab 添加挂载

```bash
/dev/disk/by-uuid/xxxxxxx /mnt/data ntfs defaults 0 0
```
这样方式挂载后， `git clone`下来的项目， 会提示`检测到可疑的所有权`， 需要添加信任 
```
git config --global --add safe.directory /mnt/data/xxxx
```

从`ChatGPT` 得知可以添加权限掩码， 修改如下

```bash
/dev/disk/by-uuid/xxxxxxx /mnt/data ntfs defaults,nofail,utf8,uid=1000,gid=1000,dmask=022,fmask=133 0 0
```

- `uid=1000,gid=1000` 设置文件所有者的用户 ID 和组 ID， 这里设置为 1000， 通常在 Linux 中， 第一个创建的用户的 ID 就是 1000。
- `dmask=022` 设置目录的权限， 这里设置为 755 (即 `rwxr-xr-x`)。
- `fmask=133` 设置文件的权限， 这里设置为 644 (即 `rw-r--r--`)。

通过上面配置解决了`git`可疑所有权的问题， 但在 `nodejs` 项目 `npm install`时会提示写入和执行权限的问题， 我怀疑是 `npm install`会使用其他 `uid`来处理， 尝试改变了下权限， 如下：

```bash
/dev/disk/by-uuid/xxxxxxxx /mnt/data ntfs defaults,nofail,utf8,uid=1000,gid=1000,dmask=002,fmask=113 0 0
```
目前所遇到的问题都解决了 :)


