# Cluster Maintenance


## OS Upgrades
Let's take a look at options available to handle cases that we might take a node down, such as patches. 

Imagine we have a cluster with few nodes and pods serving an application, what happens when one of the nodes go down. Of course the pods on them are not accessible! Depending how we use those pods our users might be impacted. For instance if we have multiple replicas of a pod across multiple nodes the users of those pods are not impacted, if we have pod only up on that node, the users of that pod are impacted. 

If the node comes back online immediately, the kubelet comes back and the pods will be accessible again. However if the node was down for more than 5 minutes the the pods will be terminated on that node (kubernetes considers them as dead). If the pods are part of a replicaset, then they are recreated on other nodes. 

The time that kubernetes wait for pods to come back online is know as `pod eviction timeout` and it's set on the controller manager with the default value of 5 minuets `kube-controller-manager --pod-eviction-timeout=5m0s`. So whenever a node goes offline, the master node waits for up to 5 minutes before considering the node dead. 

When the node comes back online after the node eviction timeout it comes back blank, without any pods scheduled on it. If the pods that they were on the down node were not part of any replicasets they are just gone! 

We can make a quick upgrade and reboot if we're sure the node will come back online before eviction timeout and our pods are part of a replicaset. But the problem is we never know if the node is going to be back online within eviction timeout. 

The safer way to do this is to purposefully drain the node from all the workloads `kubectl drain node-1` the workload will be moved to other nodes on the cluster. When we drain a node, the pods on that node are gracefully terminated and they are recreated on other nodes. Then the node will be automatically marked as unschedulable. No pods can be scheduled on that node until we specifically removed that restriction. 

Now we can safely reboot the node, when it comes back online it is still unschedulable. We have to `uncordon` it so the node can ba schedulable. 

```bash
kubectl uncordon node-1
```
The pods that moved to the other nodes, do not automatically fall back!

We can also manually mark a node as unschedulable with `cordon` command!
```bash
kubectl cordon node-1
```



## Kubernetes Software Versions

When we install kubernetes cluster, we install a specific version of kubernetes, we will see the versions when we run `kubectl get nodes` command. 

The kubernetes release versions consist of three parts, vMAJOR.MINOR.PATCH, while minor versions are released every few month with new features and functionalities. Patches are more frequent which include bug fixes. 

Kubernetes follows a standard software release approach, The first major release v1.0 was released on July 2015. Apart from stable releases we will see alpha nad beta releases, all the bugfixes and improvements first goes to alpha release vX.Y.X-alpha, in this release the features are disabled and buggy. And from there they make their way to beta release v.X.Y.Z-beta were it becomes tested and new features become enabled by default and finally they make their way to main stable release.

When you download the kubernetes binary it comes with some control-plane components such as
* kube-apiserver 
* controller-manager
* kube-scheduler
* kubelet
* kube-proxy
* kubectl

All of these have same version, except:
* ETCD Cluster
* CoreDNS

As they are separate projects
## Cluster Upgrade Process

Since `kube-apiserver` is the primary component that all other components talk to, none of the other components can be at a version higher than `kube-apiserver`. 

The `controller-manager` and `kube-scheduler` can be at one version lower

The `kubelet` and `kube-proxy` can be at two version lower.

The `kubectl` can be at one version higher or lower than, `kube-apiserver`

This allows us to upgrade component by components. 

At any time kubernetes supports only below three minor versions, for example if we are at `v1.10` and kubernetes releases `v1.11` and `v1.12`, so our cluster is still fully supported. But when `v1.13` is released our cluster is not supported anymore. Best time to upgrade cluster to the next release is before the release of `v1.13`.

The recommended approach is to upgrade one minor version at a time, meaning no jump from `v1.10` to `v1.13` rather one by one. 

Kubeadm can help us to plan and apply a cluster upgrade:
```bash
kubeadm upgrade plan
```
```bash
kubeadm upgrade apply
```
If we deploy the cluster from scratch then we're suppose to handle upgrades manually, ourselves. 

### Kubeadm
Say we have a cluster on production with multiple nodes, the nodes and components are at `v1.10` upgrading a cluster involves two major steps, 
1. Upgrade the master nodes
    * While master is being upgraded the control-plane components such as `kube-apiserver`, `scheduler` etc., goes down briefly 
    * This does not mean that worker nodes and applications on them are impacted
    * Meanwhile all management functions are down, we cannot access the cluster using `kubectl`, we cannot deploy new applications or delete or modify existing ones. If a pod fails an new pod won't be recreated automatically
    * Once upgrade is complete everything should function normally

2. Then upgrade the worker nodes: There are different strategies to upgrade worker nodes
    1. Upgrade all at once
        * Pods are down and users are no longer able to access applications
        * Once upgrade is complete the nodes are back up, pods are scheduled and users can resume access
    2. Upgrade one node at a time
        * We first upgrade the first node 
        * Move the workload to the other nodes 
        * Once upgraded and back up, we bring back its workload
        * Then we update the second node and so on
    3. Add new nodes to the cluster: Nodes which already have newer kubernetes version
        * Create a node with newer version of kubernetes
        * Move the workload to the new one
        * Remove the old node

> Kubeadm does not install and upgrade the kubelets

The process of upgrading from `v1.11` to `v1.13` is as follows:

First upgrade the kubeadm tool itself
```bash
apt-get upgrade -y kubeadm=1.12.0-00
```
The using the output of plan, upgrade the cluster:
```bash
kubeadm upgrade apply v1.12.0
```
If we run `kubectl get nodes` we will still see master node at `v1.11` This is because this shows the versions of kubelet on each node.

To upgrade kubelet:

```bash
apt-get upgrade -y kubelet=1.12.0-00
```
Once it's upgraded, restart the kubelet service:
```bash
systemctl restart kubelet
```


Now it's time to upgrade worker nodes:

One at a time:

First move the workload from one worker node to others:
```bash
kubectl drain node-1
```

Then upgrade the kubeadm and kubelet on the worker node
```bash
apt-get upgrade -y kubeadm=1.12.0-00
```
```bash
apt-get upgrade -y kubelet=1.12.0-00
```
Update the node configuration for the new kubelet version
```bash
kubeadm upgrade node config --kubelet-version v1.12.0
```
Then restart the kubelet service
```bash
systemctl restart kubelet
```
Now we have to uncordon the node so new pods can be scheduled on it.

```bash
kubectl uncordon node-1
```

The node is now schedulable but not necessarily pods will come back on the node, only when those pods are deleted form the other pods or the new pods are scheduled they will come back on this node. When we take down the second node the workload will be back here.

## Backup and Restore Methods

## Working with ETCDCTL
