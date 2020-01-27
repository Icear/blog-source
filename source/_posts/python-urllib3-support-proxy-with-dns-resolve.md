---
title: Urllib 通过 socks 协议解析 DNS
date: 2020-01-27 21:41:44
tags: [urllib, socks, dns]
---

socks 协议是支持进行 DNS 解析的，但是很多使用该协议的程序并没有实现从 socks 协议来解析域名，仅仅是将流量转发到了 socks 端口上。

今天在使用 python 的 urllib 的时候意外地发现 urllib 是支持设定从 socks 协议进行解析的，具体的 PR 在 https://github.com/urllib3/urllib3/pull/1036

在设定 proxy 时将协议头设置为 socks5h 即可