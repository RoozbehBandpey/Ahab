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

## Network Troubleshooting
### Network Plugin in kubernetes

Kubernetes uses CNI plugins to setup network. The kubelet is responsible for executing plugins as we mention the following parameters in kubelet configuration.

* `cni-bin-dir`: Kubelet probes this directory for plugins on startup

* `network-plugin`: The network plugin to used from `cni-bin-dir`. It must match the name reported by a plugin probed from the plugin directory.



There are several plugins available and these are some.



1. Weave Net:
    This is the only plugin mentioned in the kubernetes documentation. To install,
    `kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"`
    You can find this in following documentation :
    https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/

2. Flanne:
    To install,
    `kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml`
    >Note: As of now flannel does not support kubernetes network policies

3. Calico:
    To install
    `curl https://docs.projectcalico.org/manifests/calico.yaml -O`
    Apply the manifest using the following command
    `kubectl apply -f calico.yaml`
    >Calico is said to have most advanced cni network plugin.


In CKA and CKAD exam, you won't be asked to install the cni plugin. But if asked you will be provided with the exact url to install it. If not, you can install weave net from the documentation 


https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/



>Note: If there are multiple CNI configuration files in the directory, the kubelet uses the configuration file that comes first by name in lexicographic order.


### DNS in Kubernetes

Kubernetes uses CoreDNS. CoreDNS is a flexible, extensible DNS server that can serve as the Kubernetes cluster DNS.

#### Memory and Pods

In large scale Kubernetes clusters, CoreDNS's memory usage is predominantly affected by the number of Pods and Services in the cluster. Other factors include the size of the filled DNS answer cache, and the rate of queries received (QPS) per CoreDNS instance.

Kubernetes resources for coreDNS are:   

1. service account named coredns
1. cluster-roles named coredns and kube-dns
1. clusterrolebindings named coredns and kube-dns, 
1. a deployment named coredns,
1. a configmap named coredns and a
1. service named kube-dns.



While analyzing the coreDNS deployment you can see that the the Corefile plugin consists of important configuration which is defined as a configmap.



Port `53` is used for for DNS resolution.


```
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }
```

This is the backend to k8s for `cluster.local` and reverse domains.


```bash
proxy . /etc/resolv.conf
```


Forward out of cluster domains directly to right authoritative DNS server.

### Troubleshooting issues related to coreDNS
1. If you find CoreDNS pods in pending state first check network plugin is installed.

2. coredns pods have CrashLoopBackOff or Error state

    If you have nodes that are running SELinux with an older version of Docker you might experience a scenario where the coredns pods are not starting. To solve that you can try one of the following options:

        1. Upgrade to a newer version of Docker.
        2. Disable SELinux.
        3. Modify the coredns deployment to set allowPrivilegeEscalation to true:

```bash
kubectl -n kube-system get deployment coredns -o yaml | \
  sed 's/allowPrivilegeEscalation: false/allowPrivilegeEscalation: true/g' | \
  kubectl apply -f -
```
3. Another cause for CoreDNS to have CrashLoopBackOff is when a CoreDNS Pod deployed in Kubernetes detects a loop.

There are many ways to work around this issue, some are listed here:



* Add the following to your kubelet config yaml: `resolvConf: <path-to-your-real-resolv-conf-file>` This flag tells kubelet to pass an alternate `resolv.conf` to Pods. For systems using `systemd-resolved`, `/run/systemd/resolve/resolv.conf` is typically the location of the "real" resolv.conf, although this can be different depending on your distribution.

* Disable the local DNS cache on host nodes, and restore `/etc/resolv.conf` to the original.

* A quick fix is to edit your Corefile, replacing forward `./etc/resolv.conf` with the IP address of your upstream DNS, for example forward . 8.8.8.8. But this only fixes the issue for CoreDNS, kubelet will continue to forward the invalid resolv.conf to all default dnsPolicy Pods, leaving them unable to resolve DNS.



4. If CoreDNS pods and the kube-dns service is working fine, check the kube-dns service has valid endpoints.
```bash
kubectl -n kube-system get ep kube-dns
```
If there are no endpoints for the service, inspect the service and make sure it uses the correct selectors and ports.





### Kube-Proxy

kube-proxy is a network proxy that runs on each node in the cluster. kube-proxy maintains network rules on nodes. These network rules allow network communication to the Pods from network sessions inside or outside of the cluster.



In a cluster configured with kubeadm, you can find kube-proxy as a daemonset.



kubeproxy is responsible for watching services and endpoint associated with each service. When the client is going to connect to the service using the virtual IP the kubeproxy is responsible for sending traffic to actual pods.



If you run a `kubectl describe ds kube-proxy -n kube-system `you can see that the kube-proxy binary runs with following command inside the kube-proxy container.


```bash
    Command:
      /usr/local/bin/kube-proxy
      --config=/var/lib/kube-proxy/config.conf
      --hostname-override=$(NODE_NAME)
```

So it fetches the configuration from a configuration file ie, `/var/lib/kube-proxy/config.conf` and we can override the hostname with the node name of at which the pod is running.

 

In the config file we define the `clusterCIDR`, `kubeproxy mode`, `ipvs`, `iptables`, `bindaddress`, `kube-config` etc.

 

### Troubleshooting issues related to kube-proxy
1. Check kube-proxy pod in the kube-system namespace is running.

2. Check kube-proxy logs.

3. Check configmap is correctly defined and the config file for running kube-proxy binary is correct.

4. kube-config is defined in the config map.

5. check kube-proxy is running inside the container
```bash
# netstat -plan | grep kube-proxy
tcp        0      0 0.0.0.0:30081           0.0.0.0:*               LISTEN      1/kube-proxy
tcp        0      0 127.0.0.1:10249         0.0.0.0:*               LISTEN      1/kube-proxy
tcp        0      0 172.17.0.12:33706       172.17.0.12:6443        ESTABLISHED 1/kube-proxy
tcp6       0      0 :::10256                :::*                    LISTEN      1/kube-proxy
```
