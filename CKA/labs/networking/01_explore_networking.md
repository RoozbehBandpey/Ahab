# Explore Networking Lab

What is the network interface configured for cluster connectivity on the controlplane node?



>node-to-node communication
```bash
ip a | grep -B2 10.46.143.9
20841: eth0@if20842: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default 
    link/ether 02:42:0a:2e:8f:09 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.46.143.9/24 brd 10.46.143.255 scope global eth0
```
What is the MAC address of the interface on the controlplane node?
```bash
ip link show eth0
20841: eth0@if20842: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP mode DEFAULT group default 
    link/ether 02:42:0a:2e:8f:09 brd ff:ff:ff:ff:ff:ff link-netnsid 0
```
What is the MAC address assigned to node01?
```bash
arp node01
Address                  HWtype  HWaddress           Flags Mask            Iface
10.46.143.11             ether   02:42:0a:2e:8f:0a   C                     eth0
```
We use Docker as our container runtime. What is the interface/bridge created by Docker on this host?

What is the state of the interface docker0?
```bash
ip link show docker0
2: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default 
    link/ether 02:42:f5:ef:24:bd brd ff:ff:ff:ff:ff:ff
```
If you were to ping google from the controlplane node, which route does it take?



What is the IP address of the Default Gateway?
```bash
ip route show default
default via 172.25.0.1 dev eth1 
```
```bash
dig google.com

; <<>> DiG 9.11.3-1ubuntu1.15-Ubuntu <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 53175
;; flags: qr rd ra; QUERY: 1, ANSWER: 6, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             300     IN      A       142.250.128.138
google.com.             300     IN      A       142.250.128.139
google.com.             300     IN      A       142.250.128.101
google.com.             300     IN      A       142.250.128.100
google.com.             300     IN      A       142.250.128.102
google.com.             300     IN      A       142.250.128.113

;; Query time: 2 msec
;; SERVER: 172.25.0.1#53(172.25.0.1) # <-------------------> Here
;; WHEN: Wed Apr 06 18:17:55 UTC 2022
;; MSG SIZE  rcvd: 135
```
What is the port the kube-scheduler is listening on in the controlplane node?
```bash
netstat -nplt | grep scheduler
tcp        0      0 127.0.0.1:10259         0.0.0.0:*               LISTEN      3809/kube-scheduler 
```

Notice that ETCD is listening on two ports. Which of these have more client connections established?
```bash
netstat -anp | grep etcd | grep 2380 | wc -l 
1
root@controlplane:~# netstat -anp | grep etcd | grep 2379 | wc -l 
81
```
>That's because 2379 is the port of ETCD to which all control plane components connect to. 2380 is only for etcd peer-to-peer connectivity. When you have multiple controlplane nodes. In this case we don't.