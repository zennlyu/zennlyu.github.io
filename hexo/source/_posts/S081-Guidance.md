---
title: S081-xv6 setup on Catalina
categories: [mit_opencourseware]
tags: [S081, Operating System]
---

# Install xv6 on Catalina

配置 `S081` 实验环境的时候，官网对于 macOS 的安装方法 `brew install riscv-tools` 等包管理无法在 Catalina 及以下的系统版本上安装成功，需要进行手动编译。编译项如下

- RISC-V工具链： 包括一系列交叉编译的工具，用于把源码编译成机器码，如 `gcc，binutils，glibc` 等
- QEMU模拟器： 用于在 X86 机器上模拟 RISC-V 架构的 CPU
- xv6源码： xv6 操作系统源码

## 一、RISC-V toolchain

下载后在源码根目录进行编译，具体参考官方文档。编译大约需要30min：

```shell
git clone --recursive https://github.com/riscv/riscv-gnu-toolchain
cd riscv-gnu-toolchain
./configure --prefix=/usr/local/opt/riscv-gnu-toolchain    #配置产物路径
make                                                       #编译
```

在macOS catalina版本下进行./configure时提示缺少GNU的 awk 和 sed: configure: error: GNU awk not found，手动安装即可: 

```shell
brew install gawk
brew install gsed
```

安装完成后需配置环境变量，与上一步设置的安装路径一致。Mac下的环境配置文件是~/.bash_profile（Linux下为~/.bashrc或~/.profile)，操作如下：

```shell
vim ~/.bash_profile                                              #打开配置文件
export PATH="$PATH:/usr/local/opt/riscv-gnu-toolchain/bin"       #末尾添加此行
source ~/.bash_profile                                           #使配置生效
```

此时在命令行输入 `riscv64-unknown-elf-gcc -v`，如果能显示版本信息则代表安装成功。

需要注意的是，在编译的过程中，会出现报错，需要手动在 `terminal.c` 和  `rltty.c` 中添加 `#include <sys/ioctl.h>` 头文件。

## 二、QEMU

编译生成的 risc-v 机器码需要通过模拟 cpu 执行，因此需要从qemu官网下载指定版本的源码并编译：

```shell
wget https://download.qemu.org/qemu-4.1.0.tar.xz  #下载后解压并进入目录
./configure                                       #默认安装所有目标平台，产物路径为/usr/local/bin
make && make install                              #编译并安装
```

## 三、xv6

进入 MIT 的官网 `https://pdos.csail.mit.edu/6.S081/2020/` , 键入以下命令

```shell
$ git clone git://g.csail.mit.edu/xv6-labs-2020
$ cd xv6-labs-2020
$ git checkout util
$ make
$ make qemu
```

即进入实验环境