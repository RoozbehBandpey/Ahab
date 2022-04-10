# Ingress Lab 1

Which namespace is the Ingress Controller deployed in?
```bash
kubectl get all -A | grep ngress
ingress-space   pod/nginx-ingress-controller-697cfbd4d9-j5b95   1/1     Running   0          3m10s
ingress-space   service/ingress-service        NodePort    10.96.53.166     <none>        80:30080/TCP,443:31400/TCP   3m11s
ingress-space   deployment.apps/nginx-ingress-controller   1/1     1            1           3m11s
ingress-space   replicaset.apps/nginx-ingress-controller-697cfbd4d9   1         1         1       3m11s
```
How many applications are deployed in the app-space namespace?



>Count the number of deployments in this namespace.
```bash
kubectl get deployments -n app-space
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
default-backend   1/1     1            1           4m43s
webapp-video      1/1     1            1           4m43s
webapp-wear       1/1     1            1           4m43s
```
Which namespace is the Ingress Resource deployed in?
```bash
kubectl get ingress -A
NAMESPACE   NAME                 CLASS    HOSTS   ADDRESS   PORTS   AGE
app-space   ingress-wear-watch   <none>   *                 80      5m34s
```
What is the Host configured on the Ingress Resource?



>The host entry defines the domain name that users use to reach the application like www.google.com

```bash
kubectl describe ingress ingress-wear-watch -n app-space
Name:             ingress-wear-watch
Namespace:        app-space
Address:          
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host        Path  Backends
  ----        ----  --------
  *           
              /wear    wear-service:8080 (10.244.0.7:8080)
              /watch   video-service:8080 (10.244.0.4:8080)
Annotations:  nginx.ingress.kubernetes.io/rewrite-target: /
              nginx.ingress.kubernetes.io/ssl-redirect: false
```

You are requested to change the URLs at which the applications are made available.



>Make the video application available at `/stream`.
```bash
kubectl edit ingress --namespace app-space
```
And change the path to the video streaming application to `/stream`.


Due to increased demand, your business decides to take on a new venture. You acquired a food delivery company. Their applications have been migrated over to your cluster.



>Inspect the new deployments in the app-space namespace.
```bash
kubectl get deployments -n app-space
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
default-backend   1/1     1            1           13m
webapp-food       1/1     1            1           68s
webapp-video      1/1     1            1           13m
webapp-wear       1/1     1            1           13m
```
You are requested to add a new path to your ingress to make the food delivery application available to your customers.



>Make the new application available at `/eat`.
```bash
kubectl get svc -n app-space
NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
default-http-backend   ClusterIP   10.96.135.152    <none>        80/TCP     14m
food-service           ClusterIP   10.102.255.93    <none>        8080/TCP   117s
video-service          ClusterIP   10.106.29.236    <none>        8080/TCP   14m
wear-service           ClusterIP   10.108.138.244   <none>        8080/TCP   14m
```
```yaml
      - backend:
          service:
            name: food-service
            port:
              number: 8080
        path: /eat
        pathType: Prefix
```

A new payment service has been introduced. Since it is critical, the new application is deployed in its own namespace.



>Identify the namespace in which the new application is deployed.
```bash
kubectl get svc -A
NAMESPACE        NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
app-space        default-http-backend   ClusterIP   10.96.135.152    <none>        80/TCP                       16m
app-space        food-service           ClusterIP   10.102.255.93    <none>        8080/TCP                     4m12s
app-space        video-service          ClusterIP   10.106.29.236    <none>        8080/TCP                     16m
app-space        wear-service           ClusterIP   10.108.138.244   <none>        8080/TCP                     16m
critical-space   pay-service            ClusterIP   10.109.195.115   <none>        8282/TCP                     28s
default          kubernetes             ClusterIP   10.96.0.1        <none>        443/TCP                      23m
ingress-space    ingress-service        NodePort    10.96.53.166     <none>        80:30080/TCP,443:31400/TCP   16m
kube-system      kube-dns               ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP       23m
```

What is the name of the deployment of the new application?
```bash
kubectl get deploy -A
NAMESPACE        NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
app-space        default-backend            1/1     1            1           17m
app-space        webapp-food                1/1     1            1           4m46s
app-space        webapp-video               1/1     1            1           17m
app-space        webapp-wear                1/1     1            1           17m
critical-space   webapp-pay                 1/1     1            1           62s
ingress-space    nginx-ingress-controller   1/1     1            1           17m
kube-system      coredns    
```
You are requested to make the new application available at `/pay`.



>Identify and implement the best approach to making this application available on the ingress controller and test to make sure its working. Look into annotations: rewrite-target as well.
```bash
kubectl get ingress -n app-space -o yaml > ingress.yaml
```
```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /pay
        pathType: Prefix
        backend:
          service:
           name: pay-service
           port:
            number: 8282
```