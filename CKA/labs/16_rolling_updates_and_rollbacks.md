# Rolling Updates and Rollbacks Lab

We have deployed a simple web application. Inspect the PODs and the Services

```bash
kubectl get pods,services
NAME                            READY   STATUS    RESTARTS   AGE
pod/frontend-7776cb7d57-d8jrl   1/1     Running   0          78s
pod/frontend-7776cb7d57-qvjrn   1/1     Running   0          78s
pod/frontend-7776cb7d57-h6nw6   1/1     Running   0          78s
pod/frontend-7776cb7d57-tq5pt   1/1     Running   0          78s

NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/kubernetes       ClusterIP   10.43.0.1       <none>        443/TCP          4m37s
service/webapp-service   NodePort    10.43.250.130   <none>        8080:30080/TCP   78s
```
Inspect the deployment and identify the number of PODs deployed by it
```bash
kubectl get deployments
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
frontend   4/4     4            4           5m14s
```

What container image is used to deploy the applications?
```bash
kubectl describe deployment frontend | grep -i Image
    Image:        kodekloud/webapp-color:v1
```

Inspect the deployment and identify the current strategy
```bash
kubectl describe deployment frontend | grep -i StrategyType
StrategyType:           RollingUpdate
```
Upgrade the application by setting the image on the deployment to kodekloud/webapp-color:v2



>Do not delete and re-create the deployment. Only set the new image name for the existing deployment.

```bash
kubectl edit deployment frontend
deployment.apps/frontend edited

controlplane ~ ➜  kubectl describe deployment frontend | grep -i Image
    Image:        kodekloud/webapp-color:v2
```

Change the deployment strategy to Recreate



>Delete and re-create the deployment if necessary. Only update the strategy type for the existing deployment.


Run the command kubectl edit deployment frontend and modify the required field. Make sure to delete the properties of rollingUpdate as well, set at strategy.rollingUpdate.
```bash
kubectl describe deployment frontend | grep -i StrategyType
StrategyType:       Recreate
```
Upgrade the application by setting the image on the deployment to kodekloud/webapp-color:v3



>Do not delete and re-create the deployment. Only set the new image name for the existing deployment.
```bash
kubectl describe deployment frontend
Name:               frontend
Namespace:          default
CreationTimestamp:  Sun, 27 Mar 2022 13:19:49 +0000
Labels:             <none>
Annotations:        deployment.kubernetes.io/revision: 3
Selector:           name=webapp
Replicas:           4 desired | 4 updated | 4 total | 0 available | 4 unavailable
StrategyType:       Recreate
MinReadySeconds:    20
Pod Template:
  Labels:  name=webapp
  Containers:
   simple-webapp:
    Image:        kodekloud/webapp-color:v3
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      False   MinimumReplicasUnavailable
  Progressing    True    ReplicaSetUpdated
OldReplicaSets:  <none>
NewReplicaSet:   frontend-c68667579 (4/4 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  18m    deployment-controller  Scaled up replica set frontend-7776cb7d57 to 4
  Normal  ScalingReplicaSet  5m9s   deployment-controller  Scaled up replica set frontend-7c7fcfc8cb to 1
  Normal  ScalingReplicaSet  5m9s   deployment-controller  Scaled down replica set frontend-7776cb7d57 to 3
  Normal  ScalingReplicaSet  5m8s   deployment-controller  Scaled up replica set frontend-7c7fcfc8cb to 2
  Normal  ScalingReplicaSet  4m46s  deployment-controller  Scaled down replica set frontend-7776cb7d57 to 1
  Normal  ScalingReplicaSet  4m46s  deployment-controller  Scaled up replica set frontend-7c7fcfc8cb to 4
  Normal  ScalingReplicaSet  4m23s  deployment-controller  Scaled down replica set frontend-7776cb7d57 to 0
  Normal  ScalingReplicaSet  35s    deployment-controller  Scaled down replica set frontend-7c7fcfc8cb to 0
  Normal  ScalingReplicaSet  3s     deployment-controller  Scaled up replica set frontend-c68667579 to 4
```