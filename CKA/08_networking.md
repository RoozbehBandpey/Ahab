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

CoreDNS binaries can be downloaded from their Github releases page or as a docker image. Let’s go the traditional route. Download the binary using curl or wget. And extract it. You get the coredns executable.

Run the executable to start a DNS server. It by default listens on port `53`, which is the default port for a DNS server.



Now we haven’t specified the IP to hostname mappings. For that you need to provide some configurations. There are multiple ways to do that. We will look at one. First we put all of the entries into the DNS servers `/etc/hosts` file.

And then we configure CoreDNS to use that file. CoreDNS loads it’s configuration from a file named Corefile. Here is a simple configuration that instructs CoreDNS to fetch the IP to hostname mappings from the file /etc/hosts. When the DNS server is run, it now picks the Ips and names from the /etc/hosts file on the server.


## Network Namespaces

Network namespaces are used by containers like docker to implement network isolation. Containers are separated from underlying host with namespaces. This can be seen when we list processes from within a container or as a root user on the host:

```bash
ps aux
```
When it comes to networking the host has its own interfaces when connects to LAN. as well as route tables. When we create a container on the host, the container will have its  own network namespace therefore it has no visibility to the networking components of the host. Within the network namespace a container can have its own virtual interface, routing and  ARP table. 

To  create a new network namespace run:

```bash
ip netns add red
```

To list the network namespaces run:

```bash
ip netns

red
```

To list the interfaces on the host
```bash
ip link
```
To list the interfaces on the `red` container:

```bash
ip netns exec red ip link
```
or
```bash
ip -n red link
```

To view the ARP table inside the container:

```bash
ip netns exec red arp
```
To view the routing table inside the container:

```bash
ip netns exec red route
```

To establish connectivity between namespaces themselves, just like how we would connect two physical machines together using a cable  to an ethernet interface, we can connect two namespaces using a virtual ethernet pair or a virtual cable. It is often refer to as a pipe.

To create the cable run the:

```bash
ip link add veth-red type veth peer name veth-blue
```
Next we have  to attach each  interface to the appropriate namespace:

```bash
ip link set veth-red netns red
```
```bash
ip link set veth-blue netns blue
```
Then we can assign ip addresses to each of these namespaces
```bash
ip  -n red addr  192.168.15.1 dev veth-red
```
```bash
ip  -n blue addr  192.168.15.2 dev veth-blue
```
Then we bring up each interfaces

```bash
ip -n red link set veth-red up
```
```bash
ip -n blue link set veth-blue up
```

For testing try to ping the ip of the blue from the red namespace

```bash
ip netns exec red ping 192.168.15.2
```

If we have various namespaces we have create  a virtual network inside the host, to create a virtual switch we can use `Linux Bridge` or `Open vSwitch` 

With `Linux Bridge` we have to first add a new interface to the host with thee type set tto `bridge`:

```bash
ip link add v-net-0 type bridge
```

## Networking in Docker

A docker host (a server  with docker installed on it) has an ethernet interface `eth0` that connects to the LAN. When you run a container you have different networking options to choose from `docker run --network none nginx`

* `none`: The docker container is not attached to any network, the container cannot reach  the outside world and no one from outside world can reach the container. 
* `host`: The container is attached to the host network, if you deploy an application on the port `80` of  the container then the container is reachable on the port `80` of the host without any additional port mapping. If we try to run another instance of the container that listens on the same port, it won't work as they share the host networking.
* `bridge`: An internal private network get created which the containers and docker host attach to it. Each device connecting to this bridge get their own private IP address. 

When docker is installed on a host, it'll create a virtual private network called `bridge` you can see this when you run `docker network ls` command. On the host the network is created by the name `docker0` you can see this in the output of `ip link` command.

Whenever a container is created, docker creates a network namespace for it. To list the namespaces run `ip netns` 

## Container Networking Interface (CNI)

