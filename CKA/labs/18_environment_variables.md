# Environment Variables Lab

What is the environment variable name set on the container in the pod?
```bash
kubectl describe pod webapp-color
Name:         webapp-color
Namespace:    default
Priority:     0
Node:         controlplane/172.25.0.37
Start Time:   Sun, 27 Mar 2022 19:14:07 +0000
Labels:       name=webapp-color
Annotations:  <none>
Status:       Running
IP:           10.42.0.9
IPs:
  IP:  10.42.0.9
Containers:
  webapp-color:
    Container ID:   containerd://f447cd733fdcd425e025f28a2f1a53ff00358e9a29e02ecc5abc7ff09cbb3aa4
    Image:          kodekloud/webapp-color
    Image ID:       docker.io/kodekloud/webapp-color@sha256:99c3821ea49b89c7a22d3eebab5c2e1ec651452e7675af243485034a72eb1423
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sun, 27 Mar 2022 19:14:14 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      APP_COLOR:  pink
```
Update the environment variable on the POD to display a green background



> Note: Delete and recreate the POD. Only make the necessary changes. Do not modify the name of the Pod.

```bash
kubectl get pods webapp-color -o yaml > web-app.yaml

kubectl delete pods webapp-color

kubectl apply -f web-app.yaml 
```

Identify the database host from the config map db-config
```bash
kubectl get configmaps
NAME               DATA   AGE
kube-root-ca.crt   1      17m
db-config          3      12s

controlplane ~ ➜  kubectl describe configmap db-config
Name:         db-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
DB_HOST:
----
SQL01.example.com
DB_NAME:
----
SQL01
DB_PORT:
----
3306

BinaryData
====

Events:  <none>
```
Create a new ConfigMap for the webapp-color POD. Use the spec given below.

* ConfigName Name: webapp-config-map
* Data: APP_COLOR=darkblue
```bash
kubectl create configmap webapp-config-map --from-literal=APP_COLOR=darkblue
configmap/webapp-config-map created
```
Update the environment variable on the POD to use the newly created ConfigMap



> Note: Delete and recreate the POD. Only make the necessary changes. Do not modify the name of the Pod.

```yaml
  - envFrom:
    - configMapRef:
        name: webapp-config-map
```
```bash
kubectl delete pods webapp-color
pod "webapp-color" deleted
```
```bash
kubectl apply -f web-app.yaml 
pod/webapp-color created
```

A nice way to explore pods:

```bash
kubectl explain pods --recursive | grep envFrom -A3
```