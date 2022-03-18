# Commands

Deploy a pod named nginx-pod using the nginx:alpine image.
```bash
kubectl run --image=nginx:alpine nginx-pod
```
Deploy a redis pod using the redis:alpine image with the labels set to tier=db.



Either use imperative commands to create the pod with the labels. Or else use imperative commands to generate the pod definition file, then add the labels before creating the pod using the file.


```bash
kubectl run --image=redis:alpine redis --dry-run=client -o yaml > redis.yaml

# add the label

kubectl apply -f redis.yaml
```
or
```bash
kubectl run --image=redis:alpine redis --label=tie=db
```
Create a service redis-service to expose the redis application within the cluster on port 6379
```bash
kubectl expose pod redis --port=6379 --target-port=6379 --name redis-service
```

Create a deployment named webapp using the image kodekloud/webapp-color with 3 replicas.
```bash
kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3
```

Create a new pod called custom-nginx using the nginx image and expose it on container port 8080.
```bash
kubectl run custom-nginx --image=nginx --port=8080
```

Create a new deployment called redis-deploy in the dev-ns namespace with the redis image. It should have 2 replicas.

```bash
kubectl create deployment redis-deploy --image=redis --replicas=2 --namespace=dev-ns
```

Create a pod called httpd using the image httpd:alpine in the default namespace. Next, create a service of type ClusterIP by the same name (httpd). The target port for the service should be 80.
```bash
kubectl run httpd --image=httpd:alpine --port=80 --expose
```