---
author: Icear
title: WSL2占用53端口的解决方案
index:  20220309220306
updated: 2022-03-09 22:03
date: 2022-03-09 22:22
aliases: 20220309220305 
description: 
finished: false
tags:  [TemplaterCreated, Document, ZKCard, WSL]
---

# WSL2占用53端口的解决方案



## 定义

[[WSL]]2在运行时由于为虚拟机架构，因此与宿主机的网络通信实际上为两块虚拟网卡之间相互通信，为了让WSL2子系统能够正常访问网络，其在初始化时会在宿主机53端口上开启一个DNS解析服务，并且由于虚拟网卡的IP端不固定，会随着每次系统的重启而变化，因此该解析服务监听了`0.0.0.0:53`，由此导致了在WSL2运行时，用户无法自行运行53端口的DNS解析服务。

## 解决方案

实际上该DNS解析服务范围为`0.0.0.0:53`，即IPv4空间，而IPv6空间下53端口为空闲状态，因此用户自行启用的53服务设定去监听IPv6即可，并且Windows主机支持只设定ipv6的DNS，且v6下的DNS也可以解析v4的IP

## 参考资料

- [WslRegisterDistribution failed with error: 0xffffffff · Issue #4364 · microsoft/WSL · GitHub](https://github.com/microsoft/WSL/issues/4364)
