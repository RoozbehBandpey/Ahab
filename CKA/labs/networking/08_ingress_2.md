# Ingress Lab 2

Let us now deploy an Ingress Controller. First, create a namespace called ingress-space.



>We will isolate all ingress related objects into its own namespace.
```bash
kubectl create namespace ingress-space
namespace/ingress-namespace created
```
The NGINX Ingress Controller requires a ConfigMap object. Create a ConfigMap object in the ingress-space.



>Use the spec given below. No data needs to be configured in the ConfigMap.
```bash
kubectl create configmap nginx-configuration --namespace ingress-space
configmap/nginx-configuration created
```


The NGINX Ingress Controller requires a ServiceAccount. Create a ServiceAccount in the ingress-space namespace.
```bash
kubectl create serviceaccount ingress-serviceaccount --namespace ingress-space
serviceaccount/ingress-serviceaccount created
```
We have created the Roles and RoleBindings for the ServiceAccount. Check it out!!
```bash
kubectl get roles -n ingress-space
NAME           CREATED AT
ingress-role   2022-04-10T19:06:10Z
root@controlplane:~# kubectl describe role ingress-role -n ingress-space
Name:         ingress-role
Labels:       app.kubernetes.io/name=ingress-nginx
              app.kubernetes.io/part-of=ingress-nginx
Annotations:  <none>
PolicyRule:
  Resources   Non-Resource URLs  Resource Names                     Verbs
  ---------   -----------------  --------------                     -----
  configmaps  []                 []                                 [get create]
  configmaps  []                 [ingress-controller-leader-nginx]  [get update]
  endpoints   []                 []                                 [get]
  namespaces  []                 []                                 [get]
  pods        []                 []                                 [get]
  secrets     []                 []                                 [get]
```
```bash
kubectl get rolebinding -n ingress-space
NAME                   ROLE                AGE
ingress-role-binding   Role/ingress-role   2m29s
root@controlplane:~# kubectl describe rolebinding ingress-role-binding -n ingress-space
Name:         ingress-role-binding
Labels:       app.kubernetes.io/name=ingress-nginx
              app.kubernetes.io/part-of=ingress-nginx
Annotations:  <none>
Role:
  Kind:  Role
  Name:  ingress-role
Subjects:
  Kind            Name                    Namespace
  ----            ----                    ---------
  ServiceAccount  ingress-serviceaccount  
```
Let us now deploy the Ingress Controller. Create a deployment using the file given.



Let us now create a service to make Ingress available to external users.

```bash
kubectl expose deployment ingress-controller -n ingress-space --type=NodePort --port=80 --name=ingress --dry-run=client -o yaml > ingress.yaml
root@controlplane:~# ls
ingress-controller.yaml  ingress.yaml  sample.yaml
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: ingress
  namespace: ingress-space
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    nodePort: 30080
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  selector:
    name: nginx-ingress
```
Create the ingress resource to make the applications available at /wear and /watch on the Ingress service.



>Create the ingress in the app-space namespace.
```bash
kubectl create ingress ingress-wear-watch --namespace=app-space --rule="host/path=service:port" --dry-run=client -o yaml > ing.yaml
```
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch
  namespace: app-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /wear
        pathType: Prefix
        backend:
          service:
           name: wear-service
           port: 
            number: 8080
      - path: /watch
        pathType: Prefix
        backend:
          service:
           name: video-service
           port:
            number: 8080
```