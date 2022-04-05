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

