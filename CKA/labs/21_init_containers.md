# Init Containers Lab

Identify the pod that has an initContainer configured.

```bash
kubectl describe pod blue | grep 'Init Containers'
```
Or
```bash
kubectl get pod blue
```
Under status we should be able to see Init:0/1


Update the pod `red` to use an `initContainer` that uses the busybox image and sleeps for 20 seconds.

```bash
kubectl get pod red -o yaml > red.yaml
```

```bash
kubectl delete pod red
```

```yaml
spec:
  initContainers:
  - image: busybox
    name: red-initcontainer
    command: ["sleep", "20"]
```