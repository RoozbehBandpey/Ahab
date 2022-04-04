# Security
Understanding kubernetes security primitives, how does someone gain access to the cluster. 


## Kubernetes Security Primitives
All access to host must be secured, root access disabled, password based authentication disabled, only SSH key authentication to be made available. 

Our focus here is on kubernetes related security, what are the risks and what  are the measures  to be taken to secure the cluster.

The first line of defense is kube-apiserver, as we do everything on the cluster via kube-apiserver. We need to make two type of decision 1. Who can access the cluster? 2. What can they do?

Who can access the kube-apiserver is defined by the authentication mechanisms:
* Files - Usernames and Passwords
* Files - Usernames and Tokens
* Certificates
* External Authentication Provider - LDAP
* Service accounts

Once they get access to the cluster, what they can do is defined by authorization mechanisms:
* RBAC Authorizations
* ABAC Authorization
* Node Authorization
* Webhook mode

All communication between the cluster and various components such as kubelet, ETCD cluster, kube-proxy, scheduler and controller manager is secured using TLS encryption. 

By default all pods can access all other pods within the cluster, we can restrict access between them using network policies. 

## Authentication
In a kubernetes cluster we have users like administrators who access cluster to perform administrative tasks. We have developers who deploy their applications and we have end users who access the applications deployed on the cluster. And we have third-party applications which access the cluster for integration purposes.

Let's focus on accessing to the kubernetes cluster with authentication mechanisms:

* Authentication of end users: will be managed internally within the application

So we're left with two types of user: humans (devs and admins) and programs (service accounts)

Kubernetes does not manage user accounts natively, it relies on an external source like files with user details or certificates or third-party identity service. We cannot create or view users but in case of service account we can do that. 

Let's focus on human users on kubernetes, all user access in kubernetes is managed via kube-apiserver. Every request goes to kube-apiserver, the kube-apiserver authenticate the requests before processing it. 

### Static password and token files
We can create a list of users and their passwords in a csv file, and use that as the source for user information. The file has three columns `password`, `username` and `user id` we then pass the file as an option to kube-apiserver `--basic-auth-file=user-details.csv`. For this option to take effect we must restart the kube-apiserver.

With kubeadm tool, we must modify the kube-apiserver pod definition file, the kubeadm then automatically restarts the kube-apiserver once the file is saved with changes.

To authenticate use the curl command:


```bash
curl -v -k https://master-node-ip:6443/api/v1/pods -u "user1:password123"
```
In the csv file we can optionally have a forth columns as `group details` to assign users to groups.

Similarly we can have a static token file, here instead of password we specify a token. Then we pass the token file as an option to the kube-apiserver `--token-auth-file=user-details.csv`. While authenticating we specify the token as an authorization Bearer token in the request:
```bash
curl -v -k https://master-node-ip:6443/api/v1/pods --header "Authorization: Bearer JHU545UYJYBJYGUU123234JHJHVBJYU89"
```

All of these is insecure and it is not the recommended approach

## Article on Setting up Basic Authentication
Setup basic authentication on Kubernetes (Deprecated in 1.19)
Note: This is not recommended in a production environment. This is only for learning purposes. Also note that this approach is deprecated in Kubernetes version 1.19 and is no longer available in later releases

Follow the below instructions to configure basic authentication in a kubeadm setup.

Create a file with user details locally at `/tmp/users/user-details.csv`

User File Contents
```csv
password123,user1,u0001
password123,user2,u0002
password123,user3,u0003
password123,user4,u0004
password123,user5,u0005
```

Edit the kube-apiserver static pod configured by kubeadm to pass in the user details. The file is located at `/etc/kubernetes/manifests/kube-apiserver.yaml`


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
      <content-hidden>
    image: k8s.gcr.io/kube-apiserver-amd64:v1.11.3
    name: kube-apiserver
    volumeMounts:
    - mountPath: /tmp/users
      name: usr-details
      readOnly: true
  volumes:
  - hostPath:
      path: /tmp/users
      type: DirectoryOrCreate
    name: usr-details
```

Modify the kube-apiserver startup options to include the basic-auth file


```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --authorization-mode=Node,RBAC
      <content-hidden>
    - --basic-auth-file=/tmp/users/user-details.csv
