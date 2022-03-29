# Security
Understanding kubernetes security primitives, how does someone gain access to the cluster. 


# Kubernetes Security Primitives
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

# Authentication

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
