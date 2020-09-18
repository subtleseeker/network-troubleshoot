brew install iproute2mac

### Layer 1: PL
```
$ ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast state DOWN mode DEFAULT group default qlen 1000
link/ether 52:54:00:82:d6:6e brd ff:ff:ff:ff:ff:ff
$ ip link set eth0 up
```
Interfaces can negotiate at the incorrect speed, or collisions and physical layer problems can cause packet loss or corruption that results in costly retransmissions.
```
$ ip -s link show eth0              # not present in mac
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
link/ether 52:54:00:82:d6:6e brd ff:ff:ff:ff:ff:ff
RX: bytes packets errors dropped overrun mcast
34107919 5808 0 6 0 0
TX: bytes packets errors dropped carrier collsns
434573 4487 0 0 0 0
```

ethtool  (not present in mac)
```
$ ethtool eth0
```

### Layer 2: DLL
ARP mappings
Linux caches the ARP entry for a period of time, so you may not be able to send traffic to your default gateway until the ARP entry for your gateway times out. You can manually delete an ARP entry, which will force a new ARP discovery process:

```
$ ip neighbor show
192.168.122.170 dev eth0 lladdr 52:54:00:04:2c:5d REACHABLE
192.168.122.1 dev eth0 lladdr 52:54:00:11:23:84 REACHABLE
$ ip neighbor delete 192.168.122.170 dev eth0
$ ip neighbor show
192.168.122.1 dev eth0 lladdr 52:54:00:11:23:84 REACHABLE
```

### Layer 3: NL
```
ip addr show
ping www.google.com
traceroute www.google.com
ip route show
```

##### DNS (not Layer 3 though)
```
nslookup google.com
dig google.com
```
if nslookup and ping use different address, check /etc/hosts.

### Layer 4: TL
The first thing that you may want to do is see what ports are listening on the localhost. The result can be useful if you can’t connect to a particular service on the machine, such as a web or SSH server.
```
$ ss -tunlp4       # not for mac
Netid State Recv-Q Send-Q Local Address:Port Peer Address:Port
udp UNCONN 0 0 *:68 *:* users:(("dhclient",pid=3167,fd=6))
udp UNCONN 0 0 127.0.0.1:323 *:* users:(("chronyd",pid=2821,fd=1))
tcp LISTEN 0 128 *:22 *:* users:(("sshd",pid=3366,fd=3))
tcp LISTEN 0 100 127.0.0.1:25 *:* users:(("master",pid=3600,fd=13))
```
Let’s break down these flags:

-t - Show TCP ports.
-u - Show UDP ports.
-n - Do not try to resolve hostnames.
-l - Show only listening ports.
-p - Show the processes that are using a particular socket.
-4 - Show only IPv4 sockets.

telnet: ping for ports: uses TCP
```
telnet database.example.com 3306
```

what about UDP? The netcat tool provides a simple way to check a remote UDP port:
```
# nc 192.168.122.1 -u 80
test
Ncat: Connection refused.
```

nmap
    TCP and UDP port scanning remote machines.
    OS fingerprinting.
    Determining if remote ports are closed or simply filtered.


---

iftop - show big users on the network/router.

---
Ping without DNS resolution
```
ping -n w.x.y.z
```
Steps:
    Your network interface is listed and enabled. Otherwise, check the device driver – see /Ethernet#Device driver or /Wireless#Device driver.
    You are connected to the network. The cable is plugged in or you are connected to the wireless LAN.
    Your network interface has an IP address.
    Your routing table is correctly set up.
    You can ping a local IP address (e.g. your default gateway).
    You can ping a public IP address (e.g. 8.8.8.8, which is a Google DNS server and is a convenient address to test with).
    Check if you can resolve domain names (e.g. archlinux.org).