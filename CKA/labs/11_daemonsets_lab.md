# DaemonSets


How many DaemonSets are created in the cluster in all namespaces?


Check all namespaces
```bash
kubectl get daemonsets --all-namespaces
NAMESPACE     NAME              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   kube-flannel-ds   1         1         1       1            1           <none>                   11m
kube-system   kube-proxy        1         1         1       1            1           kubernetes.io/os=linux   11m
```
On how many nodes are the pods scheduled by the DaemonSet kube-proxy
```bash
kubectl describe daemonset kube-proxy --namespace=kube-system

Name:           kube-proxy
Selector:       k8s-app=kube-proxy
Node-Selector:  kubernetes.io/os=linux
Labels:         k8s-app=kube-proxy
Annotations:    deprecated.daemonset.template.generation: 1
Desired Number of Nodes Scheduled: 1
Current Number of Nodes Scheduled: 1
Number of Nodes Scheduled with Up-to-date Pods: 1
Number of Nodes Scheduled with Available Pods: 1
Number of Nodes Misscheduled: 0
...
```

What is the image used by the POD deployed by the kube-flannel-ds DaemonSet?
```bash
kubectl describe daemonset kube-flannel-ds --namespace=kube-system
Name:           kube-flannel-ds
Selector:       app=flannel
Node-Selector:  <none>
Labels:         app=flannel
                tier=node
Annotations:    deprecated.daemonset.template.generation: 1
Desired Number of Nodes Scheduled: 1
Current Number of Nodes Scheduled: 1
Number of Nodes Scheduled with Up-to-date Pods: 1
Number of Nodes Scheduled with Available Pods: 1
Number of Nodes Misscheduled: 0
Pods Status:  1 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:           app=flannel
                    tier=node
  Service Account:  flannel
  Init Containers:
   install-cni:
    Image:      quay.io/coreos/flannel:v0.13.1-rc1
...
```
Deploy a DaemonSet for FluentD Logging.



Use the given specifications.


Name: elasticsearch
Namespace: kube-system
Image: k8s.gcr.io/fluentd-elasticsearch:1.20


An easy way to create a DaemonSet is to first generate a YAML file for a Deployment with the command `kubectl create deployment elasticsearch --image=k8s.gcr.io/fluentd-elasticsearch:1.20 -n kube-system --dry-run=client -o yaml > fluentd.yaml`. Next, remove the replicas, strategy and status fields from the YAML file using a text editor. Also, change the kind from Deployment to DaemonSet.

Finally, `create the Daemonset by running kubectl create -f fluentd.yaml`

```bash
kubectl apply -f fluentd.yaml 
error: error validating "fluentd.yaml": error validating data: [ValidationError(DaemonSet.spec): unknown field "progressDeadlineSeconds" in io.k8s.api.apps.v1.DaemonSetSpec, ValidationError(DaemonSet.spec.selector): unknown field "type" in io.k8s.apimachinery.pkg.apis.meta.v1.LabelSelector]; if you choose to ignore these errors, turn validation off with --validate=false
```

Fox the fields and apply again