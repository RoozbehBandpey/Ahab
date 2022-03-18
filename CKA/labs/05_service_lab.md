# Namespace practice test
```bash
kubectl get services

# Output
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   7m57s
```

That is a default service created by Kubernetes at launch.

What is the targetPort configured on the kubernetes service?

```bash
kubectl describe service kubernetes
or
kubectl describe svc kubernetes

# Output
Name:              kubernetes
Namespace:         default
Labels:            component=apiserver
                   provider=kubernetes
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.43.0.1
IPs:               10.43.0.1
Port:              https  443/TCP
TargetPort:        6443/TCP
Endpoints:         10.42.238.6:6443
Session Affinity:  None
Events:            <none>
```