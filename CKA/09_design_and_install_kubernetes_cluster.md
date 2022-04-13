# Design and Install Kubernetes Cluster

## Design a kubernetes Cluster
What is the purpose of the cluster? Learning, development, testing or serving production-grade applications

Kind of workload:
* How many applications are going to be hosted on the cluster?
* What kind of application, web application, big data / analytics?
* Resource requirements, cpu intensive, memory intensive?
* What type of network traffic is the application expected to handle, heavy traffic, burst traffic?

For the learning purpose a solution based on minikube or a single node cluster should do.

For hosting production grade applications a highly available cluster with multiple master nodes and multiple nodes is recommended.

* We can have up to 5000 nodes in the cluster
* Up to 150,000 pods in the cluster
* Up to 300,000 containers
* Up to 100 pods per nodes

### Storage consideration

* For high performance workload use SSD backed storage
* Multiple concurrent access consider network based storage
* For shared access across multiple pods consider persistent shared volumes
* Label nodes with specific disk types
* Use node selectors to assign applications to nodes  with specific disk types

### Node consideration

* The nodes forming a kubernetes cluster can be physical for virtual
* Minimum of 4 node cluster (choice based on workload)
* Best practice is  not to host workload on master nodes
* Linux X86_64 architecture
* Kubeadm  prevent workload to be hosted on master node by adding a taint to the master node
* In large environments we might choose to dedicate a separate node to ETCD cluster

## Install Kubernetes with Kubeadm

First you must have multiple systems or virtual machines provisioned for the kubernetes cluster. 

Once the systems are created, designate one node as master and others as worker nodes.

Next we install a container runtime on the hosts, for example we install docker on all the nodes.

Next we install kubeadm on all the nodes, the kubeadm helps us bootstrap kubernetes solution by installing and configuring all the required components.

Then  we initialize the master server, during this process all the required components are installed and configured on the master.

Before joining the workers we must ensure that network prerequisites are met. A normal network connectivity between systems is not sufficient for this, kubernetes requires a special networking solution between master and worker nodes which is known as pod network.

Finally we join the worker nodes to the master.

## Installation practice

Install the kubeadm and kubelet packages on the controlplane and node01.

