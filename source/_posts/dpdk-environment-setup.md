---
author: Icear
title: DPDK 环境搭建记录
index:  20220309140381
updated: 2022-04-13 13:04
date: 2022-03-09 14:37
aliases: DPDK 
description: 
finished: false
tags:  [TemplaterCreated, Document, ZKCard, Network]

---

# DPDK环境搭建记录


## 定义

由Intel发起的开源网络数据处理框架及相关生态环境，本身面向更通用化的数据平面编程，并且在和Intel硬件的结合后能够有更好的运行性能（如DPDK在Intel CPU及Intel NIC环境中，数据包在到达NIC后能够直接上传至CPU的三级缓存中，相比其他PF_RING等框架为NIC通过DMA上传至系统内存，这种优化能够减少一次CPU读取内存的操作）

```ad-comment
对大多数平台，使用基本DPDK功能无需对BIOS进行特殊设置。然而，对于HPET定时器和电源管理功能，以及为了获得40G网卡上小包处理的高性能，则可能需要更改BIOS设置。可以参阅章节  [Enabling Additional Functionality](https://doc.dpdk.org/guides/linux_gsg/enable_func.html#enabling-additional-functionality)以获取更为详细的信息。
```

## 环境配置

### 配置依赖

- General development tools including a supported C compiler such as gcc (version 4.9+) or clang (version 3.4+), and `pkg-config`  or `pkgconf` to be used when building end-user binaries against DPDK.

	- For RHEL/Fedora systems these can be installed using
	```shell
	dnf groupinstall "Development  Tools"
	```

	- For Ubuntu/Debian systems these can be installed using
	```shell
	apt install build-essential
	```

	- For Alpine Linux
	```shell
	apk add alpine-sdk bsd-compat-headers libexecinfo-dev
	```

- Python 3.5 or later.
- Meson (version 0.49.2+) and ninja
	- `meson`  &  `ninja-build`  packages in most Linux distributions
	- If the packaged version is below the minimum version, the latest versions can be installed from Python’s “pip” repository: `pip3 install meson ninja`
- `pyelftools`  (version 0.22+)
	- For Fedora systems it can be installed using  `dnf install python-pyelftools`
	- For RHEL/CentOS systems it can be installed using `pip3 install pyelftools`
	- For Ubuntu/Debian it can be installed using  `apt install python3-pyelftools`
	- For Alpine Linux,  `apk add py3-elftools`
- Library for handling NUMA (Non Uniform Memory Access).
	- `numactl-devel`  in RHEL/Fedora;
	- `libnuma-dev`  in Debian/Ubuntu;
	- `numactl-dev`  in Alpine Linux

#### Optional Tools

- Intel® C++ Compiler (icc). For installation, additional libraries may be required. See the icc Installation Guide found in the Documentation directory under the compiler installation.
- IBM® Advance ToolChain for Powerlinux. This is a set of open source development tools and runtime libraries which allows users to take leading edge advantage of IBM’s latest POWER hardware features on Linux. To install it, see the IBM official installation document.

#### Additional Libraries

 A number of DPDK components, such as libraries and poll-mode drivers (PMDs) have additional dependencies. For DPDK builds, the presence or absence of these dependencies will be automatically detected enabling or disabling the relevant components appropriately.

In each case, the relevant library development package (  -devel or  -dev  ) is needed to build the DPDK components.

For libraries the additional dependencies include:

- libarchive: for some unit tests using tar to get their resources.
- libelf: to compile and use the bpf library.

