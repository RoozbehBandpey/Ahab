# Taints and Tolarations Lab

How many nodes exist on the system?

Including the controlplane node.
```bash
kubectl get node --no-headers | wc -l
```
Do any taints exist on node01 node?
```bash
kubectl describe node node01 | grep Taints
Taints:             <none>
```

Create a taint on node01 with key of spray, value of mortein and effect of NoSchedule

```bash
kubectl taint nodes node01 spray=mortein:NoSchedule

node/node01 tainted
```
Create a new pod with the nginx image and pod name as mosquito.

```bash
kubectl run mosquito --image=nginx

kubectl  get pods

NAME       READY   STATUS    RESTARTS   AGE
mosquito   0/1     Pending   0          24s
```
Why do you think the pod is in a pending state?

```bash
kubectl describe pods mosquito
Name:         mosquito
Namespace:    default
Priority:     0
Node:         <none>
Labels:       run=mosquito
Annotations:  <none>
Status:       Pending
IP:           
IPs:          <none>
Containers:
  mosquito:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-jdh4q (ro)
Conditions:
  Type           Status
  PodScheduled   False 
Volumes:
  default-token-jdh4q:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-jdh4q
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age                From               Message
  ----     ------            ----               ----               -------
  Warning  FailedScheduling  66s (x2 over 66s)  default-scheduler  0/2 nodes are available: 1 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate, 1 node(s) had taint {spray: mortein}, that the pod didn't tolerate.
```

Create another pod named bee with the nginx image, which has a toleration set to the taint mortein.

```bash
kubectl run bee --image=nginx --dry-run=client -o yaml > bee.yaml

vi bee.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: bee
  name: bee
spec:
  containers:
  - image: nginx
    name: bee
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  tolerations:
  - key: "spray"
    operator: "Equal"
    value: "mortein"
    effect: "NoSchedule"
status: {}
```
Notice the bee pod was scheduled on node node01 despite the taint.


Do you see any taints on controlplane node?

```bash
kubectl describe node controlplane | grep Taints
Taints:             node-role.kubernetes.io/master:NoSchedule
```

Remove the taint on controlplane, which currently has the taint effect of NoSchedule.

```bash
kubectl taint nodes controlplane node-role.kubernetes.io/master:NoSchedule-
```
Which node is the POD mosquito on now?
```bash
kubectl get pods -o wide
NAME       READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
bee        1/1     Running   0          4m17s   10.244.1.2   node01         <none>           <none>
mosquito   1/1     Running   0          11m     10.244.0.4   controlplane   <none>           <none>
```