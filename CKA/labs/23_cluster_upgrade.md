# Cluster Upgrade Lab

What is the current version of the cluster?
```bash
kubectl get nodes
NAME           STATUS   ROLES    AGE   VERSION
controlplane   Ready    master   36m   v1.19.0
node01         Ready    <none>   36m   v1.19.0
```
How many nodes can host workloads in this cluster?



>Inspect the applications and taints set on the nodes.
```bash
kubectl describe node node01 | grep Taints
Taints:             <none>
root@controlplane:~# kubectl describe node controlplane | grep Taints
Taints:             <none>
```
```bash
kubectl get deployments --no-headers | wc -l
1
```

What nodes are the pods hosted on?
```bash
kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
blue-746c87566d-5rq69   1/1     Running   0          7m24s   10.244.1.4   node01         <none>           <none>
blue-746c87566d-7vl98   1/1     Running   0          7m24s   10.244.0.5   controlplane   <none>           <none>
blue-746c87566d-lg8d5   1/1     Running   0          7m24s   10.244.1.3   node01         <none>           <none>
blue-746c87566d-n2vhm   1/1     Running   0          7m24s   10.244.1.2   node01         <none>           <none>
blue-746c87566d-v6gzl   1/1     Running   0          7m24s   10.244.0.4   controlplane   <none>           <none>
```
What is the latest stable version available for upgrade?



> Use the kubeadm tool
```bash
kubeadm upgrade plan
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade] Fetching available versions to upgrade to
[upgrade/versions] Cluster version: v1.19.0
[upgrade/versions] kubeadm version: v1.19.0
I0328 15:22:10.524539   28995 version.go:252] remote version is much newer: v1.23.5; falling back to: stable-1.19
[upgrade/versions] Latest stable version: v1.19.16
[upgrade/versions] Latest stable version: v1.19.16
[upgrade/versions] Latest version in the v1.19 series: v1.19.16
[upgrade/versions] Latest version in the v1.19 series: v1.19.16

Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT       AVAILABLE
kubelet     2 x v1.19.0   v1.19.16

Upgrade to the latest version in the v1.19 series:

COMPONENT                 CURRENT   AVAILABLE
kube-apiserver            v1.19.0   v1.19.16
kube-controller-manager   v1.19.0   v1.19.16
kube-scheduler            v1.19.0   v1.19.16
kube-proxy                v1.19.0   v1.19.16
CoreDNS                   1.7.0     1.7.0
etcd                      3.4.9-1   3.4.9-1

You can now apply the upgrade by executing the following command:

        kubeadm upgrade apply v1.19.16

Note: Before you can perform this upgrade, you have to update kubeadm to v1.19.16.

_____________________________________________________________________

The table below shows the current state of component configs as understood by this version of kubeadm.
Configs that have a "yes" mark in the "MANUAL UPGRADE REQUIRED" column require manual config upgrade or
resetting to kubeadm defaults before a successful upgrade can be performed. The version to manually
upgrade to is denoted in the "PREFERRED VERSION" column.

API GROUP                 CURRENT VERSION   PREFERRED VERSION   MANUAL UPGRADE REQUIRED
kubeproxy.config.k8s.io   v1alpha1          v1alpha1            no
kubelet.config.k8s.io     v1beta1           v1beta1             no
_____________________________________________________________________
```


We will be upgrading the master node first. Drain the master node of workloads and mark it UnSchedulable
```bash
kubectl drain controlplane --ignore-daemonsets
node/controlplane cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/kube-flannel-ds-wbx58, kube-system/kube-proxy-q4r76
evicting pod default/blue-746c87566d-7vl98
evicting pod default/blue-746c87566d-v6gzl
evicting pod kube-system/coredns-f9fd979d6-ht5s7
evicting pod kube-system/coredns-f9fd979d6-q68dv
pod/blue-746c87566d-7vl98 evicted
pod/blue-746c87566d-v6gzl evicted
pod/coredns-f9fd979d6-ht5s7 evicted
pod/coredns-f9fd979d6-q68dv evicted
node/controlplane evicted
```
```bash
kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP            NODE     NOMINATED NODE   READINESS GATES
blue-746c87566d-5rq69   1/1     Running   0          12m   10.244.1.4    node01   <none>           <none>
blue-746c87566d-kmlnj   1/1     Running   0          42s   10.244.1.12   node01   <none>           <none>
blue-746c87566d-lg8d5   1/1     Running   0          12m   10.244.1.3    node01   <none>           <none>
blue-746c87566d-n2vhm   1/1     Running   0          12m   10.244.1.2    node01   <none>           <none>
blue-746c87566d-z82vz   1/1     Running   0          42s   10.244.1.9    node01   <none>           <none>
```
Upgrade the controlplane components to exact version v1.20.0



