---
author: Icear
title: Zinit 介绍
index:  20220226170258
updated: 2022-02-27 16:02
date: 2022-02-26 17:04
aliases: Untitled 
description: 
tags: [TemplaterCreated, Document, ZKCard, Computer ]
finished: false
---

# Zinit 介绍

## 定义

一个为ZSH设计的插件管理框架，主要特性是轻量，支持延迟加载插件，避免ZSH启动时响应缓慢

## 安装

### 手动安装

- 下载源代码仓库

```shell
ZINIT_HOME="${XDG_DATA_HOME:-${HOME}/.local/share}/zinit/zinit.git"
mkdir -p "$(dirname $ZINIT_HOME)"
git clone https://github.com/zdharma-continuum/zinit.git "$ZINIT_HOME"
```

- 注册Zinit到`.zshrc`，添加这两行，完成基础配置

```shell
ZINIT_HOME="${XDG_DATA_HOME:-${HOME}/.local/share}/zinit/zinit.git"
source "${ZINIT_HOME}/zinit.zsh"
```

- （可选）（执行 `.zshrc`，启动Zinit

```shell
source ~/.zshrc
```

- （可选）（运行Zinit自动更新，让Zinit自行完成剩余配置

```shell
zinit self-update
```

## 添加插件

插件安装来源为github任意仓库，指定`用户名/仓库名`即可，将命令放入zshrc即可配置

```shell
zinit load  <repo/plugin> # Load with reporting/investigating.
zinit light <repo/plugin> # Load without reporting/investigating.
zinit snippet <URL> # Install direct script file, support local file
```

### OhMyZSH插件

对于 [[OhMyZSH]] 和 Prezto，还可以使用缩写 `OMZ::` 和 `PZT::`：

```bash
zinit snippet OMZ::plugins/git/git.plugin.zsh
zinit snippet PZT::modules/helper/init.zsh
```

## 常用配置插件

```shell
# Theme
zinit ice depth"1" # git clone depth
zinit light romkatv/powerlevel10k 

# 快速目录跳转 
zinit ice lucid wait='1' 
zinit light skywind3000/z.lua

# 语法高亮 
zinit ice lucid wait='0' atinit='zpcompinit' 
zinit light zdharma/fast-syntax-highlighting 
# 自动建议 
zinit ice lucid wait="0" atload='_zsh_autosuggest_start' 
zinit light zsh-users/zsh-autosuggestions

zinit load zdharma-continuum/history-search-multi-word

# 加载 OMZ 框架及部分插件 
zinit snippet OMZ::lib/completion.zsh 
zinit snippet OMZ::lib/history.zsh 
zinit snippet OMZ::lib/key-bindings.zsh 
zinit snippet OMZ::lib/theme-and-appearance.zsh 
zinit snippet OMZ::plugins/colored-man-pages/colored-man-pages.plugin.zsh 
zinit snippet OMZ::plugins/sudo/sudo.plugin.zsh

zinit ice lucid wait='1' 
zinit snippet OMZ::plugins/git/git.plugin.zsh


```

## 参考资料

- [GitHub - zdharma-continuum/zinit: 🌻 Flexible and fast ZSH plugin manager](https://github.com/zdharma-continuum/zinit)
- [加速你的 zsh —— 最强 zsh 插件管理器 zplugin/zinit 教程 - Aloxaf's Blog](https://www.aloxaf.com/2019/11/zplugin_tutorial/)