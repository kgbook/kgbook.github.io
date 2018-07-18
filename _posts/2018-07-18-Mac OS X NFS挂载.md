---
layout: post
title: segfault 问题分析
date: 2018-07-18
tags: [数据通信]
---

### 配置`/etc/exports`

- 创建`/etc/exports`
```shell
sudo touch /etc/exports
sudo chmod +rwx /etc/exports
```
- 配置`/etc/exports`
  - `-maproot=user`，配置nfs登陆的用户、用户组
>-maproot=user The credential of the specified user is used for remote access by root.  The credential includes all the groups to which the user is a member on the local machine (
     see id(1) ). The user may be specified by name or number.

获取当前用户所属的用户组，root为wheel。

```shell
kang:kang root# groups
wheel daemon kmem sys tty operator procview procmod everyone staff certusers localaccounts admin com.apple.sharepoint.group.1 _appstore _lpadmin _lpoperator _developer _analyticsusers com.apple.access_ftp com.apple.access_screensharing com.apple.access_ssh
```
  - `-network`配置subnetwork
>The third component of a line specifies the host set to which the line applies.  The set may be specified in three ways.  The first way is to list the host name(s) separated by
     white space.  (Standard internet IPv4 ``dot'' addresses or IPv6 colon addresses may be used in place of names.)  The second way is to specify a ``netgroup'' as defined in the
     netgroup file (see netgroup(5) ). The third way is to specify an internet sub-network using a network and network mask that is defined as the set of all hosts with addresses
     within the sub-network.  This latter approach requires less overhead within the kernel and is recommended for cases where the export line refers to a large number of clients
     within an administrative sub-net.

我们使用的第三种方式; nfs主要用于局域网共享数据，需注意和设备端网段保持一致。
```shell
kang:~ kang$ cat /etc/exports
/Users/kang/project -maproot=root:wheel -network 10.2.11.0 -mask 255.255.255.0
```

设备端挂载，`mount -t nfs -o nolock 10.2.8.12:/Users/kang/project /home/nfs`。

打印所有挂载信息:
```shell
kang:~ kang$ showmount -a
All mounts on localhost:
10.2.11.186:/Users/kang/project
```

### 网络配置

Mac:
```shell
kang:~ kang$ ifconfig en0
en0: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
	ether 7c:04:d0:bd:24:5e 
	inet6 fe80::8ba:c991:c7c6:a48%en0 prefixlen 64 secured scopeid 0x5 
	inet 10.2.8.12 netmask 0xfffffc00 broadcast 10.2.11.255
	nd6 options=201<PERFORMNUD,DAD>
	media: autoselect
	status: active
```
arm-linux device: 

```shell
~ # ifconfig
eth0      Link encap:Ethernet  HWaddr E6:74:FC:61:62:2A  
          inet addr:10.2.11.186  Bcast:10.2.11.255  Mask:255.255.252.0
          inet6 addr: fe80::e474:fcff:fe61:622a/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:176378 errors:0 dropped:1883 overruns:0 frame:0
          TX packets:4651 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:22524298 (21.4 MiB)  TX bytes:4528148 (4.3 MiB)
          Interrupt:57 
```
首先，确认是否能ping通：
```shell
kang:~ kang$ ping 10.2.11.186
PING 10.2.11.186 (10.2.11.186): 56 data bytes
64 bytes from 10.2.11.186: icmp_seq=0 ttl=64 time=2.514 ms
64 bytes from 10.2.11.186: icmp_seq=1 ttl=64 time=2.903 ms
^C
--- 10.2.11.186 ping statistics ---
2 packets transmitted, 2 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 2.514/2.708/2.903/0.195 ms
```
可以ping通，确认是否同一网段：

如果两者网络号相同，则为同一网段；ip 与 netmask按位与的运算结果即为网络号（network id）。

Mac
> ip：10.2.8.12, netmask:0xfffffc00
```python
>>> 11 & 252
8
```
网络号为：10.2.8.0

arm-linux device 
>ip:10.2.11.186 netmask:255.255.252.0
```python
>>> 8 & 0xfc
8
```
网络号为：10.2.8.0

mac 设备和板端的网络号一致，认为两设备在同一网段。

### 参考资料
- [OS X Server：如何配置 NFS exports](https://support.apple.com/zh-cn/HT202243)
- [How to determine whether two IP addresses belong to the same network segment](https://stackoverflow.com/questions/13148747/determining-if-two-ip-adresses-are-on-same-subnet-is-it-leading-or-trailing-0s)
- `man exports`
- `man showmount`