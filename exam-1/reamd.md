#DevOps Career Exam One
Make two two network namespaces using 'red' and 'green' names, connect them with a bridge and check connectivity. You have to successfully ping Google's public IP from those network namespaces.

##Step 1: Creating network namespaces
Use below command to create to namespaces
```bash
$ sudo ip netns add red
$ sudo ip netns add green
```
Show all namespaces
```bash
$ sudo ip netns list
```
##Step 2: Create virtual cabels
```bash
$ sudo ip link add red_veth type veth peer name red_veth_br
$ sudo ip link add green_veth type veth peer name green_veth_br
```
##Step 3: Connect cables with namespaces
```bash
$ sudo ip link set red_veth netns red
$ sudo ip link set green_veth netns green
```
##Step 3: Creating bridge
```bash
$ sudo ip link add rg_bridge type bridge
```
##Step 4: Connecting cables with bridge

```bash
$ sudo ip link set red_veth_br master rg_bridge
$ sudo ip link set green_veth_br master rg_bridge
```
##Step 5: Provide IP address to veths

```bash
$ sudo ip netns exec red ip addr add 192.20.0.1/16 dev red_veth
$ sudo ip netns exec green ip addr add 192.20.0.2/16 dev green_veth
```
##Step 6: Down to up all virtual devices

Currently all of virtual devices are in down, so we need to up all these devices

```bash
$ sudo ip link set rg_bridge up
$ sudo ip link set red_veth_br up
$ sudo ip link set green_veth_br up
$ sudo ip netns exec red ip link set dev lo up
$ sudo ip netns exec red ip link set dev red_veth up
$ sudo ip netns exec green ip link set dev lo up
$ sudo ip netns exec green ip link set green_veth lo up
```
##Step 7: Checking connectivity between namespaces

```bash
$ sudo ip netns exec red ping 192.20.0.2
$ sudo ip netns exec green ping 192.20.0.1
```

##Step 8: Provide IP address to birdge

```bash
$ sudo ip addr add 192.20.0.10/16 dev rg_bridge
```
##Step 9: Set default gateway in namespaces

```bash
$ sudo ip netns exec red ip route add default via 192.20.0.10
$ sudo ip netns exec green ip route add default via 192.20.0.10
```

##Step 10: Enable ip_forward

```bash
$ sudo sysctl -w net.ipv4.ip_forward=1
```

##Step 11: Adding a source NAT rule in the POSTROUTING chain

```bash
$ sudo iptables -t nat -A POSTROUTING -s 192.20.0.10/16 -j MASQUERADE
```
##Now check Google's public IP ping

```bash
$ sudo ip netns exec red ping 8.8.8.8
$ sudo ip netns exec green ping 8.8.8.8
```