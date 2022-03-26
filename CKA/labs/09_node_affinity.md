# Node Affinity Lab

How many Labels exist on node node01?
```bash
kubectl get nodes node01 --show-labels
NAME     STATUS   ROLES    AGE   VERSION   LABELS
node01   Ready    <none>   14m   v1.20.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=node01,kubernetes.io/os=linux
```


Apply a label color=blue to node node01
```bash
kubectl label node node01 color=blue
```

Create a new deployment named blue with the nginx image and 3 replicas.
```bash
kubectl create deployment blue --image=nginx --replicas=3
deployment.apps/blue created
```
Which nodes can the pods for the blue deployment be placed on?

Make sure to check taints on both nodes!
```bahs
kubectl describe nodes node01 | grep Taints
Taints:             <none>

kubectl describe nodes controlplane | grep Taints
Taints:             <none>
```
Answer : both nodes


Set Node Affinity to the deployment to place the pods on node01 only.



Add affinity parallel  to containers
```yaml
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDurinngExecution:
             nodeSelectorTerms:
               - matchExpressions:
                 - key: color
                   operator: In
                   values:
                     - blue
```
```bash
kubectl get pods -o wide
```


Create a new deployment named red with the nginx image and 2 replicas, and ensure it gets placed on the controlplane node only.



Use the label - node-role.kubernetes.io/master - set on the controlplane node.
```bash
kubectl get nodes controlplane --show-labels
NAME           STATUS   ROLES                  AGE   VERSION   LABELS
controlplane   Ready    control-plane,master   38m   v1.20.0   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=controlplane,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node-role.kubernetes.io/master=
```
```yaml
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/master
                operator: Exists
```

```bash
kubectl get pods -o wide
NAME                    READY   STATUS              RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
blue-566c768bd6-l8k2q   1/1     Running             0          9m15s   10.244.1.5   node01         <none>           <none>
blue-566c768bd6-m6bb7   1/1     Running             0          9m11s   10.244.1.6   node01         <none>           <none>
blue-566c768bd6-tbk9h   1/1     Running             0          9m7s    10.244.1.7   node01         <none>           <none>
red-5cbd45ccb6-dwsc5    0/1     ContainerCreating   0          0s      <none>       controlplane   <none>           <none>
red-5cbd45ccb6-zc6xk    1/1     Running             0          15s     10.244.0.4   controlplane   <none>           <none>
red-7748479b97-2wqpz    1/1     Terminating         0          6m36s   10.244.1.9   node01         <none>           <none>
red-7748479b97-6mfqs    1/1     Running             0          6m36s   10.244.1.8   node01         <none>           <none>
```

