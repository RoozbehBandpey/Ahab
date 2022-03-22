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