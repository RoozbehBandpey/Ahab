# CoreDNS Lab

Identify the DNS solution implemented in this cluster.
```bash
kubectl get pods -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
coredns-74ff55c5b-pszbl                1/1     Running   0          9m24s
coredns-74ff55c5b-r5djr                1/1     Running   0          9m24s
etcd-controlplane                      1/1     Running   0          9m34s
kube-apiserver-controlplane            1/1     Running   0          9m34s
kube-controller-manager-controlplane   1/1     Running   0          9m34s
kube-flannel-ds-l4wj4                  1/1     Running   0          9m25s
kube-proxy-q947n                       1/1     Running   0          9m25s
kube-scheduler-controlplane            1/1     Running   0          9m34s
```
How many pods of the DNS server are deployed?
```bash
kubectl get deployments -n kube-system
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
coredns   2/2     2            2           10m
```
What is the name of the service created for accessing CoreDNS?
```bash
kubectl get service -n kube-system
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   9m28s
```
What is the IP of the CoreDNS server that should be configured on PODs to resolve services?
```bash
kubectl get service -n kube-system
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   9m28s
```
Where is the configuration file located for configuring the CoreDNS service?
```bash
kubectl describe deployments coredns -n kube-system | grep -A2 Args
    Args:
      -conf
      /etc/coredns/Corefile
```
How is the Corefile passed in to the CoreDNS POD?
```bash
kubectl get cm -n kube-system
NAME                                 DATA   AGE
coredns                              1      16m
```
What is the root domain/zone configured for this kubernetes cluster?
```bash
kubectl describe cm coredns -n kube-system
Name:         coredns
Namespace:    kube-system
Labels:       <none>
Annotations:  <none>

Data
====
Corefile:
----
.:53 {
    errors
    health {
       lameduck 5s
    }
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }
    prometheus :9153
    forward . /etc/resolv.conf {
       max_concurrent 1000
    }
    cache 30
    loop
    reload
    loadbalance
}

Events:  <none>
```
We just deployed a web server - webapp - that accesses a database mysql - server. However the web server is failing to connect to the database server. Troubleshoot and fix the issue.



They could be in different namespaces. First locate the applications. The web server interface can be seen by clicking the tab Web Server at the top of your terminal.
```bash
kubectl get po,svc -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
default       pod/hr                                     1/1     Running   0          15m
default       pod/simple-webapp-1                        1/1     Running   0          15m
default       pod/simple-webapp-122                      1/1     Running   0          15m
default       pod/test                                   1/1     Running   0          15m
default       pod/webapp-84ffb6ddff-vhsn6                1/1     Running   0          73s
kube-system   pod/coredns-74ff55c5b-pszbl                1/1     Running   0          23m
kube-system   pod/coredns-74ff55c5b-r5djr                1/1     Running   0          23m
kube-system   pod/etcd-controlplane                      1/1     Running   0          24m
kube-system   pod/kube-apiserver-controlplane            1/1     Running   0          24m
kube-system   pod/kube-controller-manager-controlplane   1/1     Running   0          24m
kube-system   pod/kube-flannel-ds-l4wj4                  1/1     Running   0          23m
kube-system   pod/kube-proxy-q947n                       1/1     Running   0          23m
kube-system   pod/kube-scheduler-controlplane            1/1     Running   0          24m
payroll       pod/mysql                                  1/1     Running   0          73s
payroll       pod/web                                    1/1     Running   0          15m

NAMESPACE     NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP                  24m
default       service/test-service     NodePort    10.97.192.9      <none>        80:30080/TCP             15m
default       service/web-service      ClusterIP   10.103.237.22    <none>        80/TCP                   15m
default       service/webapp-service   NodePort    10.107.211.191   <none>        8080:30082/TCP           73s
kube-system   service/kube-dns         ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP   24m
payroll       service/mysql            ClusterIP   10.99.126.201    <none>        3306/TCP                 73s
payroll       service/web-service      ClusterIP   10.97.226.230    <none>        80/TCP                   15m
```
>Run the command: `kubectl edit deploy webapp` and correct the `DB_Host` value.


From the hr pod nslookup the mysql service and redirect the output to a file `/root/CKA/nslookup.out`


```bash
kubectl exec -it hr -- nslookup mysql.payroll > /root/CKA/nslookup.out
root@controlplane:~# cat /root/CKA/nslookup.out
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   mysql.payroll.svc.cluster.local
Address: 10.99.126.201
```