The CNI is set of standards that defines how programs should be developed to solve the networking challenges, in a container runtime environment. The program is referred to as a plugin, in this case the `bridge` is a plugin for CNI. CNI defines a set of responsibilities for container runtimes and pliugins. For container runtime CNI specifies that it is responsible for creating a namespace for each container.  


## Cluster Networking

The kubernetes cluster consist of master and worker nodes, each node must have at least one network interface connected to the private network. Each interface must have an address configured. The host must have a unique hostname set as well as a unique mac address. 

There are some ports that need to be opened as well, The master should accept connections on the `6443` for the api-server, The kubelet on master and worker nodes listen on port `10250` the kube-scheduler requires the `10251`, the kube-controller-manager requires the `10252`.

The worker nodes exposes services for external access on ports `30000` to `32767`

The ETCD server is on its own port `2379`


## Pod Networking

Kubernetes expects every pod to get its own unique IP address. Every pod should be able to reach every other pod within the same node using IP addresses. Every pod should be able to reach every other pod on other nodes as well. 

There are many networking solution out there that implement this model. 

## CNI in Kubernetes

The CNI plugin must be invoked by components within kubernetes responsible for creating containers, because that component must invoke the appropriate network plugin after the container is created. The CNI plugin is configured within kubelet service within each node in the cluster. If you look at the kubelet service file `kubelet.service` you'll see an option set to CNI `--network-plugin=cni` You can also view this option in kubelet service `ps aux | grep kubelet`.

## WeaveWorks

## IP Address Management - Weave

CNI says it is the responsibility of the CNI plugin (the network solution provider) to take care of assigning IPs to the containers. 

Kubernetes does not care how we assign IPs, we just need to make sure there's no duplicate IPs

An easy way to do it is to store the list of IPs in a file, and make sure we have the necessary code in our script to manage this file properly. This file will be placed on each host and manages the IP of the pods on each node. 

CNI comes with two built-in plugins which we can outsource this task to.
* `DHCP`
* `host-local`

We have to invoke this plugin in our script,  the CNI specification file has a section called `IPAM` which specify the type of plugin to be used. `cat /etc/cni/net.d/net-script.conf`

This details can be red from our script, to use the right plugin instead of hardcoding it. 

Weave by default allocate the IP range `10.32.0.0/12` for the entire network. 

## Service Networking

In practice you'd rarely configure pods to connect to each other, if you want a pod to access another pod you would use a service. 

In order to make orange pod accessible to the blue pod, we have to make an orange service. The orange service gets an IP address and a name assign to it, the blue pod can now access orange pod through orange service's IP or its name. 

When a service is created it is accessible from all pods in the cluster, irrespective of what nodes the pods are on. 

A clusterIP service is only accessible from within the cluster. It's useful for services such as databases. 

A nodePort service is accessible from outside of the cluster, It's useful for services such as web applications. It works just like clusterIP but in addition it exposes the application on all nodes in the cluster.

Kubelet is responsible for creating pods on each node, kubelet watches the changes in the cluster through the kube api-server. After creating pods it will invoke the CNI plugin to configure networking for that pod. Kube-proxy watches the changes in the cluster through the kube api-server and every time a new service is to be created, kube-proxy gets into action. Unlike pods, services are not limited to nodes they are a cluster-wide concept. 

When we create a service object in kubernetes, it is assigned an IP address from a predefined range, the kube-proxy gets that IP address, and create forwarding rules on each node in the cluster. Saying any traffic coming to this IP should go to the IP of the pod.  Whenever a service is created or deleted, the kube-proxy creates or deletes these rules. 


kube-proxy supports different ways such as `userspace` where it listens on a port for each service and proxies the connection to a pod. `ipvs` rules, or more familiar option is `iptables`. 

The proxy mode can be set while configuring kube-proxy service.

```bash
kube-proxy --proxy-mode [userspace | iptables | ipvs]
```
The default is `iptables` 

You can see the rules created by kube proxy in the IP table NAT table output. 

```bash
iptables -L -t nat | grep <service-name>
```

 