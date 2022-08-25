---
title: "iptables"           # 文章标题
author: "陈金鑫"              # 文章作者
description : "iptables"    # 文章描述信息
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
sysctl -w net.ipv4.ip_forward=1
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.3.63:80
iptables -t nat -A POSTROUTING -p tcp -d 192.168.3.63 --dport 80 -j SNAT --to-source 192.168.3.243

sysctl -w net.ipv4.ip_forward=1
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 172.17.0.2:80
iptables -t nat -A POSTROUTING -p tcp -d 172.17.0.2 --dport 80 -j SNAT --to-source 192.168.3.63

iptables -t nat -A PREROUTING -d 192.168.3.63 -p tcp --dport 80 -j DNAT --to-destination 172.17.0.2:80
iptables -t nat -A POSTROUTING -d 172.17.0.2 -p tcp --dport 80 -j SNAT --to 192.168.3.63

B:
iptables -t nat -A POSTROUTING -s 192.168.3.0/24 -d 172.17.0.0/24 -o docker0 -j MASQUERADE
iptables -t nat -A POSTROUTING -s 172.17.0.0/24 -d 192.168.3.0/24 -o enp0s3 -j MASQUERADE

A: 
route add -net 172.17.0.0 netmask 255.255.255.0 gw 192.168.3.63

C: 
route add -net 192.168.5.0 netmask 255.255.255.0 gw 192.168.3.63




echo 1 > /proc/sys/net/ipv4/ip_forward
B:
iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -d 192.168.122.0/24 -o virbr0 -j MASQUERADE
iptables -t nat -A POSTROUTING -s 192.168.122.0/24 -d 192.168.100.0/24 -o virbr1 -j MASQUERADE

A: 
iptables -t nat -A POSTROUTING -s 192.168.122.0/24 -d 192.168.100.0/24 -o enp1s0 -j MASQUERADE

route add -net 192.168.100.0 netmask 255.255.255.0 gw 192.168.122.1 enp1s0

C: 
route add -net 192.168.122.0 netmask 255.255.255.0 gw 192.168.100.1 enp1s0

A: 
route add -net 192.168.100.0 netmask 255.255.255.0 gw 192.168.100.1

C: 
route add -net 192.168.122.0 netmask 255.255.255.0 gw 192.168.122.1


iptables -A FORWARD -i virbr1 -o virbr0 -j ACCEPT
iptables -A FORWARD -i virbr0 -o virbr1 -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -t nat -A POSTROUTING -o virbr0 -j MASQUERADE



iptables -t nat -vnL POSTROUTING --line-number
iptables -t nat -D POSTROUTING 20
iptables-save
route
route del -net 192.168.40.0 netmask 255.255.255.0 
route del -net 192.168.30.0 netmask 255.255.255.0

route del -net 192.169.30.0 netmask 255.255.255.0 dev virbr0 
route del -net 192.169.40.0 netmask 255.255.255.0 dev virbr1


B:
echo 1 > /proc/sys/net/ipv4/ip_forward
wlo1
iptables -t nat -A POSTROUTING -s 192.168.20.0/24 -d 192.168.40.0/24 -o wlo1 -j MASQUERADE
iptables -t nat -A POSTROUTING -s 192.168.20.0/24 -d 192.168.40.0/24 -o virbr1 -j MASQUERADE
iptables -t nat -A POSTROUTING -s 192.168.30.0/24 -d 192.168.40.0/24 -o virbr1 -j MASQUERADE
iptables -t nat -A POSTROUTING -s 192.168.40.0/24 -d 192.168.30.0/24 -o virbr0 -j MASQUERADE

route add -net 192.168.40.0 netmask 255.255.255.0 gw 192.168.40.205
route add -net 192.168.30.0 netmask 255.255.255.0 gw 192.168.30.235

route add -net 192.169.30.0 netmask 255.255.255.0 dev virbr0
route add -net 192.169.40.0 netmask 255.255.255.0 dev virbr1


A: 
route add -net 192.168.40.0 netmask 255.255.255.0 gw 192.168.30.1 

route del -net 192.168.40.0 netmask 255.255.255.0 
route add -net 192.168.40.0 netmask 255.255.0.0 gw 192.168.3.34 dev enp1s0

C: 
route add -net 192.168.30.0 netmask 255.255.255.0 gw 192.168.40.1 

route del -net 192.168.30.0 netmask 255.255.255.0
route add -net 192.168.30.0 netmask 255.255.0.0 gw 192.168.3.34

# Linux一块网卡添加多个IP地址
https://cloud.tencent.com/developer/article/1431717

https://virt-manager.org/


# 黄总
sudo ip addr add 10.252.3.88/29 dev ens1f1
sudo route add 88.88.43.8 gw 10.246.1.173
# sudo ip route add 88.88.43.0/24 via 10.246.1.173
sudo iptables -t nat -A PREROUTING -i ens1f1 -d 10.252.3.88 -j DNAT --to 88.88.43.8
sudo iptables -t nat -A POSTROUTING -d 88.88.43.8 -o ens1f1  -j SNAT --to-source 10.252.3.88

# 查看登录日志
sudo tail /var/log/auth.log