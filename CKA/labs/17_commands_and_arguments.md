# Commands And Arguments Lab


How many PODs exist on the system?
> in the current(default) namespace
```bash
kubectl get pods --no-headers | wc -l
1
```
What is the command used to run the pod ubuntu-sleeper?
```bash
kubectl describe pods ubuntu-sleeper
Name:         ubuntu-sleeper
Namespace:    default
Priority:     0
Node:         controlplane/172.25.0.11
Start Time:   Sun, 27 Mar 2022 15:12:38 +0000
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           10.42.0.9
IPs:
  IP:  10.42.0.9
Containers:
  ubuntu:
    Container ID:  containerd://8b5c40ef44c1402bf721c5d86419974ebe5320b850c8a4a9f5e63fbedee3bd3d
    Image:         ubuntu
    Image ID:      docker.io/library/ubuntu@sha256:bea6d19168bbfd6af8d77c2cc3c572114eb5d113e6f422573c93cb605a0e2ffb
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      4800
```
Create a pod with the ubuntu image to run a container to sleep for 5000 seconds. Modify the file ubuntu-sleeper-2.yaml.



> Note: Only make the necessary changes. Do not modify the name.
```yaml
---
apiVersion: v1 
kind: Pod 
metadata:
  name: ubuntu-sleeper-2 
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command:
      - "sleep"
      - "5000"
```
```bash
kubectl get pods --watch
NAME               READY   STATUS    RESTARTS   AGE
ubuntu-sleeper     1/1     Running   0          6m20s
ubuntu-sleeper-2   1/1     Running   0          10s
```
Update pod ubuntu-sleeper-3 to sleep for 2000 seconds.

> kubectl edit cannot work here as, command is not an editable field
```bash
The Pod "ubuntu-sleeper-3" is invalid: spec: Forbidden: pod updates may not change fields other than `spec.containers[*].image`, `spec.initContainers[*].image`, `spec.activeDeadlineSeconds`, `spec.tolerations` (only additions to existing tolerations) or `spec.terminationGracePeriodSeconds` (allow it to be set to 1 if it was previously negative)
```
Then delete the pod
```bash
kubectl delete pods ubuntu-sleeper-3
pod "ubuntu-sleeper-3" deleted
```
Then apply 
```bash
kubectl apply -f ubuntu-sleeper-3.yaml 
pod/ubuntu-sleeper-3 created
```

Inspect the file Dockerfile given at /root/webapp-color directory. What command is run at container startup?
```bash
cat webapp-color/Dockerfile
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]
```
Inspect the file Dockerfile2 given at /root/webapp-color directory. What command is run at container startup?
```bash
cat webapp-color/Dockerfile2
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]

CMD ["--color", "red"]
```
Inspect the two files under directory webapp-color-2. What command is run at container startup?



> Assume the image was created from the Dockerfile in this folder.
```bash
cat webapp-color-2/Dockerfile2
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]

CMD ["--color", "red"]
```
```bash
cat webapp-color-2/webapp-color-pod.yaml
apiVersion: v1 
kind: Pod 
metadata:
  name: webapp-green
  labels:
      name: webapp-green 
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    command: ["--color","green"]
```
Inspect the two files under directory webapp-color-3. What command is run at container startup?



> Assume the image was created from the Dockerfile in this folder.
```bash
cat webapp-color-3/Dockerfile2
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]

CMD ["--color", "red"]

controlplane ~ ➜  cat webapp-color-3/webapp-color-pod-2.yaml
apiVersion: v1 
kind: Pod 
metadata:
  name: webapp-green
  labels:
      name: webapp-green 
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    command: ["python", "app.py"]
    args: ["--color", "pink"]
```

Create a pod with the given specifications. By default it displays a blue background. Set the given command line arguments to change it to green


* Pod Name: webapp-green
* Image: kodekloud/webapp-color
* Command line arguments: --color=green
```bash
kubectl run webapp-green --image=kodekloud/webapp-color --dry-run=client -o yaml > webapp-green.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: webapp-green
  name: webapp-green
spec:
  containers:
  - image: kodekloud/webapp-color
    name: webapp-green
    args: ["--color", "green"]
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```
