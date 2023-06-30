---
layout: post
title: lazycat-emacs 入门
published: true
categories: [Emacs]
---

### 楔子

初入 deepin 时， 看到[勇哥](https://manateelazycat.github.io/) 使用 emacs 写代码速度， 让当时用 eclipse 开发的我大吃一惊， emacs 竟然还可以这么玩， 在没见到勇哥之前感觉 emacs 就是个普通的编辑器， 当时就决定我要用上勇哥手上的 emacs， 再接下来的一周时间我就从编辑代码常用的功能开始， 差不到把 `init-key.el` 中的快捷键盘都按了一遍， 在使用上跟勇哥差不多达到了一致， 当时发了一个帖子 [emacs 要这样用](https://tieba.baidu.com/p/1402662061)， 但内功修炼上为 0， 毕竟还不会写一行 elisp， 仅仅当作工具用了。 在创业期间也用 emacs 完成了平台项目的开发， 在 2018 年买了小米游戏本用不了左边的`徽标键`后就没有在用 emacs 了， 再加上这几年代码量也少了， 主要以二进制分析为主， 再用 visual studio 写一些分析项目。 这几年看着勇哥又开发了 [emacs-application-framework](https://github.com/emacs-eaf/emacs-application-framework) 和 [lsp-bridge](https://github.com/manateelazycat/lsp-bridge.git) 让 emacs 在编辑功能跟 vscode 一个级别并且达到真正`live in emacs`的境界， 就想换个电脑重回 emacs, 在勇哥的推荐下换上了 `alienware M16`， 安装上`ubuntu`后， 勇哥把 [lazycat-emacs](https://github.com/manateelazycat/lazycat-emacs) 帮我配置好后， 回到 emacs 感觉实在泰酷辣。 具体如何生活在 emacs 中， 请看勇哥的 [如何跟 emacs 达到身心合一](https://manateelazycat.github.io/emacs/2022/11/07/how-i-use-emacs.html)。

### 安装`lazycat-emacs`

安装必要的编译工具和库
```bash
sudo apt install build-essential libncurses-dev libgnutls28-dev libjansson4 git
```
下载&编译 emacs
```
git clone --depth 1 git://git.savannah.gnu.org/emacs.git 
cd emacs
./autogen.sh
./configure
./make
sudo make install
```
下载 lazycat-emacs

```
git clone --recursive https://github.com/manateelazycat/lazycat-emacs.git
```
安装 eaf 项目插件
```
cd lazycat-emacs/site-lisp/extensions/emacs-application-framework
./install-eaf.py
```

配置.emacs
```elisp
(require 'cl-lib)

(tool-bar-mode -1)                      ;禁用工具栏
(menu-bar-mode -1)                      ;禁用菜单栏
(scroll-bar-mode -1)                    ;禁用滚动条

(defun add-subdirs-to-load-path (search-dir)
  (interactive)
  (let* ((dir (file-name-as-directory search-dir)))
    (dolist (subdir
             ;; 过滤出不必要的目录， 提升 Emacs 启动速度
             (cl-remove-if
              #'(lambda (subdir)
                  (or
                   ;; 不是目录的文件都移除
                   (not (file-directory-p (concat dir subdir)))
                   ;; 父目录、 语言相关和版本控制目录都移除
                   (member subdir '("." ".."
                                    "dist" "node_modules" "__pycache__"
                                    "RCS" "CVS" "rcs" "cvs" ".git" ".github"))))
              (directory-files dir)))
      (let ((subdir-path (concat dir (file-name-as-directory subdir))))
        ;; 目录下有 .el .so .dll 文件的路径才添加到 `load-path' 中， 提升 Emacs 启动速度
        (when (cl-some #'(lambda (subdir-file)
                           (and (file-regular-p (concat subdir-path subdir-file))
                                ;; .so .dll 文件指非 Elisp 语言编写的 Emacs 动态库
                                (member (file-name-extension subdir-file) '("el" "so" "dll"))))
                       (directory-files subdir-path))

          ;; 注意： `add-to-list' 函数的第三个参数必须为 t ， 表示加到列表末尾
          ;; 这样 Emacs 会从父目录到子目录的顺序搜索 Elisp 插件， 顺序反过来会导致 Emacs 无法正常启动
          (add-to-list 'load-path subdir-path t))

        ;; 继续递归搜索子目录
        (add-subdirs-to-load-path subdir-path)))))

;;这里改成 layzcat-eamcs 所在的目录
(add-subdirs-to-load-path "/home/evilbeast/lazycat-emacs")

(require 'init)

```

### 开发 EAF 扩展配置

在`.emacs`中添加

```lisp
;; Support jump to define of EAF root from EAF application directory.
(setq lsp-bridge-get-project-path-by-filepath
      (lambda (filepath)
        (when (string-prefix-p (expand-file-name "~/lazycat-emacs/site-lisp/extensions/emacs-application-framework/app") filepath)
          (expand-file-name "~/lazycat-emacs/site-lisp/extensions/emacs-application-framework/"))))

```


### eaf 中的 python 使用 pyenv

```bash
# 在原来的 python 环境下导出依赖包
pip freeze > requirements.txt 

python -m venv emacs_venv
source emacs_venv/bin/activate 
pip install -r requirements.txt
```

添加启动 emacs 的脚本 start_emacs.sh

```bash
#!/bin/bash
exec zsh -c 'source /home/evilbeast/emacs_venv/bin/activate; exec emacs "$@"' -- "$@"
```

### eaf gdb 调试运行

在.emacs 中添加 

```lisp
;; debug
(setq eaf-enable-debug t)
```

### eaf 登录 google

在 .emacs 中添加
```lisp
(setq eaf-webengine-pc-user-agent "Mozilla/5.0 ({os_info}; rv:90.0) Gecko/20100101 Firefox/90.0")
```


### 启动 emacs 并安装所需语言的 treesit

按下 `alt + x` 输入 `treesit-install-language-grammar` 安装所用语言的 treesit, 如 python, rust, vue


### 如何入门用 lazycat-emacs 编辑代码

如果从来没有用过 emacs, 学习方式就是记住快捷键， 使用的频率多了就会形成肌肉记忆， 就像五笔打字一样， 只需要将常用的功能如`复制`、 `粘贴`、 `换行`、 `到行头`和`到行尾`等记住就可以快速上手。 


首先使用 `ctrl + x + f` 打开一个文本文件或代码文件

1. 入门移动按键

|  按键  |   功能说明 |
|--------|-------------|
| ctrl + n| 下一行|
| ctrl + p| 上一行|
| ctrl + a| 到行头|
| ctrl + e| 到行尾|
| ctrl + f| 向前一个字符|
| ctrl + b| 向后一个字符|
| alt + f | 向前一个分词| 
| alt + b | 向后一个分词|

2. 选中按键

|  按键  |   功能说明 |
|--------|-------------|
| ctrl + shift + n| 选中下一行|
| ctrl + shift + p| 选中上一行|
| ctrl + shift + a| 选中到行头|
| ctrl + shift + e| 选中到行尾|
| ctrl + shift + f| 选中前一个字符|
| ctrl + shift + b| 选中后一个字符|
| ctrl + shift + b| 选中后一个字符|
| ctrl + x  h| 全选|

3. 复制剪切粘贴删除

`剪切可以当成删除来用`

|  按键  |   功能说明 |
|--------|-------------|
| alt + w| 复制 |
| ctrl + w| 剪切|
| ctrl + y| 粘贴|
| ctrl + k| 剪切到行尾|
| alt + shift + n | 剪切前一个分词|
| alt + shift + m | 剪切后一个分词|



### 导航诊断

Python 中安装 ruff 
```python
pip install ruff-lsp pyright
```
[详情](https://github.com/manateelazycat/lazycat-emacs/blob/master/site-lisp/config/init-key.el#L536)


|  按键  |   功能说明 |
|--------|-------------|
| alt + super + j | 下一个诊断 |
| alt + super + k | 上一个诊断 |


### Git 管理

|  按键  |   功能说明 |
|--------|-------------|
|super + x f |  git keyMenu |


### Python 代码块

|  按键  |   功能说明 |
|--------|-------------|
| ctrl + shift + y |  粘贴后简单的向右 |
| ctrl + shift + j |  选中向左移动 |
| ctrl + shift + k |  选中向右移动 |

[矩形操作](https://github.com/manateelazycat/lazycat-emacs/blob/master/site-lisp/config/init-key.el#L194)

|  按键  |   功能说明 |
|--------|-------------|
| super + shift + m |  开始矩形操作 |
| super + shift + f |  选中空格 |
| super + shift + i |  插入 |
| super + shift + d |  删除 |

### 音乐播放器

|  按键  |   功能说明 |
|--------|-------------|
| super + 1 |  全局随机播放 |
| super + 2 |  全局暂停/继续播放 |
| super + 3 |  唤出播放器 |


### 代码阅读模式

[详情](https://github.com/manateelazycat/lazycat-emacs/blob/master/site-lisp/config/init-easy-nav.el#L94C1-L94C1)

|  按键  |   功能说明 |
|--------|-------------|
| ctrl + super + l |  进入代码阅读模式 |
| q |  退出 |

### markdown 预览

|  按键  |   功能说明 |
|--------|-------------|
| ctrl + c  ctrl + c |  在 md 文件中直接打开预览 |



