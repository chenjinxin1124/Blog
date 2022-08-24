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

# Linux一块网卡添加多个IP地址
https://cloud.tencent.com/developer/article/1431717