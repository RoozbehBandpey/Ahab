# Static Pods Lab

How many static pods exist in this cluster in all namespaces?

```bash
kubectl get pods --all-namespaces -o wide
NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE     IP             NODE           NOMINATED NODE   READINESS GATES
kube-system   coredns-74ff55c5b-88ll5                1/1     Running   0          7m39s   10.244.0.3     controlplane   <none>           <none>
kube-system   coredns-74ff55c5b-w6pc7                1/1     Running   0          7m39s   10.244.0.2     controlplane   <none>           <none>
kube-system   etcd-controlplane                      1/1     Running   0          7m47s   10.34.226.12   controlplane   <none>           <none>
kube-system   kube-apiserver-controlplane            1/1     Running   0          7m47s   10.34.226.12   controlplane   <none>           <none>
kube-system   kube-controller-manager-controlplane   1/1     Running   0          7m47s   10.34.226.12   controlplane   <none>           <none>
kube-system   kube-flannel-ds-dnw7b                  1/1     Running   0          7m22s   10.34.226.3    node01         <none>           <none>
kube-system   kube-flannel-ds-xg8cv                  1/1     Running   0          7m40s   10.34.226.12   controlplane   <none>           <none>
kube-system   kube-proxy-l4h2b                       1/1     Running   0          7m40s   10.34.226.12   controlplane   <none>           <none>
kube-system   kube-proxy-rzx68                       1/1     Running   0          7m22s   10.34.226.3    node01         <none>           <none>
kube-system   kube-scheduler-controlplane            1/1     Running   0          7m47s   10.34.226.12   controlplane   <none>           <none>
```

> The static pods have node name as suffix


What is the path of the directory holding the static pod definition files?


Run the command ps -aux | grep kubelet and identify the config file - --config=/var/lib/kubelet/config.yaml. Then check in the config file for staticPodPath.
```bash
ps -aux | grep kubelet
root      4021  0.0  0.1 1104964 358288 ?      Ssl  17:54   0:51 kube-apiserver --advertise-address=10.34.226.12 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestriction --enable-bootstrap-token-auth=true --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key --etcd-servers=https://127.0.0.1:2379 --insecure-port=0 --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key --requestheader-allowed-names=front-proxy-client --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt --requestheader-extra-headers-prefix=X-Remote-Extra- --requestheader-group-headers=X-Remote-Group --requestheader-username-headers=X-Remote-User --secure-port=6443 --service-account-issuer=https://kubernetes.default.svc.cluster.local --service-account-key-file=/etc/kubernetes/pki/sa.pub --service-account-signing-key-file=/etc/kubernetes/pki/sa.key --service-cluster-ip-range=10.96.0.0/12 --tls-cert-file=/etc/kubernetes/pki/apiserver.crt --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
root      4657  0.0  0.0 4077336 106512 ?      Ssl  17:54   0:22 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.2
root     12827  0.0  0.0  11468  1052 pts/0    R+   18:05   0:00 grep --color=auto kubelet
```
```bash
cat /var/lib/kubelet/config.yaml

apiVersion: kubelet.config.k8s.io/v1beta1
...
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 0s
...
```

Or

```
grep -i static /var/lib/kubelet/config.yaml
```

How many pod definition files are present in the manifests folder?
```bash
cd /etc/kubernetes/manifests
root@controlplane:/etc/kubernetes/manifests# ls
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
```
What is the docker image used to deploy the kube-api server as a static pod?

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```
or 

```bash
grep -i image /etc/kubernetes/manifests/kube-apiserver.yaml
```
Create a static pod named static-busybox that uses the busybox image and the command sleep 1000
```bash
kubectl run static-busybox --image=busybox --command sleep 1000 --restart=Never --dry-run=client -o yaml > /etc/kubernetes/manifests/busybox.yaml

kubectl get pods
NAME                          READY   STATUS      RESTARTS   AGE
static-busybox-controlplane   0/1     Completed   0          7s
```

We just created a new static pod named static-greenbox. Find it and delete it.


> Identify which node the static pod is created on, ssh to the node and delete the pod definition file. If you don't know the IP of the node, run the `kubectl get nodes -o wide` command and identify the IP. Then, SSH to the node using that IP. For static pod manifest path look at the file `/var/lib/kubelet/config.yaml` on `node01`
```bash
kubectl get nodes -o wide
NAME           STATUS   ROLES                  AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
controlplane   Ready    control-plane,master   28m   v1.20.0   10.34.226.12   <none>        Ubuntu 18.04.5 LTS   5.4.0-1068-gcp   docker://19.3.0
node01         Ready    <none>                 28m   v1.20.0   10.34.226.3    <none>        Ubuntu 18.04.5 LTS   5.4.0-1068-gcp   docker://19.3.0

ssh 10.34.226.3
The authenticity of host '10.34.226.3 (10.34.226.3)' can't be established.
ECDSA key fingerprint is SHA256:u9OdNaCmrfntgjseysL28yF4sjvpXR6aev+/cLrHsvU.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.34.226.3' (ECDSA) to the list of known hosts.

ps -aux | grep kubelet/config
root     13037  0.0  0.0 3632896 96800 ?       Ssl  18:18   0:09 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.2
root     16480  0.0  0.0  11468  1020 pts/0    S+   18:24   0:00 grep --color=auto kubelet/config


cat /var/lib/kubelet/config.yaml
apiVersion: kubelet.config.k8s.io/v1beta1
...
staticPodPath: /etc/just-to-mess-with-you
streamingConnectionIdleTimeout: 0s
syncFrequency: 0s
volumeStatsAggPeriod: 0s

ls /etc/just-to-mess-with-you
greenbox.yaml
rm -rf /etc/just-to-mess-with-you/greenbox.yaml
```