>Upgrade kubeadm tool (if not already), then the master components, and finally the kubelet. Practice referring to the kubernetes documentation page. Note: While upgrading kubelet, if you hit dependency issue while running the apt-get upgrade kubelet command, use the apt install kubelet=1.20.0-00 command instead
```bash
apt update
Get:2 https://download.docker.com/linux/ubuntu bionic InRelease [64.4 kB]                                  
Get:3 http://archive.ubuntu.com/ubuntu bionic InRelease [242 kB]                                                                                  
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial InRelease [9383 B]                         
Get:4 http://security.ubuntu.com/ubuntu bionic-security InRelease [88.7 kB]                                         
Get:5 http://archive.ubuntu.com/ubuntu bionic-updates InRelease [88.7 kB]                       
Get:6 https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages [27.9 kB]           
Get:7 http://archive.ubuntu.com/ubuntu bionic-backports InRelease [74.6 kB]         
Get:8 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 Packages [54.7 kB]                            
Get:9 http://archive.ubuntu.com/ubuntu bionic/main amd64 Packages [1344 kB]
Get:10 http://archive.ubuntu.com/ubuntu bionic/multiverse amd64 Packages [186 kB]
Get:11 http://archive.ubuntu.com/ubuntu bionic/universe amd64 Packages [11.3 MB]     
Get:12 http://archive.ubuntu.com/ubuntu bionic/restricted amd64 Packages [13.5 kB]     
Get:13 http://archive.ubuntu.com/ubuntu bionic-updates/universe amd64 Packages [2262 kB]
Get:14 http://archive.ubuntu.com/ubuntu bionic-updates/restricted amd64 Packages [893 kB]
Get:15 http://archive.ubuntu.com/ubuntu bionic-updates/main amd64 Packages [3098 kB] 
Get:16 http://archive.ubuntu.com/ubuntu bionic-updates/multiverse amd64 Packages [29.8 kB]
Get:17 http://archive.ubuntu.com/ubuntu bionic-backports/main amd64 Packages [11.6 kB]  
Get:18 http://archive.ubuntu.com/ubuntu bionic-backports/universe amd64 Packages [12.6 kB]
Get:19 http://security.ubuntu.com/ubuntu bionic-security/multiverse amd64 Packages [21.1 kB]
Get:20 http://security.ubuntu.com/ubuntu bionic-security/restricted amd64 Packages [860 kB]
Get:21 http://security.ubuntu.com/ubuntu bionic-security/main amd64 Packages [2660 kB]
Get:22 http://security.ubuntu.com/ubuntu bionic-security/universe amd64 Packages [1484 kB]
Fetched 24.9 MB in 5s (4649 kB/s)                           
Reading package lists... Done
Building dependency tree       
Reading state information... Done
65 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

```bash
apt install kubeadm=1.20.0-00
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following held packages will be changed:
  kubeadm
The following packages will be upgraded:
  kubeadm
1 upgraded, 0 newly installed, 0 to remove and 64 not upgraded.
Need to get 7707 kB of archives.
After this operation, 111 kB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubeadm amd64 1.20.0-00 [7707 kB]
Fetched 7707 kB in 0s (24.7 MB/s)
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 15149 files and directories currently installed.)
Preparing to unpack .../kubeadm_1.20.0-00_amd64.deb ...
Unpacking kubeadm (1.20.0-00) over (1.19.0-00) ...
Setting up kubeadm (1.20.0-00) ...
```

```bash
kubeadm upgrade apply v1.20.0
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade/version] You have chosen to change the cluster version to "v1.20.0"
[upgrade/versions] Cluster version: v1.19.0
[upgrade/versions] kubeadm version: v1.20.0
[upgrade/confirm] Are you sure you want to proceed with the upgrade? [y/N]: y
[upgrade/prepull] Pulling images required for setting up a Kubernetes cluster
[upgrade/prepull] This might take a minute or two, depending on the speed of your internet connection
[upgrade/prepull] You can also perform this action in beforehand using 'kubeadm config images pull'
[upgrade/apply] Upgrading your Static Pod-hosted control plane to version "v1.20.0"...
Static pod: kube-apiserver-controlplane hash: 8d3ca6433d9f078cfb8e5666a31642c6
Static pod: kube-controller-manager-controlplane hash: f6a9bf2865b2fe580f39f07ed872106b
Static pod: kube-scheduler-controlplane hash: 5146743ebb284c11f03dc85146799d8b
[upgrade/etcd] Upgrading to TLS for etcd
Static pod: etcd-controlplane hash: b3f005df09f8b9f44b6022ad88ec3208
[upgrade/staticpods] Preparing for "etcd" upgrade
[upgrade/staticpods] Renewing etcd-server certificate
[upgrade/staticpods] Renewing etcd-peer certificate
[upgrade/staticpods] Renewing etcd-healthcheck-client certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/etcd.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2022-03-28-15-50-43/etcd.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
Static pod: etcd-controlplane hash: b3f005df09f8b9f44b6022ad88ec3208
...
```


```bash
kubectl version --short
Client Version: v1.19.0
Server Version: v1.20.0
```

```bash
apt install kubelet=1.20.0-00
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following held packages will be changed:
  kubelet
