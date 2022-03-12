# Pods practice test

Create a new pod with the nginx image.

kubectl run nginx --image nginx

controlplane ~ ➜  kubectl get pods -n default
NAME    READY   STATUS              RESTARTS   AGE
nginx   0/1     ContainerCreating   0          5s

controlplane ~ ➜  kubectl get pods -n default
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          12s


What is the image used to create the new pods?



You must look at one of the new pods in detail to figure this out.



kubectl describe pod newpods-2vh4b

Name:         newpods-2vh4b
Namespace:    default
Priority:     0
Node:         controlplane/172.25.0.45
Start Time:   Sat, 12 Mar 2022 12:18:11 +0000
Labels:       tier=busybox
Annotations:  <none>
Status:       Running
IP:           10.42.0.11
IPs:
  IP:           10.42.0.11
Controlled By:  ReplicaSet/newpods
Containers:
  busybox:
    Container ID:  containerd://45c5025536136d2f60ed2ff83fbb19426f1d8d8231d4ce7bf8cac4105d6a85ea
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:caa382c432891547782ce7140fb3b7304613d3b0438834dce1cad68896ab110a
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      1000
    State:          Running
      Started:      Sat, 12 Mar 2022 12:18:18 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-fwscn (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-fwscn:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  91s   default-scheduler  Successfully assigned default/newpods-2vh4b to controlplane
  Normal  Pulling    90s   kubelet            Pulling image "busybox"
  Normal  Pulled     84s   kubelet            Successfully pulled image "busybox" in 5.697927889s
  Normal  Created    84s   kubelet            Created container busybox
  Normal  Started    84s   kubelet            Started container busybox


  Which nodes are these pods placed on?


  kubectl get pods -o wide
  NAME            READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
nginx           1/1     Running   0          3m54s   10.42.0.9    controlplane   <none>           <none>
newpods-2vh4b   1/1     Running   0          3m26s   10.42.0.11   controlplane   <none>           <none>
newpods-zz4p8   1/1     Running   0          3m26s   10.42.0.10   controlplane   <none>           <none>
newpods-xtrkm   1/1     Running   0          3m26s   10.42.0.12   controlplane   <none>           <none>

How many containers are part of the pod webapp?

kubectl describe pod webapp


What does the READY column in the output of the kubectl get pods command indicate?

Running Containers in POD/Total Containers in POD

Delete the webapp Pod.

kubectl delete pods webapp


Create a new pod with the name redis and with the image redis123.



Use a pod-definition YAML file. And yes the image name is wrong!


controlplane ~ ➜  cat redis.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: redis
  labels:
    app: redis

specs:
  containers:
  - name: redis-container
    image: redis123



    spec instead of specs nop S


    Now change the image on this pod to redis.



Once done, the pod should be in a running state.

vi redis.yaml

controlplane ~ ➜  cat redis.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: redis
  labels:
    app: redis

spec:
  containers:
  - name: redis-container
    image: redis

controlplane ~ ➜  kubectl create -f redis.yaml
Error from server (AlreadyExists): error when creating "redis.yaml": pods "redis" already exists

controlplane ~ ✖ kubectl delete pod redis