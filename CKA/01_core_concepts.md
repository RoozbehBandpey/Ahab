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