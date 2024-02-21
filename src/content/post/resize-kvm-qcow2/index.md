---
title: "扩容KVM磁盘流程"
description: "记录一次qemu虚拟机磁盘映像扩容办法，云主机在扩容磁盘后也可通过Parted及后续命令完成磁盘扩容。"
publishDate: "21 Feb 2024"
tags: ["kvm", "qemu"]
---

记录一次qemu虚拟机磁盘映像扩容办法，云主机在扩容磁盘后也可通过Parted及后续命令完成磁盘扩容。

## 确认qcow2

#### 列出所有node

`$ virsh list --all`
![列出所有node](./1.png)

#### 查看qcow2磁盘位置

`virsh domblklist {node-id}`
![查看qcow2磁盘位置](./2.png)

#### 查看qcow2信息

`qemu-img info -U {file-name.qcow2}`
![查看qcow2磁盘位置](./3.png)

## 扩容qcow2

#### 关闭VM避免数据错误

`virsh destroy {node-name}`
![关闭VM避免数据错误](./4.png)

#### 扩容qcow2容量 + 100G

`qemu-img resize {filename.qcow2} +100G`
![扩容qcow2容量 + 100G](./5.png)

## 进入VM扩容分区

#### 开启VM

`virsh start {node-name}`
![开启VM](./6.png)

#### Parted 扩容分区

![Parted 扩容分区](./7.png)

#### 调整物理卷

`pvresize /dev/vda3`
![调整物理卷](./8.png)

#### 查看物理卷

`lvdisplay`
![查看物理卷](./9.png)

#### 扩容物理卷

`lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv`
![扩容物理卷](./10.png)

#### 重扫描分区

`sudo resize2fs /dev/ubuntu-vg/ubuntu-lv`
![重扫描分区](./11.png)

## 检查扩容成果

`df -h`
![检查扩容成果](./12.png)
