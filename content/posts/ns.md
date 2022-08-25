A               B                                 C
192.168.10.1    192.168.10.10   192.168.50.10     192.168.50.1

ip netns exec A ip addr add 192.168.30.246/24 dev A-To-br-A-B
ip netns exec B ip addr add 192.168.30.1/24 dev B-To-br-A-B
ip netns exec B ip addr add 192.168.40.1/24 dev B-To-br-B-C
ip netns exec C ip addr add 192.168.40.133/24 dev C-To-br-B-C

ip netns exec B ping 192.168.30.246 -c1
ip netns exec B ping 192.168.40.133 -c1

ip netns exec A ping 192.168.30.246 -c1
ip netns exec A ping 192.168.40.133 -c1

ip netns exec C ping 192.168.30.246 -c1
ip netns exec C ping 192.168.40.133 -c1

ip netns exec A ip route add 192.168.50.0/24 via 192.168.10.10 dev A-To-br-A-B
ip netns exec A ip route add 192.168.40.0/24 via 192.168.30.1 dev A-To-br-A-B

ip netns exec C ip route add 192.168.10.0/24 via 192.168.50.10 dev C-To-br-B-C
ip netns exec C ip route add 192.168.30.0/24 via 192.168.40.1 dev C-To-br-B-C

ip netns exec A ping 192.168.40.133 -c 1
