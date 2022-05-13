---
author: Icear
title: Mellanox ASAP2
index:  20220322170328
updated: 2022-05-13 18:05
date: 2022-03-22 17:14
aliases: Nvidia ASAP2 
description: 
finished: false
tags:  [TemplaterCreated, Document, ZKCard, Network, Mellanox, Nvidia, HardwareAccleration]
---

# Mellanox ASAP2


## 定义

Accelerated Switching and Packet Processing

加速交换与数据报处理技术，原由Mellanox提出但在Nvidia收购后更名为 Nvidia ASAP2

## 应对场景

主要针对高速网络下的加速网络数据包场景，传统网络数据包处理需要将网络通信数据包全部提交至CPU进行处理，这些处理过程占用了CPU资源并且开销日益增大

## 应对思路

让网卡接管网络IO的处理，并且在硬件上完成原本CPU需要进行的网络数据解析任务，并且携带网络数据分流的特性，支持Single Root Virtualization等方式模拟出多个网络设备，将不同的网络流量分别呈递，能够接入[[OpenVirtualSwitch]] 虚拟交换或其它虚拟网络，使得这一过程对CPU的开销降低到非常低的程度，从而达到硬件加速的效果

## 性能

```
Benchmarks show that on a server with a 25G interface, OVS accelerated by ASAP2 Direct achieves 33 million packets per second (mpps) for a single flow with near-zero CPU cores consumed by the OVS data plane, and about 18 mpps with 60,000 flows performing VXLAN encap/decap. These performance results are three to ten times higher than DPDK-accelerated OVS. ASAP2 offers the best of both worlds, software-defined flexible network programmability, and high network I/O performance for the state-of-art speeds from 10G to 25/40/50/100/200G.
```

25G 端口下，通过`ASAP2 Direct` 技术能够处理单条流33mpps数据包，且无CPU占用，在`VXLAN`解包生成包操作下能够达到最多6万条流一共18mpps的性能

目前此技术可应用在25/40/50/100/200G网络通信模式中

## 参考资料

- [https://network.nvidia.com/pdf/products/SB_asap2.pdf](https://network.nvidia.com/pdf/products/SB_asap2.pdf)
- [OVS Offload Using ASAP2 Direct - MLNX_EN v4.9-3.1.5.0 LTS - NVIDIA Networking Docs](https://docs.nvidia.com/networking/display/MLNXENv493150/OVS+Offload+Using+ASAP2+Direct)
