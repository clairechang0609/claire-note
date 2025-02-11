---
title: 透過 NVM 管理多版本 Node.js（MacOS）
date: 2023-02-18
tags: [ node ]
category: Node
description: 在開發時，我們常會碰到專案 Node.js 版本不同的問題，導致編譯或開發發生錯誤，因此常常需要來回進行版本切換，透過 NVM，我們可以輕鬆地管理 Node.js 版本
image: https://i.imgur.com/t4LX3EU.png
---

<div style="display: flex; justify-content: center; margin: 20px 0;">
    <img style="width: 100%; max-width: 450px;" src="https://i.imgur.com/t4LX3EU.png">
</div>

**NVM（Node Version Manager）為 Node.js 版本管理工具。**

每個專案開發可能會搭配不同的 Node.js 版本，版本差異會導致編譯或開發發生問題，因此常常需要來回進行版本切換，而 NVM 最大的用途是可以輕鬆切換 Node.js 版本。

<!-- more -->

## **安裝 NVM**

打開「Terminal 終端機」輸入指令安裝 NVM（v0.39.3 為版本號）

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
```

如果我們是使用其他的 shell 指令工具，輸入 `nvm` 可能會報錯，像是：

{% colorquote danger %}
zsh: command not found: nvm
{% endcolorquote %}

{% colorquote info %}
**shell：**命令解析器，讓使用者透過終端機跟作業系統核心做溝通，而 **bash** 跟 **zsh** 是 shell 指令工具
{% endcolorquote %}

發生錯誤需要手動設定，**bash** 跟 **zsh** 兩種指令工具則一：

### **bash**

在根目錄檢查是否有 `.bash_profile` 檔案（為隱藏檔案，輸入 `shift + command + .` 可以顯示隱藏檔），如果沒有使用指令手動建立

```bash
touch ~/.bash_profile
```

接下來打開檔案加入程式碼

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

接著輸入 `nvm` 取得以下內容代表安裝成功

```bash
Node Version Manager (v0.39.3)

Note: <version> refers to any version-like string nvm understands. This includes:
    - full or partial version numbers, starting with an optional "v" (0.10, v0.1.2, v1)
    - default (built-in) aliases: node, stable, unstable, iojs, system
    - custom aliases you define with `nvm alias foo`

    Any options that produce colorized output should respect the `--no-colors` option.

Usage:
    nvm --help                                  Show this message
        --no-colors                               Suppress colored output
    nvm --version                               Print out the installed version of nvm
    nvm install [<version>]                     Download and install a <version>. Uses .nvmrc if available and version is omitted.
    The following optional arguments, if provided, must appear directly after `nvm install`:
...
```

### **zsh**

在根目錄檢查是否有 `.zshrc` 檔案（為隱藏檔案），如果沒有使用指令手動建立

```bash
touch ~/.zshrc
```

接下來打開檔案加入程式碼

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

## **NVM 相關指令**

- nvm ls-remote：當前可用的遠端 Node.js 版本
- nvm list：本機安裝的 Node.js 版本
- nvm install {{ version }} ：安裝特定版本 Node.js
- nvm alias default {{ version }}：命令列預設使用 Node.js 版本
- nvm use {{ version }}：當前命令列使用 Node.js 版本

## **透過 NVM 安裝多版本 Node.js**

執行 `nvm ls-remote` 可以看到如下（截取片段）

```bash
...
v14.11.0
v14.12.0
v14.13.0
v14.13.1
v14.14.0
v14.15.0   (LTS: Fermium)
v14.15.1   (LTS: Fermium)
v14.15.2   (LTS: Fermium)
v14.15.3   (LTS: Fermium)
...
v14.20.1   (LTS: Fermium)
v14.21.0   (LTS: Fermium)
v14.21.1   (LTS: Fermium)
v14.21.2   (LTS: Fermium)
v14.21.3   (Latest LTS: Fermium)
v15.0.0
v15.0.1
v15.1.0
...
```

{% colorquote info %}
挑選版本時建議安裝 **LTS（長期支援）版本**
{% endcolorquote %}

假設想同時安裝 v14 跟 v16，供不同專案使用，該如何做：

**1. 安裝 Node.js**

```bash
nvm install 14
nvm install 16
```

**2. 指定預設版本**

```bash
nvm alias default 14
```

透過 `nvm list` 查看本機安裝版本

<div style="display: flex; justify-content: left; margin: 20px 0;">
    <img style="width: 100%; max-width: 400px;" src="https://i.imgur.com/AOpVXxL.png">
</div>

**3. 專案切換版本**

特定專案版本切換

```bash
nvm use 16
```

---

參考文章：

[https://ithelp.ithome.com.tw/articles/10261849](https://ithelp.ithome.com.tw/articles/10261849)

[https://www.casper.tw/development/2022/01/10/install-nvm/](https://www.casper.tw/development/2022/01/10/install-nvm/)