# Service Networking Lab

What network range are the nodes in the cluster part of?

one way to do this is to make use of the `ipcalc` utility. If it is not installed, you can install it by running:
```bash
apt update

apt install ipcalc
```
Then use it to determine the network range as shown below:

First, find the Internal IP of the nodes.
```bash
root@controlplane:~# ip a | grep eth0 
4113: eth0@if4114: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default 
    inet 10.2.37.12/24 brd 10.2.37.255 scope global eth0
```

Next, use the ipcalc tool to see the network details:
```bash
root@controlplane:~# ipcalc -b 10.2.37.12/24                                   
Address:   10.2.37.12           
Netmask:   255.255.255.0 = 24   
Wildcard:  0.0.0.255            
=>
Network:   10.2.37.0/24         
HostMin:   10.2.37.1            
HostMax:   10.2.37.254          
Broadcast: 10.2.37.255          
Hosts/Net: 254                   Class A, Private Internet
```
What is the range of IP addresses configured for PODs on this cluster?
```bash
kubectl logs weave-net-gnkqd -c weave -n kube-system | grep range
INFO: 2022/04/07 13:05:39.675339 Command line options: map[conn-limit:200 datapath:datapath db-prefix:/weavedb/weave-net docker-api: expect-npc:true http-addr:127.0.0.1:6784 ipalloc-init:consensus=0 ipalloc-range:10.50.0.0/16 metrics-addr:0.0.0.0:6782 name:be:a4:c6:7a:e5:15 nickname:controlplane no-dns:true no-masq-local:true port:6783]
```

What is the IP Range configured for the services within the cluster?
```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep cluster-ip-range
    - --service-cluster-ip-range=10.96.0.0/12
```
What type of proxy is the kube-proxy configured to use?

```bash
kubectl logs kube-proxy-cczg8 -n kube-system | grep proxy
W0407 13:06:12.529585       1 proxier.go:661] Failed to load kernel module ip_vs_wrr with modprobe. You can ignore this message when kube-proxy is running inside container without mounting /lib/modules
W0407 13:06:12.533589       1 proxier.go:661] Failed to load kernel module ip_vs_sh with modprobe. You can ignore this message when kube-proxy is running inside container without mounting /lib/modules
I0407 13:06:12.591796       1 server_others.go:142] kube-proxy node IP is an IPv4 address (10.31.46.6), assume IPv4 operation
W0407 13:06:12.618835       1 server_others.go:578] Unknown proxy mode "", assuming iptables proxy
```
How does this Kubernetes cluster ensure that a kube-proxy pod runs on all nodes in the cluster?



Inspect the kube-proxy pods and try to identify how they are deployed


```bash
kubectl get ds -n kube-system
NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-proxy   2         2         2       2            2           kubernetes.io/os=linux   38m
weave-net    2         2         2       2            2           <none>                   38m
```