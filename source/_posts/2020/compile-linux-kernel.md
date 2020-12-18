---
title: compile-linux-kernel
top: false
cover: false
toc: true
mathjax: true
tags:
  - Algorithm
categories:
  - - Algorithm
date: 2020-10-19 20:31:54
summary:
---

# 编译并导入Linux内核
reference:
- CSC3150 Tutorial 2&3
- https://www.cnblogs.com/acm-icpcer/p/8029656.html

+ 虚拟机硬盘空间要足够大，否则编译过程中空间会炸掉
+ 首次编译且安装的命令：
    ```sh
    sudo make mrproper 
    sudo make clean 
    sudo make menuconfig
    sudo make -j4 
    sudo make modules_install
    sudo make install
    reboot
    ```
+ 更改某些源文件后不需要重新编译：
    ```sh
    sudo make bzImage
    sudo make modules_install
    sudo make install
    reboot
    ```
+ 按住`shift`键进入grub启动菜单选择内核