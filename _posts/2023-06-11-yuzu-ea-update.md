---
title: yuzu 模拟器自动更新
published: true
---

使用 shell 实现自动检测新版本并更新, 同时支持 zsh 和 bash

### update_yuzu.sh 

```bash
#!/bin/bash

GITURL='https://api.github.com/repos/pineappleEA/pineapple-src/releases/latest'
content=$(wget -qO- "$GITURL")

GITVER=$(echo "$content" | grep -o 'Linux-Yuzu-EA-[[:digit:]]*.AppImage' | uniq | grep -o '[[:digit:]]*' )
DOWNLOAD_URL=$(echo "$content" | grep "browser_download_url.*AppImage" | cut -d : -f 2,3 | tr -d \" )

files=(Linux-Yuzu-EA-*.AppImage)
if [ -f "${files[0]}" ]; then
    APPVER=$(echo "${files[-1]}" | grep -o '[[:digit:]]*')
else
    APPVER=0
fi

if [ "$GITVER" -gt "$APPVER" ]; then
        echo "Check for a new version: $GITVER, start download"
        wget --no-verbose --show-progress $DOWNLOAD_URL
        chmod +x ./Linux-Yuzu-EA-"$GITVER".AppImage
        files=(Linux-Yuzu-EA-*.AppImage)
        file_count=${#files[@]}
        if [ $file_count -gt 2 ]; then
                delete=("${files[@]:0:$(($file_count-2))}")
                rm "${delete[@]}"
        fi
        echo "Successfully updated to the latest version: $GITVER"
else
        echo "Already the latest version"
fi
```

### run_yuzu.sh

```bash
RUN_DIR=$(dirname "$0")
APP=$(ls $RUN_DIR/Linux-Yuzu-EA-*.AppImage | tail -n 1 )
$APP
```

### ~/.local/share/applications/yuzu.desktop

```ini
[Desktop Entry]
Name=yuzu
Exec=/home/evilbeast/Game/run_yuzu.sh
Icon=/home/evilbeast/icons/yuzu.svg
Type=Application
Categories=Utility;
```