For poll-mode drivers, the additional dependencies for each driver can be found in that driver’s documentation in the relevant DPDK guide document, e.g. [Network Interface Controller Drivers — Data Plane Development Kit 22.03.0 documentation](https://doc.dpdk.org/guides/nics/index.html)

#### System Software

**Required:**

- Kernel version >= 4.4

	The kernel version required is based on the oldest long term stable kernel available at kernel.org when the DPDK version is in development. Compatibility for recent distribution kernels will be kept, notably RHEL/CentOS 7.

	The kernel version in use can be checked using the command:

	uname -r

- glibc >= 2.7 (for features related to cpuset)

	The version can be checked using the `ldd --version` command.

- Kernel configuration

	In the Fedora OS and other common distributions, such as Ubuntu, or Red Hat Enterprise Linux, the vendor supplied kernel configurations can be used to run most DPDK applications.

	For other kernel builds, options which should be enabled for DPDK include:

	- HUGETLBFS
	- PROC_PAGE_MONITOR support
	- HPET and HPET_MMAP configuration options should also be enabled if HPET support is required. See the section on [High Precision Event Timer (HPET) Functionality](https://doc.dpdk.org/guides/linux_gsg/enable_func.html#high-precision-event-timer) for more details.

### 编译

#### 下载源代码并解压

```shell
tar xJf dpdk-<version>.tar.xz
cd dpdk-<version>
```

The DPDK is composed of several directories, including:

- doc: DPDK Documentation
- license: DPDK license information
- lib: Source code of DPDK libraries
- drivers: Source code of DPDK poll-mode drivers
- app: Source code of DPDK applications (automatic tests)
- examples: Source code of DPDK application examples
- config, buildtools: Framework-related scripts and configuration
- usertools: Utility scripts for end-users of DPDK applications
- devtools: Scripts for use by DPDK developers
- kernel: Kernel modules needed for some operating systems

#### DPDK Configuration

To configure a DPDK build use:

```shell
meson <options> build
```

where “build” is the desired output build directory, and `<options>` can be empty or one of a number of meson or DPDK-specific build options, described later in this section. The configuration process will finish with a summary of what DPDK libraries and drivers are to be built and installed, and for each item disabled, a reason why that is the case. This information can be used, for example, to identify any missing required packages for a driver.

Once configured, to build and then install DPDK system-wide use:

```shell
cd build
ninja
ninja install
ldconfig
```

The last two commands above generally need to be run as root, with the ninja install step copying the built objects to their final system-wide locations, and the last step causing the dynamic loader ld.so to update its cache to take account of the new objects.

### 配置网卡驱动

DPDK使用PMD驱动操作网卡，并且有多个可选项，默认使用 `UIO` or `VFIO`驱动，但是也能够基于[Bifurcated Driver](https://doc.dpdk.org/guides/linux_gsg/linux_drivers.html#bifurcated-driver) 运行


```ad-comment
It is recommended that `vfio-pci` be used as the kernel module for DPDK-bound ports in all cases. If an IOMMU is unavailable, the `vfio-pci` can be used in [no-iommu](https://doc.dpdk.org/guides/linux_gsg/linux_drivers.html#vfio-noiommu) mode. If, for some reason, vfio is unavailable, then UIO-based modules, `igb_uio` and `uio_pci_generic` may be used. See section [UIO](https://doc.dpdk.org/guides/linux_gsg/linux_drivers.html#uio) for details.
```

大部分设备被DPDK使用时，需要将其脱离Linux Kernel的控制，这意味着Linux Kernel将无法看到该设备并管理，**其它非DPDK应用程序在Kernel重新获得控制权之前也无法使用该设备。对于网络设备可以基于端口进行分割绑定**

操作设备绑定与解绑，DPDK提供一个实用脚本 `dpdk-devbind.py` 可使用
- 查看系统中网络端口状态
```shell
./usertools/dpdk-devbind.py --status

Network devices using DPDK-compatible driver
============================================
0000:82:00.0 '82599EB 10-GbE NIC' drv=vfio-pci unused=ixgbe
0000:82:00.1 '82599EB 10-GbE NIC' drv=vfio-pci unused=ixgbe

Network devices using kernel driver
===================================
0000:04:00.0 'I350 1-GbE NIC' if=em0  drv=igb unused=vfio-pci *Active*
0000:04:00.1 'I350 1-GbE NIC' if=eth1 drv=igb unused=vfio-pci
0000:04:00.2 'I350 1-GbE NIC' if=eth2 drv=igb unused=vfio-pci
0000:04:00.3 'I350 1-GbE NIC' if=eth3 drv=igb unused=vfio-pci

Other network devices
=====================
<none>
```
- 绑定设备“04：00.1”到驱动vfio-pci
```shell
./usertools/dpdk-devbind.py --bind=vfio-pci 04:00.1
# or
./usertools/dpdk-devbind.py --bind=vfio-pci eth1
```

- 释放设备到系统，即切换其驱动到标准驱动，支持通配符
```shell
./usertools/dpdk-devbind.py --bind=ixgbe 82:00.*
```

#### VFIO 驱动相关

默认情况下，`VFIO 驱动` 通常已集成在系统中，能够直接启用运行

通过前述提到的 `dpdk-devbind.py` 能够将设备切换到 `VFIO驱动` 中

部分情况下使用脚本将设备绑定到 `VFIO驱动` 

## 开发DPDK应用

### 编译环境配置

When installed system-wide, DPDK provides a pkg-config file `libdpdk.pc` for applications to query as part of their build. It’s recommended that the pkg-config file be used, rather than hard-coding the parameters (cflags/ldflags) for DPDK into the application build process.

An example of how to query and use the pkg-config file can be found in the `Makefile` of each of the example applications included with DPDK. A simplified example snippet is shown below, where the target binary name has been stored in the variable `$(APP)` and the sources for that build are stored in `$(SRCS-y)`.

PKGCONF = pkg-config

CFLAGS += -O3 $(shell $(PKGCONF) --cflags libdpdk)

LDFLAGS += $(shell $(PKGCONF) --libs libdpdk)

$(APP): $(SRCS-y) Makefile
		$(CC) $(CFLAGS) $(SRCS-y) -o $@ $(LDFLAGS)

```ad-comment
Unlike with the make build system present in older DPDK releases, the meson system is not designed to be used directly from a build directory. Instead it is recommended that it be installed either system-wide or to a known location in the user’s home directory. The install location can be set using the –prefix meson option (default: /usr/local).
```

an equivalent build recipe for a simple DPDK application using meson as a build system is shown below:

project('dpdk-app', 'c')

dpdk = dependency('libdpdk')

sources = files('main.c')

executable('dpdk-app', sources, dependencies: dpdk)

## 参考资料

- [Linux平台上DPDK入门指南 - allcloud - 博客园](https://www.cnblogs.com/allcloud/p/7723305.html)
- [3. Compiling the DPDK Target from Source — Data Plane Development Kit 22.03.0 documentation](https://doc.dpdk.org/guides/linux_gsg/build_dpdk.html)