```
Create the necessary roles and role bindings for these users:


```yaml
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
 
---
# This role binding allows "jane" to read pods in the "default" namespace.
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: user1 # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```
Once created, you may authenticate into the kube-api server using the users credentials
```bash
curl -v -k https://localhost:6443/api/v1/pods -u "user1:password123"
```
## TLS Basics
A certificate is used to guarantee trust between two parties during a transaction. For example when a user tries to access a webserver, TLS certificate ensures that the communication between the user and the server is encrypted and the server is who it says it is. 

Without secure connectivity, a user trying to access his/her bank account the credentials they enter in the portal would be send in the plain text. The hackers sniffing into the traffic could easily retrieve the credentials and log into user's bank account. So we must encrypt the data being transferred using encryption keys. Now data is encrypted so the hacker won't be able to do anything with the data, but the server as well cannot do anything with the data without encryption key. So a copy of the key must also be send to the server, since the key is also send using the same network it can be compromised and used to decrypt the data. This is known as symmetric encryption. 

That's where asymmetric encryption comes in, instead of using a single key to encrypt and decrypt data, this method uses a pair of keys a private key and a public key. The public key is more of a public lock. The trick is if you encrypt your data with the lock you can only unlock it with the associated key. 

For example when you have a server in your environment that you need to access to, and you don't want to use passwords as  they are too ricky. So you decide to use key-pairs, you generate a public a private key-pair. 

```bash
ssh-keygen
```

It generates two files, `id_rsa` is the private key and `id_rsa.pub` is the public key. Then we secure the server by locking down all access to it, except through a door that is locked using public key. It is usually done by adding an entry with your public key in the server's authorized keys file `cat ./.ssh/authorized_keys` The lock is public and anyone can attempt to break through. When we try to ssh, we specify the location of our private key in our command. For multiple servers we can make a copies of the public key and place then on as many servers as possible. TODO
## TLS in Kubernetes

## TLS in Kubernetes - Certificate Creation
Generate private key with openssl
### Generating Certificates
```bash
openssl genrsa -out ca.key 2048 
```

Generate a certificate signing request
```bash
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
```
Sign the request
```bash
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```
### Generating Client Certificates
```bash
openssl genrsa -out admin.key 2048 
```

Generate a certificate signing request
```bash
openssl req -new -key admin.key -subj "/CN=kube-admin" -out admin.csr
```
Sign the request with specifying the ca certificate and ca key
```bash
openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
```

We can differentiate users by adding group detail while creating the csr
```bash
openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr
```
## View Certificate Details
Performing health check on certificates:

If thee cluster was setup from scratch, you generate all the certificates by yourself, 
```bash
cat /etc/systemd/system/kube-apiserver.service
```


Kubeadm takes care of generating the certificates
```bash
cat /rtc/kubernetes/manifests/kube-apiserver.yaml
```

## Certificates API

A user first creates a key
```bash
openssl genrsa -out roozbeh.key 2048
```
Then creates a csr using the key with the name on it
```bash
openssl req -new -key roozbeh.key -subj "/CN=roozbeh" -out roozbeh.csr
```

Then sends the request to the administrator, the admin takes the key and creates a csr object. The admin won't place the csr as plain text in request section on CSR manifest file, instead encodes it with base64

```bash
cat roozbeh.csr | base64
```
Then move the encoded request in the request filed and submit the request. 

Once object is created the admin can view all CSRs

```bash
kubectl get csr
```
 Approve the request by running
```bash
kubectl certificate approve roozbeh
```


## KubeConfig
kubectl config view

## Persistent Key/Value Store

## API Groups

## Authorization
Inspect the environment and identify the authorization modes configured on the cluster.



Check the kube-apiserver settings.

cat /etc/kubernetes/manifests/kube-apiserver.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 10.12.220.3:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=10.12.220.3
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC

Use the command kubectl describe pod kube-apiserver-controlplane -n kube-system and look for --authorization-mode.

kubectl describe pod kube-apiserver-controlplane -n kube-system  | grep authorization-mode
      --authorization-mode=Node,RBAC

How many roles exist in the default namespace?

kubectl get roles

