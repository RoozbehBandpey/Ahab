# Manual Scheduling

Why is the POD in a pending state?



Inspect the environment for various kubernetes control plane components.
```bash
kubectl get pods -n kube-system
```
Manually schedule the pod on node01.



Delete and recreate the POD if necessary.
```bash
kubectl delete pods nginx
```
```bash
vi nginx.yaml
```
```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  -  image: nginx
     name: nginx
  nodeName: node01
```