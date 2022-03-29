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

# Article on Setting up Basic Authentication

# TLS Introduction

# TLS Basics

# TLS in Kubernetes

# TLS in Kubernetes - Certificate Creation

# View Certificate Details

# Resource: Download Kubernetes Certificate Health Check Spreadsheet


# Certificates API


# KubeConfig


# Persistent Key/Value Store

# API Groups

# Authorization

# Role Based Access Controls


# Cluster Roles and Role Bindings


# Service Accounts


# Image Security


# Security Contexts


# Network Policy

# Developing network policies


# Solution - Network Policies (optional)