How many roles exist in all namespaces together?

kubectl get roles -A --no-headers | wc -l
12

What are the resources the kube-proxy role in the kube-system namespace is given access to?

kubectl describe roles kube-proxy -n kube-system
Name:         kube-proxy
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources   Non-Resource URLs  Resource Names  Verbs
  ---------   -----------------  --------------  -----
  configmaps  []                 [kube-proxy]    [get]

Which account is the kube-proxy role assigned to it?

kubectl describe rolebindings kube-proxy -n kube-system
Name:         kube-proxy
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  kube-proxy
Subjects:
  Kind   Name                                             Namespace
  ----   ----                                             ---------
  Group  system:bootstrappers:kubeadm:default-node-token  

A user dev-user is created. User's details have been added to the kubeconfig file. Inspect the permissions granted to the user. Check if the user can list pods in the default namespace.



Use the --as dev-user option with kubectl to run commands as the dev-user.

kubectl get pods --as dev-user
Error from server (Forbidden): pods is forbidden: User "dev-user" cannot list resource "pods" in API group "" in the namespace "default"

Create the necessary roles and role bindings required for the dev-user to create, list and delete pods in the default namespace.



Use the given spec:


* Role: developer
* Role Resources: pods
* Role Actions: list
* Role Actions: create
* Role Actions: delete
* RoleBinding: dev-user-binding
* RoleBinding: Bound to dev-user

To create a Role:- kubectl create role developer --namespace=default --verb=list,create,delete --resource=pods
To create a RoleBinding:- kubectl create rolebinding dev-user-binding --namespace=default --role=developer --user=dev-user


The dev-user is trying to get details about the dark-blue-app pod in the blue namespace. Investigate and fix the issue.



We have created the required roles and rolebindings, but something seems to be wrong.

Run the command: kubectl edit role developer -n blue and correct the resourceNames field. You don't have to delete the role.

Grant the dev-user permissions to create deployments in the blue namespace.



Remember to add for api group "apps".

kubectl create role dev-user-role --namespace=blue --verb=create  --resource=deploymentsrole.rbac.authorization.k8s.io/dev-user-role created
root@controlplane:~# kubectl edit role dev-user-role
Error from server (NotFound): roles.rbac.authorization.k8s.io "dev-user-role" not found
root@controlplane:~# kubectl edit role dev-user-role -n blue
Edit cancelled, no changes made.
root@controlplane:~# kubectl create rolebinding dev-user-role-binding --namespace=blue --role=dev-user-role --user=dev-user
rolebinding.rbac.authorization.k8s.io/dev-user-role-binding created
## Role Based Access Controls


## Cluster Roles and Role Bindings
How many ClusterRoles do you see defined in the cluster?


kubectl get clusterroles --no-headers | wc -l
69

kubectl get clusterroles --no-headers -o json | jq '.items | length'

kubectl get clusterrolebindings --no-headers | wc -l
54

A new user michelle joined the team. She will be focusing on the nodes in the cluster. Create the required ClusterRoles and ClusterRoleBindings so she gets access to the nodes.

kubectl create clusterrole michelle --verb=get,watch,list,create,delete --resource=nodes
clusterrole.rbac.authorization.k8s.io/michelle created

kubectl create clusterrolebinding michelle-binding --clusterrole=michelle --user=michelleclusterrolebinding.rbac.authorization.k8s.io/michelle-binding created


michelle's responsibilities are growing and now she will be responsible for storage as well. Create the required ClusterRoles and ClusterRoleBindings to allow her access to Storage.



Get the API groups and resource names from command kubectl api-resources. Use the given spec:

ClusterRole: storage-admin
Resource: persistentvolumes
Resource: storageclasses
ClusterRoleBinding: michelle-storage-admin
ClusterRoleBinding Subject: michelle
ClusterRoleBinding Role: storage-admin

kubectl auth can-i list storageclasses --as michelle


kubectl create clusterrole storage-admin --verb=get,watch,list,create,delete --resource=storageclasses
clusterrole.rbac.authorization.k8s.io/storage-admin created

controlplane ~ ➜  kubectl create clusterrolebinding michelle-storage-binding --clusterrole=storage-admin --user=michelle
clusterrolebinding.rbac.authorization.k8s.io/michelle-storage-binding created

