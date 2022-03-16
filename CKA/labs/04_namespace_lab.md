# Namespace practice test

How many namespaces exists:

```bash
kubectl get namespaces

# Output
NAME              STATUS   AGE
default           Active   7m8s
kube-system       Active   7m8s
kube-public       Active   7m8s
kube-node-lease   Active   7m8s
finance           Active   34s
marketing         Active   34s
dev               Active   34s
prod              Active   33s
manufacturing     Active   33s
research          Active   33s
```

```bash
kubectl get ns --no-headers | wc -l
```

How many pods exist in the research namespace?

```bash
kubectl get pods -n research --no-headers
NAME    READY   STATUS             RESTARTS      AGE
dna-2   0/1     CrashLoopBackOff   3 (46s ago)   113s
dna-1   0/1     CrashLoopBackOff   3 (45s ago)   113s
```

Create a POD in the finance namespace.

Use the spec given below.

Name: redis
Image Name: redis
```bash
kubectl run redis --image=redis -n finance
```

Which namespace has the blue pod in it?
```bash
kubectl get pods --all-namespaces
NAMESPACE       NAME                                     READY   STATUS             RESTARTS      AGE
kube-system     helm-install-traefik-crd--1-nfz5w        0/1     Completed          0             13m
kube-system     helm-install-traefik--1-8kzfs            0/1     Completed          1             13m
kube-system     traefik-74dd4975f9-nw65j                 1/1     Running            0             12m
kube-system     coredns-85cb69466-cdk8w                  1/1     Running            0             13m
kube-system     local-path-provisioner-64ffb68fd-6fr4m   1/1     Running            0             13m
kube-system     metrics-server-9cf544f65-qbl6c           1/1     Running            0             13m
kube-system     svclb-traefik-hflnf                      2/2     Running            0             12m
marketing       redis-db                                 1/1     Running            0             7m30s
dev             redis-db                                 1/1     Running            0             7m30s
finance         payroll                                  1/1     Running            0             7m29s
manufacturing   red-app                                  1/1     Running            0             7m30s
marketing       blue                                     1/1     Running            0             7m30s
finance         redis                                    1/1     Running            0             75s
research        dna-2                                    0/1     CrashLoopBackOff   6 (77s ago)   7m30s
research        dna-1                                    0/1     CrashLoopBackOff   6 (67s ago)   7m30s
```

```bash
kubectl get pods --all-namespaces | grep blue
```