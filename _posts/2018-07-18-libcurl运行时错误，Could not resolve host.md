---
layout: post
title: libcurl运行时报错"Cloud not resolve host"
date: 2018-07-18
tags: [数据通信]
---
### 现象
- 设备端运行程序，报错 "Could not resolve host: api.bi.tuputech.com"。
- 设备端, 域名无法ping通，ip也无法ping通。
```shell
~ # ping api.bi.tuputech.com
ping: bad address 'api.bi.tuputech.com'

~ # ping  183.60.177.251
PING 183.60.177.251 (183.60.177.251): 56 data bytes
ping: sendto: Network is unreachable
```
- pc 端, 可以ping通。
```shell
kang:~ kang$  ping api.bi.tuputech.com
PING api.bi.tuputech.com (183.60.177.251): 56 data bytes
64 bytes from 183.60.177.251: icmp_seq=0 ttl=38 time=40.709 ms
64 bytes from 183.60.177.251: icmp_seq=1 ttl=38 time=42.530 ms
64 bytes from 183.60.177.251: icmp_seq=2 ttl=38 time=41.461 ms
^C
--- api.bi.tuputech.com ping statistics ---
3 packets transmitted, 3 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 40.709/41.567/42.530/0.747 ms
```
### 定位问题
1. ping: bad address

```shell
~ # ls /etc/resolv.conf
ls: /etc/resolv.conf: No such file or directory
```
DNS 客户机配置文件`/etc/resolv.conf`不存在, ping 域名时找不到 DNS 服务器!

配置如下:
```shell
nameserver 211.162.66.66
nameserver 211.162.78.1
```

2. ping: sendto: Network is unreachable

pc 可以 ping通，从 pc 端入手，跟踪其路由记录：
```shell
kang:~ kang$ traceroute api.bi.tuputech.com
traceroute to api.bi.tuputech.com (183.60.177.251), 64 hops max, 52 byte packets
 1  10.2.8.1 (10.2.8.1)  3.338 ms  4.933 ms  7.258 ms
 2  10.2.15.250 (10.2.15.250)  3.372 ms  6.810 ms  4.096 ms
 3  * * *
 4  * * *
^C
```

board 仍然无法ping 通，路由记录：
```shell
/home # traceroute api.bi.tuputech.com
traceroute: bad address 'api.bi.tuputech.com'
```

board 路由表：
```shell
/home # route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
10.2.8.0        0.0.0.0         255.255.252.0   U     0      0        0 eth0
```

pc 路由表：
```shell
kang:~ kang$ route get default
   route to: default
destination: default
       mask: default
    gateway: 10.2.8.1
  interface: en0
      flags: <UP,GATEWAY,DONE,STATIC,PRCLONING>
 recvpipe  sendpipe  ssthresh  rtt,msec    rttvar  hopcount      mtu     expire
       0         0         0         0         0         0      1500         0 
```
board 可以 ping 通 `10.2.8.1`，gateway没配置；结合pc 端路由记录，以及路由表，认为`10.2.8.1`应该设为默认网关。

```shell
/home # route add default gw 10.2.8.1

/home # route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.2.8.1        0.0.0.0         UG    0      0        0 eth0
10.2.8.0        0.0.0.0         255.255.252.0   U     0      0        0 eth0
```

### 解决方案
1. 配置`/etc/resolv.conf`。
2. 配置route table。

### 参考资料
- [Why does libcurl sometimes complain 'Couldn't resolve host name'?](https://stackoverflow.com/questions/43450899/why-does-libcurl-sometimes-complain-couldnt-resolve-host-name)
- [What Is A Domain Name Server (DNS) And How Does It Work](http://www.networksolutions.com/support/what-is-a-domain-name-server-dns-and-how-does-it-work/)
- [ping: bad address 'google.com'](https://www.toradex.com/community/questions/8325/ping-bad-address-googlecom.html)
- [resolv.conf wiki](https://en.wikipedia.org/wiki/Resolv.conf)
- [network is unreachable](https://www.linuxquestions.org/questions/linux-networking-3/network-is-unreachable-81728/)
- [Linux route command](https://www.computerhope.com/unix/route.htm)