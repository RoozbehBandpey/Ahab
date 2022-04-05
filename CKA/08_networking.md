# Networking

## Routing & Switches

List and modify interfaces on the host

```bash
ip link
```

See ip addresses assigned to those interfaces

```bash
ip addr
```

Set ip addresses on the interfaces
```bash
ip addr add 192.168.1.10/24 dev eth0
```

>Changes made by these commands are only valid until next restart

>To persist  the changes we must set them in the etc network interface file

View the routing table
```bash
route

or

ip route
```

Adding entries into the routing table

```bash
ip route add 192.168.1.10/24 via 192.168.2.1
```

Finally to check if ip forwarding is enabled on the host

```
cat /proc/sys/net/ipv4/ip_forward

1
```

## DNS on Linux

DNS name resolution is handled via a DNS server, the host will bee pointed to the server to look up the DNS names. Every host has a DNS configuration file at `etc/resolve.conf` with an entry into it specifying the address of the DNS server. 

```bash
cat /etc/resolve.conf

nameserver 192.168.1.100
```

Once this is set up across all hosts, every time that a name comes up that host does not know about it, it'll look it up from the DNS server.

the local  `etc/hosts` file has priority over the DNS server, meaning  if we have the same name in the host file and DNS server the file in the `etc/hosts` file is taken into account. 

The order is defined in `etc/nsswitch.conf` and can  be changed

```bash
cat /etc/nsswitch.conf

...
hosts:      files   dns
...
```

### Core DNS