Use the exact version of 1.21.0-00 for both.
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
br_netfilter
```
```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
> net.bridge.bridge-nf-call-ip6tables = 1
> net.bridge.bridge-nf-call-iptables = 1
> EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
```
```bash
sudo sysctl --system
* Applying /etc/sysctl.d/10-console-messages.conf ...
kernel.printk = 4 4 1 7
* Applying /etc/sysctl.d/10-ipv6-privacy.conf ...
net.ipv6.conf.all.use_tempaddr = 2
net.ipv6.conf.default.use_tempaddr = 2
* Applying /etc/sysctl.d/10-kernel-hardening.conf ...
kernel.kptr_restrict = 1
* Applying /etc/sysctl.d/10-link-restrictions.conf ...
fs.protected_hardlinks = 1
fs.protected_symlinks = 1
* Applying /etc/sysctl.d/10-magic-sysrq.conf ...
kernel.sysrq = 176
* Applying /etc/sysctl.d/10-network-security.conf ...
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.tcp_syncookies = 1
* Applying /etc/sysctl.d/10-ptrace.conf ...
kernel.yama.ptrace_scope = 1
* Applying /etc/sysctl.d/10-zeropage.conf ...
vm.mmap_min_addr = 65536
* Applying /usr/lib/sysctl.d/50-default.conf ...
net.ipv4.conf.all.promote_secondaries = 1
net.core.default_qdisc = fq_codel
* Applying /etc/sysctl.d/99-sysctl.conf ...
* Applying /etc/sysctl.d/k8s.conf ...
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
* Applying /etc/sysctl.conf ...
```
```bash
sudo apt-get update
Get:2 https://download.docker.com/linux/ubuntu bionic InRelease [64.4 kB]                                            
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial InRelease [9383 B]                                     
Get:3 http://archive.ubuntu.com/ubuntu bionic InRelease [242 kB]                                                     
Get:4 https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages [27.9 kB]                             
Get:5 http://security.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]                                         
Get:6 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 Packages [54.7 kB]   
Get:7 http://archive.ubuntu.com/ubuntu bionic-updates InRelease [88.7 kB]                             
Get:8 http://security.ubuntu.com/ubuntu bionic-security/restricted amd64 Packages [884 kB]
Get:9 http://archive.ubuntu.com/ubuntu bionic-backports InRelease [74.6 kB]     
Get:10 http://archive.ubuntu.com/ubuntu bionic/universe amd64 Packages [11.3 MB]          
Get:11 http://security.ubuntu.com/ubuntu bionic-security/universe amd64 Packages [1490 kB]  
Get:12 http://security.ubuntu.com/ubuntu bionic-security/main amd64 Packages [2694 kB]       
Get:13 http://security.ubuntu.com/ubuntu bionic-security/multiverse amd64 Packages [21.1 kB]  
Get:14 http://archive.ubuntu.com/ubuntu bionic/main amd64 Packages [1344 kB]                    
Get:15 http://archive.ubuntu.com/ubuntu bionic/restricted amd64 Packages [13.5 kB]
Get:16 http://archive.ubuntu.com/ubuntu bionic/multiverse amd64 Packages [186 kB]
Get:17 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 Packages [3133 kB]
Get:18 http://archive.ubuntu.com/ubuntu bionic-updates/multiverse amd64 Packages [29.8 kB]
Get:19 http://archive.ubuntu.com/ubuntu bionic-updates/universe amd64 Packages [2268 kB]
Get:20 http://archive.ubuntu.com/ubuntu bionic-updates/restricted amd64 Packages [918 kB]
Get:21 http://archive.ubuntu.com/ubuntu bionic-backports/main amd64 Packages [12.2 kB]
Get:22 http://archive.ubuntu.com/ubuntu bionic-backports/universe amd64 Packages [12.9 kB]
Fetched 25.0 MB in 4s (6747 kB/s)                             
Reading package lists... Done
```

```bash
sudo apt-get install -y apt-transport-https ca-certificates curl
```
```bash
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```
```bahs
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
```bash
sudo apt-get update
sudo apt-get install -y kubelet=1.21.0-00 kubeadm=1.21.0-00 kubectl=1.21.0-00
sudo apt-mark hold kubelet kubeadm kubectl
```


Initialize Control Plane Node (Master Node). Use the following options:

```bash
apiserver-advertise-address - Use the IP address allocated to eth0 on the controlplane node

apiserver-cert-extra-sans - Set it to controlplane

pod-network-cidr - Set to 10.244.0.0/16
```

Once done, set up the default kubeconfig file and wait for node to be part of the cluster.
```bash
ifconfig eth0
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
        inet 10.54.102.3  netmask 255.255.255.0  broadcast 10.54.102.255
        ether 02:42:0a:36:66:03  txqueuelen 0  (Ethernet)
        RX packets 5421  bytes 677616 (677.6 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 4681  bytes 1698249 (1.6 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
```bash
kubeadm init --apiserver-cert-extra-sans=controlplane --apiserver-advertise-address 10.54.102.3 --pod-network-cidr=10.244.0.0/16
```
```bash
mkdir -p $HOME/.kube
root@controlplane:~#   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
root@controlplane:~#   sudo chown $(id -u):$(id -g) $HOME/.kube/config
root@controlplane:~# 
```
```bash
kubeadm join 10.54.102.3:6443 --token m46tv3.z8fzuu48forndsva \
        --discovery-token-ca-cert-hash sha256:3b7fbad4079c30e46d4c8ed119bd1f852dba2b167e29176fd6258af62ab5625f 
```


Join node01 to the cluster using the join token
```bash
kubeadm token create --print-join-commandkubeadm join 10.54.102.3:6443 --token 7cwc5b.08lxunts7oqnmdw8 --discovery-token-ca-cert-hash sha256:3b7fbad4079c30e46d4c8ed119bd1f852dba2b167e29176fd6258af62ab5625f 
```
```bash
ssh node01
```
Install a Network Plugin. As a default, we will go with flannel

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
Warning: policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
podsecuritypolicy.policy/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
```