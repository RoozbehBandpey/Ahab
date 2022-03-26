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
The primary purpose of node affinity features are to ensure pods are hosted on particular nodes. The same pod as before with node affinity will look like the following although both will do exact same thing.

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
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values:
              - Large
```

The node affinity ensure that the pod is place on a node that its size label value is in the given values. If we want to place the pod in  the node that is large or medium we have to  specify the following:

```yml
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values:
              - Large
              - Medium
```
For placing the pod on any node that are not small, we can use `NotIn` operator

```yml
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: NotIn
            values:
              - Small
```


The `Exists` operator will simply check if label exists on the node, and we don't need the value section of it.

What if there are no node with the label called `size`, or what if  someone changes the labels at some point in time. All of this addressed by type of node affinity, in our case `requiredDuringSchedulingIgnoredDuringExecution` 

The type of node affinity will define the behavior of the scheduler with respect to the affinity, and the stages in the lifecycle of the pod

* `requiredDuringSchedulingIgnoredDuringExecution` 
* `preferredDuringSchedulingIgnoredDuringExecution` 
* `requiredDuringSchedulingRequiredDuringExecution` 

There are two sates in the lifecycle of the pods when it comes to node affinity

||DuringScheduling|DuringExecution|
|--------------|--------------|--------------|
|Type 1|required|ignored|
|Type 2|preferred|ignored|
|Type 3|required|required|


DuringScheduling: is when the pod does not exist and it's created for the first time, in this  case the affinity rules will place the pods on the right nodes

If the nodes with matching labels are not available, or for example we forgot to label nodes as `Large` in that case the type of node affinity rules comes into play
* required: The scheduler will mandate  that the pod with the given affinity rules will be placed on the node, if it cannot find one, the pod will not be scheduled. This type will be used where the placement of the pods are crucial.
* preferred: If the pod placement is less important than running the workload itself, if the matching node is not found the scheduler will ignore node affinity rules, and place the pod on any available nodes. This is a way to tell scheduler to try your best to place the pod on any matching node, ig tou cannot find one just  place it anywhere.

DuringExecution: is when the pod has been running and a change has been made in the environment that effects node affinity, such as changes in the label. For example an administrator removed the `Large` label

* ignored: means pods will continue to run, and any changes in node affinity will not impact them once they are scheduled
* required: Will evict any pod that are running on the node that do not meet affinity rules.


## Taints & Tolerations vs Node Affinity

Imagine we have three nodes (blue, green and red) and three pods with same colors. The ultimate aim is to place the pods on the nodes with the correct color. And we are sharing the kubernetes cluster with other teams so we don not want to place pods on other team's node as well as preventing them to place pods on our nodes. 

Using taints and tolerations we apply taints on our nodes using their color. The we set tolerations on the pods to tolerate the respective colors. Now when the pods are created the nodes will ensure that they will accept the pods with the tolerations. But taints and tolerations does not guarantee the the pods will end up in the tainted nodes, so they can actually end up on other teams pods if their nodes are not tainted.

With node affinity we first label the nodes with their respective color, The we set node selectors on the pods to tie the node to the pods. however this does not guarantee that other teams pods does not end up on our nodes. 

For this a combination of taints and tolerations with node affinity need to be used. We first use taints and tolerations to prevent other pods on our nodes, then we use node affinity to prevent our pods to be placed on other nodes. 

## Resource Requirements and Limits

Imagine a three node kubernetes cluster, each node has set of CPU, memory and disk resources available. Every pod consumes set of resources, whenever a pod is placed on a node, it consumes resources available to that node, it is the kubernetes scheduler that decides which node a pod goes to. The scheduler take into consideration the amount of resources  required by a pod and those available on the nodes. If the node has no sufficient resources, the scheduler avoids placing the pod on that node, instead places the pod on a node where sufficient resources are available. If there's nop sufficient resources available on any node, kubernetes holds back the scheduling of the pod, we will see the pod on the `pending` state. If you look up the events you'll see the reason `Insufficient cpu`. 

By default kubernetes assumes that a pod (or a container within the pod), requires `0.5 cpu` and `256 Mi` of memory, this is known as the **resource request** for a container. When the scheduler tries to place the pod on the node it uses these number to identify the node which has sufficient amount of resources available. 

If the application needs more than these we can specify the amount of resources in pod or deployment definition file:

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
    resources:
      requests:
        memory: "1Gi"
        cpu: 1
```
In this case the pod is going to request 1 Gibibyte of memory and 1 count of vcpu. 

For cpu we can specify any value as low as `0.1`, this can be also expressed as `100m` where `m` stands for mili. We can go as low as `1m` but not lower than that. 

1 count of cpu is equal to `1 AWS vCPU`, `1 GCP Core`, `1 Azure Core` and  `1 Hyperthread` 


For memory note the difference between `G` and `Gi`

* `1 G (Gigabyte) = 1,000,000,000 bytes`
* `1 M (Megabyte) = 1,000,000 bytes`
* `1 K (Kilobytes) = 1,000 bytes`
* `1 Gi (Gibibyte) = 1,073,741,828 bytes`
* `1 Mi (Mebibyte) = 1,048,576 bytes`
* `1 Ki (Kibibyte) = 1,024 bytes`

In the docker world a docker container has no limit on the resources it can consume on a node, let's say a container can start with `1 vCPU` on a node it can go up and consume as much as resources it requires, suffocating the native processes on the node or other containers.

However we can set a resource usage limit on the pods by default kubernetes set the limit of `1 vCPU` and `512 Mi` to the containers. This can be changed with specifying the limits:

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
    resources:
      requests:
        memory: "1Gi"
        cpu: 1
      limits:
        memory: "2Gi"
        cpu: 2
```

> The limits and requests are set for each container within the pod

If a pod tries to exceed resources beyond its specified limit, in case of cpu, kubernetes throttles the cpu. But for the memory a container can use more memory than its specified limits, if a pod tries to consume more memory than its limit constantly the pod will be terminated.


