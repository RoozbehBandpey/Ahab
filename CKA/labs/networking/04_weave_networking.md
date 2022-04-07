# Weave Networking Lab

What is the Networking Solution used by this cluster?
```bash
ls /etc/cni/net.d/
10-weave.conflist
root@controlplane:~# cat /etc/cni/net.d/10-weave.conflist
{
    "cniVersion": "0.3.0",
    "name": "weave",
    "plugins": [
        {
            "name": "weave",
            "type": "weave-net",
            "hairpinMode": true
        },
        {
            "type": "portmap",
            "capabilities": {"portMappings": true},
            "snat": true
        }
    ]
}
```
How many weave agents/peers are deployed in this cluster?

```bash
kubectl get pods -n kube-system | grep weave
weave-net-9hvhp                        2/2     Running   1          17m
weave-net-thxq7                        2/2     Running   0          17m
```

On which nodes are the weave peers present?
```bash
kubectl get pods -n kube-system -o wide | grep weave
weave-net-9hvhp                        2/2     Running   1          18m   10.19.148.9    controlplane   <none>           <none>
weave-net-thxq7                        2/2     Running   0          17m   10.19.148.12   node01         <none>           <none>
```

Identify the name of the bridge network/interface created by weave on each node
```bash
ip a
```
What is the POD IP address range configured by weave?
```bash
ip addr show weave
5: weave: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1376 qdisc noqueue state UP group default qlen 1000
    link/ether d2:39:b6:64:17:c5 brd ff:ff:ff:ff:ff:ff
    inet 10.50.0.2/16 brd 10.50.255.255 scope global weave
       valid_lft forever preferred_lft forever
```

What is the default gateway configured on the PODs scheduled on node01?



Try scheduling a pod on `node01` and check ip route output

```bash
kubectl run busybox --image=busybox --command sleep 1000 --dry-run=client -o yaml > pod.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: busybox
  name: busybox
spec:
  nodeNmae: node01
  containers:
  - command:
    - sleep
    - "1000"
    image: busybox
    name: busybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
After apply
```bash
kubectl exec -ti busybox -- sh
```
```bash
ip r
default via 10.50.192.0 dev eth0 
10.50.0.0/16 dev eth0 scope link  src 10.50.192.1 
```