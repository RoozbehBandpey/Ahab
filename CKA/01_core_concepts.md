# Kubernetes Core Concepts

## Cluster Architecture

The purpose of Kubernetes is to host application in form of containers, in an automated fashion so that you can easily deploy as many instance of the application as required and easily enable communication between different services. At high level Kubernetes consist of control-plane and data-plane (nodes). Control plane is responsible for monitoring and managing nodes while nodes are actually carrying the workload. Using ships analogy, nodes are cargo ships while control-plane is the master ship. 



![Kubernetes High Level Architecture](images/CKA-high-level.drawio.png)


The worker node on the cluster are like cargo ships that can load containers.
The control plane is responsible for loading the the containers on the cargo ships, it plans how to load, identify the right ship, store information about them, monitor and track the location of containers on the cargo ships etc., This relates to master node. The master node is responsible for managing kubernetes cluster. The master node does all of these with control-plane components: 

* ETCD Cluster: Is a highly available key-value store which maintain information about the containers
* Scheduler: Identifies the right node on kubernetes to place a container on. Based on containers resource requirement, worker capacity or any other policy or constrains
* Node-controller: Responsible for onboarding new nodes into the cluster handling situations when nodes become unavailable or gets destroyed 
* Replication controller: Ensures that desired containers are running at all time on the replication group 
* Kube API Server: Is primary management component of kubernetes which is responsible for orchestrating all operations within cluster it exposes kubernetes API which is used by external users to perform management operations on the cluster, it also monitors state of cluster and perform actions based on that 
* Container runtime engine: such as docker it is installed on the all nodes including the master nodes to be able to run containers 
* Kubelet: Is an agent that runs on each mode on a cluster, it listens for instructions from the kube-api server and deploys and destroys containers on the node, the kube-api server periodically fetches status report from kubelet to monitor the status of nodes and containers on them 
* Kube-proxy: Ensures that necessary rules are in place on the worker nodes to allow containers running on them to reach each other 


## ETCD

ETCD is a super fast and highly available key-value store. Whe you install and run it it will start a service that will listen on port 2379 by default you can the attach any client to ETCD to store and retrieve information. The default client that comes with it is ETCD control client, it is a CLI client for ETCD.

To store a key value pair run the following command which create an entry in the database with the given information:

```bash
./etcdctl set key1 value1
```
To retrive data run the following command, 
```bash
./etcdctl get key1
```

The ETCD stores information for Nodes, Pods, Configs, Secrets, Accounts, Roles, Role-bindings etc., Everything that comes from `kubectl get` is from ETCD cluster. Every change that we make to cluster such as adding addional nodes, deployment, pods and replicasets are updated in ETCD cluster. Only once it ios update in ETCD server then the change is considered to be complete. 

ETCD has different deployment mode depending how the cluster is set up:
Set up cluster from scratch: Then we deploy ETCD by downloading its binary, then configuring ETCD as a server in the master node.
For high availability we have to set up ETCD as a cluster 

Advertised client url: This is the address on which ETCD listens, it is on the IP of the server and on port 2379. This is the url that should be configured on the api server when it tries to reach ETCD server. 

Set up with kubeadm: In this deployment mode, kubeadm sets up ETCD as a pod in the kube-system namespace. We can explore ETCD database using ETCD control utility in this pod. 

To list all keys stored by kubernetes run the following command:

```bash
kubectl exec etcd-master -n kube-system etcdctl get / --prefix -keys-only
```

Kubernetes stores data in a specific directory structure
Root: is registry where we have various kubernetes constructs such as minions, pods, replicasets. deployments, roles, secrets, etc.,

```bash
/registry/apiregistration.k8s.io/apiservices/v1.
/registry/apiregistration.k8s.io/apiservices/v1.apps
/registry/apiregistration.k8s.io/apiservices/v1.authentications.k8s.io
...
```
In high availability environment we have multiple master nodes in cluster, then we have multiple ETCD instances across master nodes.

In this case we have to make sure ETCD instances know about eachother by setting up the right parameter in ETCD service configuration in `--initial-cluster controller-0-https://{CONTROLLER0_IP}:2380,controller-1-https://{CONTROLLER1_IP}:2380`