kubectl auth can-i list storageclasses --as michelle
Warning: resource 'storageclasses' is not namespace scoped in group 'storage.k8s.io'
yes


## Service Accounts
How many Service Accounts exist in the default namespace?

kubectl get serviceaccounts
NAME      SECRETS   AGE
default   1         17m

What is the secret token used by the default service account?

kubectl describe serviceaccount default
Name:                default
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   default-token-snlkt
Tokens:              default-token-snlkt

nspect the deployment. What is the image used by the deployment?

kubectl describe deployment web-dashboard | grep Image
    Image:      gcr.io/kodekloud/customimage/my-kubernetes-dashboard

Inspect the Dashboard Application POD and identify the Service Account mounted on it.

At what location is the ServiceAccount credentials available within the pod?
ubectl describe pod web-dashboard-5899cf7849-hphq2 | grep account
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-snlkt (ro)

The application needs a ServiceAccount with the Right permissions to be created to authenticate to Kubernetes. The 'default' ServiceAccount has limited access. Create a new ServiceAccount named 'dashboard-sa'.


kubectl create serviceaccount dashboard-sa
serviceaccount/dashboard-sa created

Enter the access token in the UI of the dashboard application. Click Load Dashboard button to load Dashboard



Retrieve the Authorization token for the newly created service account , copy it and paste it into the token field of the UI.

To do this, run kubectl describe against the secret created for the dashboard-sa service account, copy the token and paste it in the UI.

kubectl describe serviceaccount dashboard-sa
Name:                dashboard-sa
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   dashboard-sa-token-wjm69
Tokens:              dashboard-sa-token-wjm69
Events:              <none>
root@controlplane:/# kubectl describe secret dashboard-sa-token-wjm69
Name:         dashboard-sa-token-wjm69
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: dashboard-sa
              kubernetes.io/service-account.uid: 51fb81b5-3779-400b-8a3d-58308d1dfb5a

Type:  kubernetes.io/service-account-token

Data
====
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6Inp3WHhoZFp6alJSMlJJNGFrQlg1azE4RmpXUW9xSmlBa1pYd0xrTnRLYlUifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRhc2hib2FyZC1zYS10b2tlbi13am02OSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJkYXNoYm9hcmQtc2EiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI1MWZiODFiNS0zNzc5LTQwMGItOGEzZC01ODMwOGQxZGZiNWEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpkYXNoYm9hcmQtc2EifQ.OlrmT_7QIT1n08XjSuf9aMvlKGjoDfgIrAn0gJPSphylwQZtN9C723G6i_PQI0cuziI9j8mKwkO8lR3uh0QLIZhyfOGFcp1aP2u_xIcucVTGty9SdIdvVJhNUTEHAO4c72Q6AiQFIJzfzvV5Z3dkV9M_PbyHNTRNYptFPeedueJOYkCp6wYwrAERva7OzsFBT34u-q5eD5_2N8hq1tFjPR0TDz3TmG5JtFfOYDsIIIx7CiD6GCF9Dypu8nyRfNx7v9gg-8r_iK5PyrM03u7CmdMvSm-jPdlXm4dsv8wemxqtdhBbqDMFaGmgZsAILOD2R3fZwBlAIoBthhDJKGVr1w
ca.crt:     1066 bytes
namespace:  7 bytes

You shouldn't have to copy and paste the token each time. The Dashboard application is programmed to read token from the secret mount location. However currently, the 'default' service account is mounted. Update the deployment to use the newly created ServiceAccount



Edit the deployment to change ServiceAccount from 'default' to 'dashboard-sa'


Use the serviceAccountName field inside the pod spec.

Use following YAML file:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-dashboard
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      name: web-dashboard
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        name: web-dashboard
    spec:
      serviceAccountName: dashboard-sa
      containers:
      - image: gcr.io/kodekloud/customimage/my-kubernetes-dashboard
        imagePullPolicy: Always
        name: web-dashboard
        ports:
        - containerPort: 8080
          protocol: TCP                          

Refresh the Dashboard application UI and you should now see the PODs listed automatically.



This time you shouldn't have to put in the token manually.




