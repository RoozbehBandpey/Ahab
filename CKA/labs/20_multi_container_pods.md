# Multi Container Pods Lab

Identify the number of containers created in the red pod.
```bash
kubectl describe pod red
```
Create a multi-container pod with 2 containers.



Use the spec given below.
> If the pod goes into the crashloopbackoff then add sleep 1000 in the lemon container.

* Name: yellow
* Container 1 Name: lemon
* Container 1 Image: busybox
* Container 2 Name: gold
* Container 2 Image: redis
```bash
kubectl run yellow --dry-run=client --image=redis -o yaml > tellow.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: yellow
  name: yellow
spec:
  containers:
  - image: redis
    name: yellow
    resources: {}
  - image: ubuntu
    name: gold
    command:
      - "sleep"
      - "1000"
  - image: ubuntu
    name: lemon
    command:
      - "sleep"
      - "1000"
  - image: busybox
    name: busybox
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

The application outputs logs to the file /log/app.log. View the logs and try to identify the user having issues with Login.



> Inspect the log file inside the pod.
```bash
kubectl -n elastic-stack exec -it app -- cat /log/app.log
```

Edit the pod to add a sidecar container to send logs to Elastic Search. Mount the log volume to the sidecar container.



> Only add a new container. Do not modify anything else. Use the spec provided below.


* Name: app
* Container Name: sidecar
* Container Image: kodekloud/filebeat-configured
* Volume Mount: log-volume
* Mount Path: /var/log/event-simulator/
* Existing Container Name: app
* Existing Container Image: kodekloud/event-simulator
```bash
kubectl edit pod app -n elastic-stack
```

```yaml

 - image: kodekloud/filebeat-configured
    name: sidecar
    volumeMounts:
    - mountPath: /var/log/event-simulator/
      name: log-volume
```

> Forbidden: pod updates may not add or remove containers
```bash
kubectl get pods -n elastic-stack -o yaml > app.yaml
root@controlplane:~# ls
app.yaml 

kubectl delete pods app -n elastic-stack 


kubectl apply -f app.yaml
pod/app created
Warning: resource pods/elastic-search is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
pod/elastic-search configured
Warning: resource pods/kibana is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
pod/kibana configured
```