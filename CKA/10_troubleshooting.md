# Troubleshooting

Let's look at a two pod application a web and a database server. The database pod hosts a database application and hosts the web server through a database service. 

Users report some issue with accessing the application, first we check if the web application is accessible via the IP of the node port

```bash
curl http://web-service-ip:node-port
```

Next we check the web application's service, did it discovered the endpoint for the web pod?

```bash
kubectl describe service web-service
```
If it did not, compare the selectors on the service with the ones on the pod

Next we check the pod itself and make sure it is in the running state. The state of the pod as well as the number of restarts can give and idea if application is running or getting restarted.

```bash
kubectl get pods
```
Check the events related to the pod using the describe command

```bash
kubectl describe pod <pod-name>
```

We can get the logs with
```bash
kubectl logs <object-name>
```

If the pods are restarting due to a failure then the logs in the current version of the pod may not reflect why it failed the last time. So we either have to view these logs with `-f` option to wait for pods to wait, or with `--previous` option to view the logs of the previous version of the pods.

The we go deeper to check the status of the db service and db pod itself

## Control plane failures

Start by checking the status of the nodes in  the cluster

```bash
kubectl get nodes
```
Then check the status of the pods running on the cluster
```bash
kubectl get pods
```

If the kubeadm was used then we have control plane components as pods in kube-system namespace

```bash
kubectl get pods -n kube-system
```
If control plane components were deployed as services, then check the status of kubernetes services:
On master node:
```bash
service kube-apiserver status
```
```bash
service kube-controller-manager status
```
```bash
service kube-scheduler status
```
and on worker nodes
```bash
service kubelet status
```
```bash
service kube-proxy status
```

Next check the logs of control plane components, in case of kubeadm

```bash
kubectl logs kube-apiserver-master -n kube-system
```
In case of services, view the service logs using the host's logging solution:

```bash
sudo journalctl -u kube-apiserver
```

## Worker Node Failure

Start by checking the state of the nodes in the cluster

```bash
kubectl get nodes
```

If the nodes are reported as not ready, check the details of the nodes using

```bash
kubectl describe node <node name>
```

Check for possible CPu, memory and disk space on the nodes

```bash
top
```
```bash
df -h
```

Check the status  of the kubelet

```bash
service kubelet status
```

Check kubelet logs for possible issues
```bash
sudo journalctl -u kubelet
```

Check kubelet certificates, make sure they are not expired and they are part of a right group

```bash
openssl x509 -in /var/lib/kubelet/worker-1.crt -text 
```

Make sure that certificates are issued by the right CA