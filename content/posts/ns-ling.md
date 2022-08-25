---
title: "ip-netns-ling"           # 文章标题
author: "陈金鑫"              # 文章作者
description : "ip-netns"    # 文章描述信息
lastmod: 2022-08-25        # 文章修改日期
date: 2022-08-25T1:38:18+08:00
tags : [                    # 文章所属标签
    "Linux",
    "网络"
]
categories : [              # 文章所属标签
    "Linux",
    "网络"
]
keywords : [                # 文章关键词
    "Linux",
    "网络"
]

next: http://changxiangyu.cn/2022/08/%E4%BF%AE%E6%94%B9%E5%8D%9A%E5%AE%A2%E4%B8%BB%E9%A2%98/      # 下一篇博客地址
prev: http://changxiangyu.cn/2020/08/%E5%9C%A8-ubuntu20.04-%E5%BB%BA%E8%AE%BE-openvpn/  # 上一篇博客地址
---
# Namespace Route LAB
![](/images/ip-netns.png)
![](https://secure2.wostatic.cn/static/j4AbdddxE4eoMBrknWnXcx/image.png)
# Target

三个NS中网卡地址都可以相互访问。

关键在创建恰当的路由，使其路由可达。

# Follow Me

## Check Env Status

```Bash
root@network-lab:~# ip netns list
root@network-lab:~# brctl show
```

## Create Bridge

```Bash
brctl addbr br-A-B # add bridge
ip link set br-A-B up # enable bridge

brctl addbr br-B-C # add bridge
ip link set br-B-C up # enable bridge

root@network-lab:~# brctl show
bridge name  bridge id    STP enabled  interfaces
br-A-B    8000.000000000000  no
br-B-C    8000.000000000000  no

root@network-lab:~# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:50:56:ac:a4:9b brd ff:ff:ff:ff:ff:ff
3: br-A-B: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether 76:8b:e0:6a:27:f8 brd ff:ff:ff:ff:ff:ff
4: br-B-C: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/ether a6:64:8b:a7:4c:2c brd ff:ff:ff:ff:ff:ff

```

## Create Net Namespace

```Bash
root@network-lab:~# ip netns list

ip netns add A
ip netns add B
ip netns add C

root@network-lab:~# ip netns list
C
B
A
```

## NS Status && Enable lo dev

```Bash
root@network-lab:~# ip -c -all netns exec ip a

netns: C
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

netns: B
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

netns: A
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    
root@network-lab:~# ip -c -all netns exec ip link set dev lo up

netns: C

netns: B

netns: A


root@network-lab:~# ip -c -all netns exec ip a

netns: C
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever

netns: B
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever

netns: A
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever

```



## Create Veth Pair && Connect Bridge NetNS via Veth Pair

```Bash
# NS A => br-A-B

ip link add A-To-br-A-B type veth peer name br-A-B-To-A
ip link set dev A-To-br-A-B netns A
ip netns exec A ip link set A-To-br-A-B up
ip link set dev br-A-B-To-A master br-A-B
ip link set br-A-B-To-A up

# NS B => br-A-B

ip link add B-To-br-A-B type veth peer name br-A-B-To-B
ip link set dev B-To-br-A-B netns B
ip netns exec B ip link set B-To-br-A-B up
ip link set dev br-A-B-To-B master br-A-B
ip link set br-A-B-To-B up

# NS B => br-B-C

ip link add B-To-br-B-C type veth peer name br-B-C-To-B
ip link set dev B-To-br-B-C netns B
ip netns exec B ip link set B-To-br-B-C up
ip link set dev br-B-C-To-B master br-B-C
ip link set br-B-C-To-B up

# NS C => br-B-C

ip link add C-To-br-B-C type veth peer name br-B-C-To-C
ip link set dev C-To-br-B-C netns C
ip netns exec C ip link set C-To-br-B-C up
ip link set dev br-B-C-To-C master br-B-C
ip link set br-B-C-To-C up

root@network-lab:~# brctl show
bridge name  bridge id    STP enabled  interfaces
br-A-B    8000.4e3feeec3e7c  no    br-A-B-To-A
                                   br-A-B-To-B
br-B-C    8000.06c767a60064  no    br-B-C-To-B
                                   br-B-C-To-C
root@network-lab:~# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:ac:a4:9b brd ff:ff:ff:ff:ff:ff
    inet 192.168.31.133/24 brd 192.168.31.255 scope global dynamic ens192
       valid_lft 36981sec preferred_lft 36981sec
    inet6 fe80::250:56ff:feac:a49b/64 scope link
       valid_lft forever preferred_lft forever
3: br-A-B: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 4e:3f:ee:ec:3e:7c brd ff:ff:ff:ff:ff:ff
    inet6 fe80::748b:e0ff:fe6a:27f8/64 scope link
       valid_lft forever preferred_lft forever
4: br-B-C: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 06:c7:67:a6:00:64 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::a464:8bff:fea7:4c2c/64 scope link
       valid_lft forever preferred_lft forever
19: br-A-B-To-A@if20: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-A-B state UP group default qlen 1000
    link/ether 4e:3f:ee:ec:3e:7c brd ff:ff:ff:ff:ff:ff link-netns A
    inet6 fe80::4c3f:eeff:feec:3e7c/64 scope link
       valid_lft forever preferred_lft forever
21: br-A-B-To-B@if22: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-A-B state UP group default qlen 1000
    link/ether be:e7:ef:32:94:6a brd ff:ff:ff:ff:ff:ff link-netns B
    inet6 fe80::bce7:efff:fe32:946a/64 scope link
       valid_lft forever preferred_lft forever
23: br-B-C-To-B@if24: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-B-C state UP group default qlen 1000
    link/ether 06:c7:67:a6:00:64 brd ff:ff:ff:ff:ff:ff link-netns B
    inet6 fe80::4c7:67ff:fea6:64/64 scope link
       valid_lft forever preferred_lft forever
25: br-B-C-To-C@if26: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-B-C state UP group default qlen 1000
    link/ether 32:e7:67:9e:8a:9b brd ff:ff:ff:ff:ff:ff link-netns C
    inet6 fe80::30e7:67ff:fe9e:8a9b/64 scope link
       valid_lft forever preferred_lft forever

root@network-lab:~# ip -all netns exec ip -c a

netns: C
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
26: C-To-br-B-C@if25: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 6e:5f:b1:c9:3c:09 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::6c5f:b1ff:fec9:3c09/64 scope link
       valid_lft forever preferred_lft forever

netns: B
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
22: B-To-br-A-B@if21: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 7a:4e:72:0a:0c:99 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::784e:72ff:fe0a:c99/64 scope link
       valid_lft forever preferred_lft forever
24: B-To-br-B-C@if23: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 32:ac:2b:9c:82:3f brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::30ac:2bff:fe9c:823f/64 scope link
       valid_lft forever preferred_lft forever

netns: A
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
20: A-To-br-A-B@if19: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 3e:6a:83:aa:96:23 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::3c6a:83ff:feaa:9623/64 scope link
       valid_lft forever preferred_lft forever

```



## Eable IP Forword

```Bash
ip netns exec B sysctl -w net.ipv4.conf.all.forwarding=1
```

## Assign IP

```Bash
ip netns exec A ip addr add 192.168.10.1/24 dev A-To-br-A-B
ip netns exec B ip addr add 192.168.10.10/24 dev B-To-br-A-B
ip netns exec B ip addr add 192.168.50.10/24 dev B-To-br-B-C
ip netns exec C ip addr add 192.168.50.1/24 dev C-To-br-B-C

root@network-lab:~# ip -all netns exec ip -c a

netns: C
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
26: C-To-br-B-C@if25: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 6e:5f:b1:c9:3c:09 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.50.1/24 scope global C-To-br-B-C
       valid_lft forever preferred_lft forever
    inet6 fe80::6c5f:b1ff:fec9:3c09/64 scope link
       valid_lft forever preferred_lft forever

netns: B
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
22: B-To-br-A-B@if21: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 7a:4e:72:0a:0c:99 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.10.10/24 scope global B-To-br-A-B
       valid_lft forever preferred_lft forever
    inet6 fe80::784e:72ff:fe0a:c99/64 scope link
       valid_lft forever preferred_lft forever
24: B-To-br-B-C@if23: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 32:ac:2b:9c:82:3f brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.50.10/24 scope global B-To-br-B-C
       valid_lft forever preferred_lft forever
    inet6 fe80::30ac:2bff:fe9c:823f/64 scope link
       valid_lft forever preferred_lft forever

netns: A
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
20: A-To-br-A-B@if19: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 3e:6a:83:aa:96:23 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.10.1/24 scope global A-To-br-A-B
       valid_lft forever preferred_lft forever
    inet6 fe80::3c6a:83ff:feaa:9623/64 scope link
       valid_lft forever preferred_lft forever
       
       
# check ip address       
root@network-lab:~# ip netns exec B ping 192.168.10.1 -c1
PING 192.168.10.1 (192.168.10.1) 56(84) bytes of data.
64 bytes from 192.168.10.1: icmp_seq=1 ttl=64 time=0.030 ms

--- 192.168.10.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.030/0.030/0.030/0.000 ms
root@network-lab:~# ip netns exec B ping 192.168.50.1 -c1
PING 192.168.50.1 (192.168.50.1) 56(84) bytes of data.
64 bytes from 192.168.50.1: icmp_seq=1 ttl=64 time=0.118 ms

--- 192.168.50.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.118/0.118/0.118/0.000 ms

# A access C - BAD
root@network-lab:~# ip netns exec A ping 192.168.50.1 -c1
ping: connect: Network is unreachable
root@network-lab:~# ip netns exec A ip r
192.168.10.0/24 dev A-To-br-A-B proto kernel scope link src 192.168.10.1

```

## Check Route

```Bash
root@network-lab:~# ip --all netns exec ip r

netns: C
192.168.50.0/24 dev C-To-br-B-C proto kernel scope link src 192.168.50.1

netns: B
192.168.10.0/24 dev B-To-br-A-B proto kernel scope link src 192.168.10.10
192.168.50.0/24 dev B-To-br-B-C proto kernel scope link src 192.168.50.10

netns: A
192.168.10.0/24 dev A-To-br-A-B proto kernel scope link src 192.168.10.1
```

## Add Route Make A access C

如何新增路由？

```Bash
# Tell A how to access C

ip netns exec A ip route add 192.168.50.0/24 via 192.168.10.10 dev A-To-br-A-B

# Tell C how to access A

ip netns exec C ip route add 192.168.10.0/24 via 192.168.50.10 dev C-To-br-B-C

root@network-lab:~# ip --all netns exec ip r

netns: C
192.168.10.0/24 via 192.168.50.10 dev C-To-br-B-C
192.168.50.0/24 dev C-To-br-B-C proto kernel scope link src 192.168.50.1

netns: B
192.168.10.0/24 dev B-To-br-A-B proto kernel scope link src 192.168.10.10
192.168.50.0/24 dev B-To-br-B-C proto kernel scope link src 192.168.50.10

netns: A
192.168.10.0/24 dev A-To-br-A-B proto kernel scope link src 192.168.10.1
192.168.50.0/24 via 192.168.10.10 dev A-To-br-A-B

# succeed
root@network-lab:~# ip netns exec A ping 192.168.50.1 -c 1
PING 192.168.50.1 (192.168.50.1) 56(84) bytes of data.
64 bytes from 192.168.50.1: icmp_seq=1 ttl=63 time=0.054 ms

--- 192.168.50.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.054/0.054/0.054/0.000 ms

```