The following packages will be upgraded:
  kubelet
1 upgraded, 0 newly installed, 0 to remove and 64 not upgraded.
Need to get 18.8 MB of archives.
After this operation, 4000 kB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 https://packages.cloud.google.com/apt kubernetes-xenial/main amd64 kubelet amd64 1.20.0-00 [18.8 MB]
Fetched 18.8 MB in 0s (60.0 MB/s)
debconf: delaying package configuration, since apt-utils is not installed
(Reading database ... 15149 files and directories currently installed.)
Preparing to unpack .../kubelet_1.20.0-00_amd64.deb ...
/usr/sbin/policy-rc.d returned 101, not running 'stop kubelet.service'
Unpacking kubelet (1.20.0-00) over (1.19.0-00) ...
Setting up kubelet (1.20.0-00) ...
/usr/sbin/policy-rc.d returned 101, not running 'start kubelet.service'
```
```bash
systemctl restart kubelet
```
```bash
kubectl get nodes
NAME           STATUS                        ROLES                  AGE   VERSION
controlplane   NotReady,SchedulingDisabled   control-plane,master   79m   v1.20.0
node01         Ready                         <none>                 79m   v1.19.0
```
Mark the controlplane node as "Schedulable" again
```bash
kubectl uncordon controlplane
node/controlplane uncordoned
```
Next is the worker node. Drain the worker node of the workloads and mark it UnSchedulable
```bash
kubectl drain node01 --ignore-daemonsets
node/node01 cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/kube-flannel-ds-pjcrp, kube-system/kube-proxy-z7m6z
evicting pod default/blue-746c87566d-lg8d5
evicting pod default/blue-746c87566d-n2vhm
evicting pod default/blue-746c87566d-kmlnj
evicting pod kube-system/coredns-74ff55c5b-4jthn
evicting pod default/blue-746c87566d-5rq69
evicting pod kube-system/coredns-74ff55c5b-nx8rf
evicting pod default/blue-746c87566d-z82vz
I0328 15:58:13.795582   15777 request.go:645] Throttling request took 1.027308466s, request: GET:https://controlplane:6443/api/v1/namespaces/default/pods/blue-746c87566d-kmlnj
I0328 15:58:24.065070   15777 request.go:645] Throttling request took 1.295908354s, request: GET:https://controlplane:6443/api/v1/namespaces/kube-system/pods/coredns-74ff55c5b-nx8rf
pod/blue-746c87566d-z82vz evicted
pod/blue-746c87566d-lg8d5 evicted
pod/blue-746c87566d-n2vhm evicted
pod/coredns-74ff55c5b-4jthn evicted
pod/coredns-74ff55c5b-nx8rf evicted
pod/blue-746c87566d-5rq69 evicted
pod/blue-746c87566d-kmlnj evicted
node/node01 evicted
```


On the node01 node, run the command run the following commands:

>If you are on the master node, run `ssh node01` to go to node01

```bash
apt update
```
This will update the package lists from the software repository.
```bash
apt install kubeadm=1.20.0-00
```
This will install the kubeadm version 1.20

```bash
kubeadm upgrade node
```
This will upgrade the node01 configuration.

```bash
apt install kubelet=1.20.0-00
```
This will update the kubelet with the version 1.20.


You may need to restart kubelet after it has been upgraded.
Run:
```bash
systemctl restart kubelet
```

Type exit or enter CTL + d to go back to the controlplane node.
