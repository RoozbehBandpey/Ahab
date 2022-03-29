# Security
Understanding kubernetes security primitives, how does someone gain access to the cluster. 


## Kubernetes Security Primitives
All access to host must be secured, root access disabled, password based authentication disabled, only SSH key authentication to be made available. 

Our focus here is on kubernetes related security, what are the risks and what  are the measures  to be taken to secure the cluster.

The first line of defence is kube-apiserver, as we do everything on the cluster via kube-apiserver. We need to make two type of decision 1. Who can access the cluster? 2. What can they do?

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

It generates two files, `id_rsa` is the private key and `id_rsa.pub` is the public key. Then we secure the server by locking down all access to it, except through a door that is locked using public key. It is usually done by adding an entry with your public key in the server's authorized keys file `cat ./.ssh/authorized_keys` The lock is public and anyone can attempt to break through. When we try to ssh, we specify the location of our private key in our command. For multiple servers we can make a copies of  the public key and place then on as many servers as possible. 
## TLS in Kubernetes

## TLS in Kubernetes - Certificate Creation

## View Certificate Details

## Resource: Download Kubernetes Certificate Health Check Spreadsheet


## Certificates API


## KubeConfig


## Persistent Key/Value Store

## API Groups

## Authorization

## Role Based Access Controls


## Cluster Roles and Role Bindings


## Service Accounts


## Image Security


## Security Contexts


## Network Policy

## Developing network policies


## Solution - Network Policies (optional)
