# Mock Exam 1

Deploy a pod named nginx-pod using the nginx:alpine image.



Once done, click on the Next Question button in the top right corner of this panel. You may navigate back and forth freely between all questions. Once done with all questions, click on End Exam. Your work will be validated at the end and score shown. Good Luck!
```bash
kubectl run nginx-pod --image=nginx:alpine
pod/nginx-pod created
```
Deploy a messaging pod using the redis:alpine image with the labels set to tier=msg.
```bash
kubectl run messaging --image=redis:alpine --labels tier=msg
```


Create a namespace named apx-x9984574.
```bash
kubectl create namespace apx-x9984574
namespace/apx-x9984574 created
```
Get the list of nodes in JSON format and store it in a file at /opt/outputs/nodes-z3444kd9.json.
```bash
kubectl get nodes -o json > /opt/outputs/nodes-z3444kd9.json
```

Create a service messaging-service to expose the messaging application within the cluster on port 6379.
```bash
kubectl expose pod messaging --port=6379 --name messaging-service
service/messaging-service exposed
```
Create a deployment named hr-web-app using the image kodekloud/webapp-color with 2 replicas.

```bash
kubectl create deployment hr-web-app --image=kodekloud/webapp-color --replicas=2
deployment.apps/hr-web-app created
```

Create a static pod named static-busybox on the controlplane node that uses the busybox image and the command sleep 1000.
```bash
kubectl run static-busybox --image=busybox --command sleep 1000 --dry-run=client -o yaml > pod.yaml

kubectl run static-busybox --image=busybox --command sleep 1000 --dry-run=client -o yaml > /etc/kubernetes/manifests/pod.yaml
root@controlplane:~# ls /etc/kubernetes/manifestsetcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml  pod.yaml
```
```bash
kubectl get pods -A -o wide
NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE     IP            NODE           NOMINATED NODE   READINESS GATES
default       hr-web-app-99dfd4c9d-b5j2z             1/1     Running   0          3m3s    10.244.0.7    controlplane   <none>           <none>
default       hr-web-app-99dfd4c9d-h85cr             1/1     Running   0          3m3s    10.244.0.6    controlplane   <none>           <none>
default       messaging                              1/1     Running   0          7m59s   10.244.0.5    controlplane   <none>           <none>
default       nginx-pod                              1/1     Running   0          11m     10.244.0.4    controlplane   <none>           <none>
default       static-busybox-controlplane            1/1     Running   0          25s     10.244.0.8    controlplane 
```
Create a POD in the finance namespace named temp-bus with the image redis:alpine.

```bash
kubectl run temp-bus -n finance --image=redis:alpine
pod/temp-bus created
root@controlplane:~# kubectl get pods -A
NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
default       hr-web-app-99dfd4c9d-b5j2z             1/1     Running   0          4m30s
default       hr-web-app-99dfd4c9d-h85cr             1/1     Running   0          4m30s
default       messaging                              1/1     Running   0          9m26s
default       nginx-pod                              1/1     Running   0          13m
default       static-busybox-controlplane            1/1     Running   0          112s
finance       temp-bus                               1/1     Running   0          4s
```
A new application orange is deployed. There is something wrong with it. Identify and fix the issue.

Expose the hr-web-app as service hr-web-app-service application on port 30082 on the nodes on the cluster.



The web application listens on port 8080.
```bash
kubectl get deployments
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
hr-web-app   2/2     2            2           12m
```
```bash
kubectl expose deployment hr-web-app --type=NodePort --port=8080 --name=hr-web-app-service --dry-run=client -o yaml > hr-web-app-service.yaml
```

>add the nodePort field 


Use JSON PATH query to retrieve the osImages of all the nodes and store it in a file /opt/outputs/nodes_os_x43kj56.txt.



The osImages are under the nodeInfo section under status of each node.
```bash
kubectl get nodes                                          
NAME           STATUS   ROLES                  AGE   VERSION
controlplane   Ready    control-plane,master   56m   v1.20.0
root@controlplane:~# kubectl get nodes -o=jsonpath='{ .items[*].status.nodeInfo.osImage }' > /opt/outputs/nodes_os_x43kj56.txt
```

Create a Persistent Volume with the given specification.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-analytics
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /pv/data-analytics
```