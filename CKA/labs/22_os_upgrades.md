# OS Upgrades Lab

How many nodes do you see in the cluster?

> Including the controlplane and worker nodes.
```bash
kubectl get nodes
NAME           STATUS   ROLES                  AGE     VERSION
controlplane   Ready    control-plane,master   4m23s   v1.20.0
node01         Ready    <none>                 3m31s   v1.20.0
```
How many applications do you see hosted on the cluster?

> Check the number of deployments.
```bash
kubectl get deployments --all-namespaces
NAMESPACE     NAME      READY   UP-TO-DATE   AVAILABLE   AGE
default       blue      3/3     3            3           44s
kube-system   coredns   2/2     2            2           5m34s
```
Which nodes are the applications hosted on?
```bash
kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE     NOMINATED NODE   READINESS GATES
blue-746c87566d-bkl4q   1/1     Running   0          3m21s   10.244.1.3   node01   <none>           <none>
blue-746c87566d-dl2tx   1/1     Running   0          3m21s   10.244.1.4   node01   <none>           <none>
blue-746c87566d-v2mh5   1/1     Running   0          3m21s   10.244.1.2   node01   <none>           <none>
```
We need to take node01 out for maintenance. Empty the node of all applications and mark it unschedulable.

```bash
kubectl drain node01
node/node01 cordoned
error: unable to drain node "node01", aborting command...
There are pending nodes to be drained:
 node01
error: cannot delete DaemonSet-managed Pods (use --ignore-daemonsets to ignore): kube-system/kube-flannel-ds-zbjqf, kube-system/kube-proxy-fw7tp
```

```bash
kubectl drain node01 --ignore-daemonsets
node/node01 already cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/kube-flannel-ds-zbjqf, kube-system/kube-proxy-fw7tp
evicting pod default/blue-746c87566d-v2mh5
evicting pod default/blue-746c87566d-bkl4q
evicting pod default/blue-746c87566d-dl2tx
pod/blue-746c87566d-v2mh5 evicted
pod/blue-746c87566d-bkl4q evicted
pod/blue-746c87566d-dl2tx evicted
node/node01 evicted
root@controlplane:~# kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
blue-746c87566d-7hv9z   1/1     Running   0          18s   10.244.0.4   controlplane   <none>           <none>
blue-746c87566d-cg4j9   1/1     Running   0          17s   10.244.0.6   controlplane   <none>           <none>
blue-746c87566d-gdrpx   1/1     Running   0          18s   10.244.0.5   controlplane   <none>           <none>
```
The maintenance tasks have been completed. Configure the node node01 to be schedulable again.

```bash
kubectl get nodes
NAME           STATUS                     ROLES                  AGE   VERSION
controlplane   Ready                      control-plane,master   11m   v1.20.0
node01         Ready,SchedulingDisabled   <none>                 10m   v1.20.0
root@controlplane:~# kubectl uncordon node01
node/node01 uncordoned
root@controlplane:~# kubectl get nodes
NAME           STATUS   ROLES                  AGE   VERSION
controlplane   Ready    control-plane,master   11m   v1.20.0
node01         Ready    <none>                 11m   v1.20.0
```
If there is a pod in node01 which is not part of a replicaset the drain will through an error
```bash
kubectl drain node01 --ignore-daemonsets
node/node01 cordoned
error: unable to drain node "node01", aborting command...

There are pending nodes to be drained:
 node01
error: cannot delete Pods not managed by ReplicationController, ReplicaSet, Job, DaemonSet or StatefulSet (use --force to override): default/hr-app
```
What would happen to hr-app if node01 is drained forcefully?

> hr-app will be lost forever