## Image Security
What is the secret type we choose for the docker registry?

kubectl create secret --help

We have an application running on our cluster. Let us explore it first. What image is the application using?


kubectl get pods
NAME                  READY   STATUS    RESTARTS   AGE
web-bd975bd87-4gpn6   1/1     Running   0          74s
web-bd975bd87-zqmx2   1/1     Running   0          74s
root@controlplane:/# kubectl describe pod web-bd975bd87-zqmx2 | grep Image
    Image:          nginx:alpine
    Image ID:       docker-pullable://nginx@sha256:44e208ac2000daeff77c27a409d1794d6bbdf52067de627c2da13e36c7d59582
root@controlplane:/# 

We decided to use a modified version of the application from an internal private registry. Update the image of the deployment to use a new image from myprivateregistry.com:5000



The registry is located at myprivateregistry.com:5000. Don't worry about the credentials for now. We will configure them in the upcoming steps.


kubectl edit deployment web

Create a secret object with the credentials required to access the registry.



Name: private-reg-cred
Username: dock_user
Password: dock_password
Server: myprivateregistry.com:5000
Email: dock_user@myprivateregistry.com


kubectl create secret docker-registry private-reg-cred --docker-username=dock_user --docker-password=dock_password --docker-server=myprivateregistry.com:5000 --docker-email=dock_user@myprivateregistry.com
secret/private-reg-cred created


Configure the deployment to use credentials from the new secret to pull images from the private registry

spec:
  containers:
    - name: foo
      image: janedoe/awesomeapp:v1
  imagePullSecrets:
    - name: myregistrykey

## Security Contexts

What is the user used to execute the sleep process within the ubuntu-sleeper pod?



In the current(default) namespace.
kubectl exec ubuntu-sleeper -- whoami
root


kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
ubuntu-sleeper   1/1     Running   0          115s

controlplane ~ ➜  kubectl get pods ubuntu-sleeper -o yaml > ubuntu-sleeper.yaml

controlplane ~ ➜  kubectl delete pods ubuntu-sleeper
pod "ubuntu-sleeper" deleted



A Pod definition file named multi-pod.yaml is given. With what user are the processes in the web container started?



The pod is created with multiple containers and security contexts defined at the Pod and Container level.

cat multi-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  securityContext:
    runAsUser: 1001
  containers:
  -  image: ubuntu
     name: web
     command: ["sleep", "5000"]
     securityContext:
      runAsUser: 1002

  -  image: ubuntu
     name: sidecar
     command: ["sleep", "5000"]

Update pod ubuntu-sleeper to run as Root user and with the SYS_TIME capability.



Note: Only make the necessary changes. Do not modify the name of the pod.

---
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu-sleeper
    securityContext:
      capabilities:
        add: ["SYS_TIME"]

Now update the pod to also make use of the NET_ADMIN capability.

---
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu-sleeper
    securityContext:
      capabilities:
        add: ["SYS_TIME", "NET_ADMIN"]



## Network Policy
How many network policies do you see in the environment?



We have deployed few web applications, services and network policies. Inspect the environment.


Run the command: kubectl get networkpolicy or kubectl get netpol


Which pod is the Network Policy applied on?

Run the command: kubectl get networkpolicy and look under the Pod Selector column.
After getting the labels, identify the correct pod name by their labels.
Run the command: kubectl get po --show-labels | grep name=payroll

What type of traffic is this Network Policy configured to handle?

kubectl describe networkpolicy payroll-policy
Name:         payroll-policy
Namespace:    default
Created on:   2022-04-01 19:46:18 +0000 UTC
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     name=payroll
  Allowing ingress traffic:
    To Port: 8080/TCP
    From:
      PodSelector: name=internal
  Not affecting egress traffic
  Policy Types: Ingress


Create a network policy to allow traffic from the Internal application only to the payroll-service and db-service.



Use the spec given on the below. You might want to enable ingress traffic to the pod to test your rules in the UI.


* Policy Name: internal-policy
* Policy Type: Egress
* Egress Allow: payroll
* Payroll Port: 8080
* Egress Allow: mysql
* MySQL Port: 3306



## Developing network policies


## Solution - Network Policies (optional)
