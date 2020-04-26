---
title: 虚拟机静态IP配置
date: 2020-04-26 17:46:47
tags: linux,虚拟机
---

创建一个虚拟机，

如果是克隆的虚拟机，记得重新生成一些HWADDR，再改配置文件
<!--more-->
修改网络配置
vi /etc/sysconfig/network-script/ifcfg-ens33
DEVICE=eth33       		#网卡名称
HWADDR=00:0C:29:42:15:C2     #MAC硬件地址
TYPE=Ethernet		#网络类型以太网
ONBOOT=yes		#开机自动启用网络连接
NM_CONTROLLED=yes	#是否托管
BOOTPROT=static		#模式   dhcp自动分配ip  static固定ip
IPADDR=192.168.0.100	#IP地址
NETMASK=255.255.255.0	#子忘掩码
GATEWAY=192.168.0.1	#网关
DNS1=8.8.8.8		#DNS
DNS2=114.114.114.114	#DNS
重启服务即可
systemctl  restart  netwrok.service

详细连接
https://blog.csdn.net/woailyoo0000/article/details/79506999







