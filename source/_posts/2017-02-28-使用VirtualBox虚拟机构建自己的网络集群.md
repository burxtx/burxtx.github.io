---
title: 使用VirtualBox虚拟机构建自己的网络集群
date: 2017-02-28 18:16:53
categories:
tags:
  - 虚拟机
  - 网络
---
最近在学习搭建Hadoop分布式系统，折腾一番后发现，利用虚拟机搭建实验环境真是很快。这里介绍一下虚拟机集群常用的网络类型。

## Bridge
最初我选择了桥接模式，这种方式是把虚拟机和主机归入一个局域网络中，虚拟机要设置成静态IP，主机和虚拟机之间互相可以访问，虚拟机可以上网。
缺点是虚拟机占用局域网的ip资源，而且如果换了网络环境，需要重新更改ip地址。

## Host-only
这种方式与桥接类似。这种方式需要创建一个虚拟以太网，然后主机和虚拟机都利用这块网卡通信。
优点是这个局域网是虚拟的，因此可以在主机上创建多个局域网环境，不依赖物理网络。

先在preference里添加一块host-only网卡，如果安装virtualbox时自动添加了，可以忽略。
检查这块网卡的DHCP server是否启用。若禁用，需要先启用它，否则你的虚拟机不会被分配对应的IP地址。如下图所示
![](http://ou7hg0tk3.bkt.clouddn.com/host-only%20network%20details.jpg)

然后在虚拟机的settings->network选项中，启用adapter1，选择NAT，再启用adapter2，选择Host-only mode，保存，启动虚拟机，大功告成。
虚拟机启动后，分别在主机和虚拟机ping检测，成功！

我用的是最新的ubuntu 16.04.2 server，启动虚拟机后检查网卡`ls /sys/class/net`，会发现网卡名不是eth0,eth1这样了，而是enp0s3，enp0s8，原因是ubuntu 16使用了新的网卡命名规范
