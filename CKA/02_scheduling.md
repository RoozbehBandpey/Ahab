# Kubernetes Scheduling

## Manual Scheduling

To understand how scheduling works we have to see how we would implement scheduling ourselves. In pod definition file, every pod has a field called `nodeName` that by default it is not set. Tou don't typically set this property when you create a pod, the scheduler goes through pods that does not hve this property set, those become the candidates for scheduling. Once it found it will set the property to the node by creating a binding object. If there's no scheduler the pods will be in  the  pending stat, in this case we can manually assign pods to nodes, the easiest way is to set the name of the node in the pod specification file, while creating a pod. 

```yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    tier: front-end
spec:
  containers:
  - name: nginx-container
    image: nginx
    ports:
    - containerPort: 8080
  nodeName:
```

This can only be done at creation time, kubernetes won't allow to assign pods to nodes, if he pod already exists, we have to create a binding object and send a post request to the pod's binding api. In the binding object we specify a target node with the name of the node, then we send a  post request to the pod's binding api with  the data set to binding object in the json format, so we must  convert the yaml into it equivalent json format

```yml
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion:
  kind: Node
  name: node02
```

```bash
curl --header "Content-Type:application/json" --request POST --data '{"apiVersion": "V1",  "kind": "Binding",...}' http://$SERVER/api/v1/namespaces/default/pods/$PODNAME/binding
```

## Labels & Selectors

Labels are a way to attach properties to objects and selectors are a way to filter based on those labels. We can group objects by their type, application or functionality. We will attach labels to each object and while selecting we'll specify a condition to narrow down the objects.

Once the pod is created, to select the pod with labels use the following parameter:

```bash
kubectl get pods --selector app=App1
```

Kubernetes cluster also uses labels and selectors to connect different objects together. For example for creating a replicaset consisting of three different pods, we first label the pod definition and then use selector in replicaset to group the pods. In a replicaset definition file we'll see labels defined in two places, the labels defined under template section are the labels configured on the pod, the labels at the top are the labels of replicaset itself. The labels of replicaset are used if we were to configure another object that  is suppose  to discover replicaset itself. For connecting the replicaset to the pods we will use the selector  field to  match the labels of the pods.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs
  labels:
    app: myapp
    tier: front-end
  annotations:
    buildVersion: 1.34
spec:
  template:
    metadata:
    name: myapp-pod
    labels:
      app: myapp
      tier: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx
  replicas: 3
  selector:
    matchLabels:
      tier: front-end
```

### Annotations

Annotations are used to record other information about the object, for example build version, name, contact details, phone numbers, emails etc.,

## Taints & Tolerations

Taints & tolerations are used to set restrictions on what pods can be scheduled on what nodes. When the pods are created kubernetes scheduler tries to place the pods in the available worker nodes. In the beginning there's no restriction and kubernetes scheduler tries to balance the pods equally on nodes. Now let's assume we have a particular application that we want the pods to go only to a specific node.  We can prevent pods on going to specific node by  placing taint on the node. By default pods have no toleration, meaning unless otherwise specified none of the pods can tolerate any taint. For the pods that we want them to go on the node we must specify toleration of the taint. 

To taint a node:

```bash
kubectl taint nodes <node name> key=value:taint-effect
```
The taint is in form of key value pairs, the taint-effect specifies the situation which the pods do not tolerate this taint. There are three tain-effects:
* `NoSchedule` Which means the pod will not be scheduled on the node
* `PreferNoSchedule` The system will try  to avoid placing a pod on the node, but that is not guaranteed
* `NoExecute` New pods will not be scheduled on the node and existing pods on the node (if any) will be evicted if they do not tolerate the taint.


```bash
kubectl taint nodes node1 app=blue:NoSchedule
```

To add a toleration to a pod add the following in the manifest:
All if these values need to be in double quotes

```yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    tier: front-end
spec:
  containers:
  - name: nginx-container
    image: nginx
  tolerations:
  - key: "app"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
```

When the pods are now created or updated with new tolerations, they  are  either not scheduled on the nodes or evicted from existing node depending on the effect.

In case of `NoExecute` if we have three nodes with pods runnin on them, if we decide at some point that we want to dedicate one node to a specific kind of application after we apply taint on the node, the moment the taint takes effect, it'll keep whatever pods that have the toleration and evict  the pods that do not have the tolertions, What happens to evicted pods?

The scheduler does not place any pod on master node, when the kubernetes cluster is first set up, it taint the master node that prevent any pod to be scheduled on master node. To see this taint run the following:

```bash
kubectl describe node kubemaster | grep Taint
```

To look up all the options available run the following:
```bash
kubectl explain pod --recursive | less
```

## Node Selectors


Imagine we have a three node cluster, one of them is a larger node with higher resources available. If we have a process like data processing that require higher resources we would like to schedule it on the bigger node so in case the process need to scale there would be enough resources available. In the default setup any pod can go to any node. To solve this we can set limitation on the pod so it can only run on specific nodes.

There are two ways to do this:

### 1. Node Selectors
Under `nodeSelector` we have to specify the size as large. The key value pair of `size: Large` are in fact labels assigned to nodes. The scheduler use this labels to match and identify nodes to assign pods on. The nodes must be labeled prior to creating these pods. 


```yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
    tier: front-end
spec:
  containers:
  - name: nginx-container
    image: nginx
  nodeSelector:
    size: Large
```

To label a node run the following command:
```bash
kubectl label node <node-name> <label-key>=<label-value>
```
For example
```bash
kubectl label node node-1 size=Large
```

Node selector has limitations, we use a single label and selector to achieve our goal here. But if the requirement is much  more complex, for instance if  we want to  place a pod on a large or medium node. Or place the pod on any node that are not small, this cannot be achieved with node labels and selectors. For this  we have to use node affinity and anti affinity rules.

## Node Affinity