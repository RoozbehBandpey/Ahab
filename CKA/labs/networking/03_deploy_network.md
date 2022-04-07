# Deploy Networking Solutions Lab



Deploy weave-net networking solution to the cluster.



>Replace the default IP address and subnet of weave-net to the 10.50.0.0/16. Please check the official weave [installation](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-installation) and [configuration](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-changing-configuration-options) guide.


By default, the range of IP addresses and the subnet used by weave-net is 10.32.0.0/12 and it's overlapping with the host system IP addresses.
To know the host system IP address by running `ip a` command :

ip a | grep eth0
3303: eth0@if3304: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default 
    inet 10.16.109.6/24 brd 10.16.109.255 scope global eth0

>If we deploy a weave manifest file directly without changing the default IP addresses it will overlap with the host system IP addresses and as a result, it's weave pods will go into an Error or CrashLoopBackOff state.

```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```
```bash
root@controlplane:~# kubectl get po -n kube-system | grep weave
weave-net-6mckb                        1/2     CrashLoopBackOff   6          6m46s
```
If we will go more deeper and inspect the logs then we can clearly see the issue:
```bash
root@controlplane:~# kubectl logs -n kube-system weave-net-6mckb -c weave
Network 10.32.0.0/12 overlaps with existing route 10.40.56.0/24 on host
```
So we need to change the default IP address by adding `&env.IPALLOC_RANGE=10.50.0.0/16` option at the end of the manifest file. It should be look like as follows:
```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')&env.IPALLOC_RANGE=10.50.0.0/16"
```
then run the `kubectl get pods -n kube-system` to see the status of weave-net pods.