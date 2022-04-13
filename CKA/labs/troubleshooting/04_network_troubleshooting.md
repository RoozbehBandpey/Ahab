# Network Troubleshooting Lab

A simple 2 tier application is deployed in the triton namespace. It must display a green web page on success. Click on the app tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.



Stick to the given architecture. Use the same names and port numbers as given in the below architecture diagram. Feel free to edit, delete or recreate objects as necessary.
```bash
kubectl get all -n triton
NAME                                READY   STATUS              RESTARTS   AGE
pod/mysql                           0/1     ContainerCreating   0          107s
pod/webapp-mysql-54db464f4f-7vtd2   0/1     ContainerCreating   0          106s

NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/mysql         ClusterIP   10.110.122.255   <none>        3306/TCP         107s
service/web-service   NodePort    10.100.202.163   <none>        8080:30081/TCP   106s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp-mysql   0/1     1            0           106s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/webapp-mysql-54db464f4f   1         1         0       106s
```
```bash
kubectl describe pod mysql -n triton
Name:         mysql
...
  Type     Reason                  Age                  From               Message
  ----     ------                  ----                 ----               -------
  Normal   Scheduled               3m49s                default-scheduler  Successfully assigned triton/mysql to controlplane
  Warning  FailedCreatePodSandBox  3m47s                kubelet            Failed to create pod sandbox: rpc error: code = Unknown desc = [failed to set up sandbox container "b2ad5397b7141c4ff83143d0cabd17b8f12b9614e699f47e72ec822134c14d05" network for pod "mysql": networkPlugin cni failed to set up pod "mysql_triton" network: unable to allocate IP address: Post "http://127.0.0.1:6784/ip/b2ad5397b7141c4ff83143d0cabd17b8f12b9614e699f47e72ec822134c14d05": dial tcp 127.0.0.1:6784: connect: connection refused, failed to clean up sandbox container "b2ad5397b7141c4ff83143d0cabd17b8f12b9614e699f47e72ec822134c14d05" network for pod "mysql": networkPlugin cni failed to teardown pod "mysql_triton" network: Delete "http://127.0.0.1:6784/ip/b2ad5397b7141c4ff83143d0cabd17b8f12b9614e699f47e72ec822134c14d05": dial tcp 127.0.0.1:6784: connect: connection refused]
  Normal   SandboxChanged          9s (x17 over 3m46s)  kubelet            Pod sandbox changed, it will be killed and re-created.
```
```bash
kubectl get endpoints -n triton
NAME          ENDPOINTS   AGE
mysql         <none>      5m35s
web-service   <none>      5m34s
```
Does the cluster have a Network Addon installed?

```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```


The same 2 tier application is having issues again. It must display a green web page on success. Click on the app tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.



Stick to the given architecture. Use the same names and port numbers as given in the below architecture diagram. Feel free to edit, delete or recreate objects as necessary.

```bash
kubectl get pods -ANAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
kube-system   coredns-74ff55c5b-hwpkn                1/1     Running   0          63m
kube-system   coredns-74ff55c5b-q967h                1/1     Running   0          63m
kube-system   etcd-controlplane                      1/1     Running   0          63m
kube-system   kube-apiserver-controlplane            1/1     Running   0          63m
kube-system   kube-controller-manager-controlplane   1/1     Running   0          63m
kube-system   kube-proxy-v2z2d                       0/1     Error     3          61s
kube-system   kube-scheduler-controlplane            1/1     Running   0          63m
kube-system   weave-net-6p86z                        2/2     Running   0          3m55s
triton        mysql                                  1/1     Running   0          60s
triton        webapp-mysql-54db464f4f-p7nbf          1/1     Running   0          58s
```
```bash
kubectl logs kube-proxy-v2z2d -n kube-system
F0413 21:14:15.594099       1 server.go:490] failed complete: open /var/lib/kube-proxy/configuration.conf: no such file or directory
```
```bash
kubectl get configmaps -A
NAMESPACE         NAME                                 DATA   AGE
default           kube-root-ca.crt                     1      66m
kube-node-lease   kube-root-ca.crt                     1      66m
kube-public       cluster-info                         2      66m
kube-public       kube-root-ca.crt                     1      66m
kube-system       coredns                              1      66m
kube-system       extension-apiserver-authentication   6      66m
kube-system       kube-proxy                           2      66m
kube-system       kube-root-ca.crt                     1      66m
kube-system       kubeadm-config                       2      66m
kube-system       kubelet-config-1.20                  1      66m
kube-system       weave-net                            0      66m
triton            kube-root-ca.crt                     1      17m
```
```bash
kubectl describe configmap kube-proxy -n kube-system | grep .conf
Annotations:  kubeadm.kubernetes.io/component-config.hash: sha256:3c1c57a3d54be93d94daaa656d9428e3fbaf8d0cbb4d2221339c2d450391ee35
config.conf:
apiVersion: kubeproxy.config.k8s.io/v1alpha1
  kubeconfig: /var/lib/kube-proxy/kubeconfig.conf
kubeconfig.conf:
```
```bash
kubectl describe daemonset kube-proxy -n kube-system
```
```bash
kubectl edit daemonset kube-proxy -n kube-system
```
