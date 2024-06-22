# Setting Gateway as Router


https://gist.github.com/gilangvperdana/cbcda28d11f186ebfe785dbfa19688c4

```
    |wifi (wlp3s0) 192.168.1.1/24
    |
|-------+
|gateway|----enp2s0 (192.168.1.10/24) 
|       |
|-------+
    |                       
    | ethernet(enp2s0) 10.10.10.1/24
    |
    |
    | 
    |-----------------------+
    |10.10.10.20/24         |
+-------+                +-------+ 
|  c1   |                |  c2   |
|       |                |       |
+-------+                +-------+


```
Ubuntu 22.04 LTS
* 2 Interface
* 1 Interface from WIFI/WAN/ISP (wlp3s0)
* 1 Interface for distribution clients (enp2s0)

Set netplan
```
network:
  ethernets:
    enp1s0:
      dhcp4: true
    enp2s0:
      addresses:
        - 10.10.10.1/24
      routes:
        - to: default
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
  version: 2
```

```
sudo netplan generate
sudo netplan apply
```

Install DHCP Server

```
sudo apt-get install isc-dhcp-server
```

Then edit dhcpd configure
```
sudo nano /etc/default/isc-dhcp-server
```

Declare Interface to User

```
INTERFACESv4="enp2s0"
```

## Configure pool for DHCP

Edit dhcp client pool 
/etc/dhcp/dhcpd.conf 
```
# A slightly different configuration for an internal subnet.
subnet 10.10.10.0 netmask 255.255.255.0 {
  range 10.10.10.10 10.10.10.200;
  option domain-name-servers 8.8.8.8, 8.8.4.4;
  option subnet-mask 255.255.255.0;
  option routers 10.10.10.1;
  option broadcast-address 10.10.10.255;
  default-lease-time 600;
  max-lease-time 7200;
}
```

## Then create Auto Forwording Rule

```
# Default policy to drop all incoming packets.
#iptables -P INPUT DROP
#iptables -P FORWARD DROP

# Accept incoming packets from localhost and the LAN interface.
$ sudo iptables -A INPUT -i lo -j ACCEPT
$ sudo iptables -A INPUT -i enp2s0 -j ACCEPT

# Accept incoming packets from the WAN if the router initiated the connection.
$ sudo iptables -A INPUT -i wlp3s0 -m conntrack  --ctstate ESTABLISHED,RELATED -j ACCEPT

# Forward LAN packets to the WAN.
$ sudo iptables -A FORWARD -i enp2s0 -o wlp3s0 -j ACCEPT

# Forward WAN packets to the LAN if the LAN initiated the connection.
$ sudo iptables -A FORWARD -i wlp3s0 -o enp2s0 -m conntrack \
--ctstate ESTABLISHED,RELATED -j ACCEPT

# NAT traffic going out the WAN interface.
$ sudo iptables -t nat -A POSTROUTING -o wlp3s0 -j MASQUERADE
```

Check route table on server
```
sudo iptables -L
```

# Check dhcp lease

```
dhcp-lease-list
```

check ip route
```
ip route show
```

# Client side
gets an IP address via DHCP.
```
dhclient -v eth0
```

## Configuring networks

https://ubuntu.com/server/docs/configuring-networks


Identify Ethernet interfaces
```
$ ip address

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether b8:27:eb:1a:8f:7d brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.10/24 brd 10.10.10.255 scope global dynamic eth0
       valid_lft 413sec preferred_lft 413sec

```

## IP addressing
The following section describes the process of configuring your system’s IP address and default gateway needed for communicating on a local area network and the Internet.

Temporary IP address assignment
```
sudo ip addr add 10.10.10.20/24 dev eth0
```

The ip can then be used to set the link up or down.
```
ip link set dev eth0 up
ip link set dev eth0 down
```

To verify the IP address configuration of enp0s25, you can use the ip command in the following manner:
```
$ ip address show dev eth0

2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether b8:27:eb:1a:8f:7d brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.10/24 brd 10.10.10.255 scope global dynamic eth0
```

To configure a default gateway, you can use the ip command in the following manner. Modify the default gateway address to match your network requirements.

```
$ sudo ip route add default via 10.102.66.1
```
You can also use the ip command to verify your default gateway configuration, as follows:

```
$ ip route show

default via 10.102.66.1 dev eth0 proto dhcp src 10.102.66.200 metric 100
10.102.66.0/24 dev eth0 proto kernel scope link src 10.102.66.200
10.102.66.1 dev eth0 proto dhcp scope link src 10.102.66.200 metric 100 
```

If you require DNS for your temporary network configuration, you can add DNS server IP addresses in the file /etc/resolv.conf. In general, editing /etc/resolv.conf directly is not recommended, but this is a temporary and non-persistent configuration. The example below shows how to enter two DNS servers to /etc/resolv.conf, which should be changed to servers appropriate for your network. A more lengthy description of the proper (persistent) way to do DNS client configuration is in a following section.
```
nameserver 8.8.8.8
nameserver 8.8.4.4
```
If you no longer need this configuration and wish to purge all IP configuration from an interface, you can use the ip command with the flush option:
```
ip addr flush eth0
```

