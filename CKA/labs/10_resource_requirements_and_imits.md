# Resource Requirements and Limits

A pod called rabbit is deployed. Identify the CPU requirements set on the Pod

in the current(default) namespace
```bash
kubectl describe pods rabbit
Name:         rabbit
Namespace:    default
...
    Restart Count:  2
    Limits:
      cpu:  2
    Requests:
      cpu:        1
    Environment:  <none>
    Mounts:
...
```

Another pod called elephant has been deployed in the default namespace. It fails to get to a running state. Inspect this pod and identify the Reason why it is not running.
```bash
kubectl describe pods elephant
...
 State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       OOMKilled
      Exit Code:    1
...
```
The status OOMKilled indicates that it is failing because the pod ran out of memory. Identify the memory limit set on the POD.



The elephant pod runs a process that consume 15Mi of memory. Increase the limit of the elephant pod to 20Mi.



Delete and recreate the pod if required. Do not modify anything other than the required fields.

```bash
controlplane ~ ➜  kubectl get pods elephant -o yaml > elephant.yaml

controlplane ~ ➜  ls
elephant.yaml  sample.yaml

controlplane ~ ➜  vi elephant.yaml 

controlplane ~ ➜  kubectl delete pods elephant 
pod "elephant" deleted

controlplane ~ ➜  kubectl apply -f elephant.yaml 
pod/elephant created

controlplane ~ ➜  kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
elephant   1/1     Running   0          7s
```

