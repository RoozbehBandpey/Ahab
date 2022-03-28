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



## Cluster Upgrade Process



## Backup and Restore Methods

## Working with ETCDCTL
