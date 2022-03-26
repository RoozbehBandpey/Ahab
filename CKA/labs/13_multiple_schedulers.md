# Multiple Schedulers

What is the name of the POD that deploys the default kubernetes scheduler in this environment?
```bash
kubectl get pods -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
coredns-74ff55c5b-fh85n                1/1     Running   0          7m12s
coredns-74ff55c5b-v9f6r                1/1     Running   0          7m11s
etcd-controlplane                      1/1     Running   0          7m20s
kube-apiserver-controlplane            1/1     Running   0          7m20s
kube-controller-manager-controlplane   1/1     Running   0          7m20s
kube-flannel-ds-fxrdk                  1/1     Running   0          7m9s
kube-proxy-vmg9s                       1/1     Running   0          7m12s
kube-scheduler-controlplane            1/1     Running   0          7m20s
```
What is the image used to deploy the kubernetes scheduler?



Inspect the kubernetes scheduler pod and identify the image
```bash
kubectl describe pods kube-scheduler-controlplane -n kube-system | grep Image
    Image:         k8s.gcr.io/kube-scheduler:v1.20.0
    Image ID:      docker-pullable://k8s.gcr.io/kube-scheduler@sha256:beaa710325047fa9c867eff4ab9af38d9c2acec05ac5b416c708c304f76bdbef
```
Deploy an additional scheduler to the cluster following the given specification.



Use the manifest file used by kubeadm tool. Use a different port than the one used by the current one.
```bash
ps -aux | grep kubelet/config
root      4844  0.0  0.0 4151580 96960 ?       Ssl  19:00   0:38 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.2
root     14736  0.0  0.0  11468   972 pts/0    R+   19:16   0:00 grep --color=auto kubelet/config
root@controlplane:~# grep -i static /var/lib/kubelet/config.yaml
staticPodPath: /etc/kubernetes/manifests
root@controlplane:~# ls /etc/kubernetes/manifests
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
```
```bash
cp /etc/kubernetes/manifests/kube-scheduler.yaml /root/my-scheduler.yaml
```
```bash
kubectl apply -f my-scheduler.yaml 
error: error validating "my-scheduler.yaml": error validating data: [ValidationError(Pod.spec.containers[0].livenessProbe): unknown field "name" in io.k8s.api.core.v1.Probe, ValidationError(Pod.spec.containers[0]): missing required field "name" in io.k8s.api.core.v1.Container]; if you choose to ignore these errors, turn validation off with --validate=false
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    component: my-scheduler
    tier: control-plane
  name: my-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
    - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
    - --bind-address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=false
    - --port=10282
    - --scheduler-name=my-scheduler
    - --secure-port=0
    image: k8s.gcr.io/kube-scheduler:v1.19.0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10282
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-scheduler
    resources:
      requests:
        cpu: 100m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: 10282
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/kubernetes/scheduler.conf
      name: kubeconfig
      readOnly: true
  hostNetwork: true
  priorityClassName: system-node-critical
  volumes:
  - hostPath:
      path: /etc/kubernetes/scheduler.conf
      type: FileOrCreate
    name: kubeconfig
status: {}
```

> Port is really important here, as well as HTTP instead of HTTPS


A POD definition file is given. Use it to create a POD with the new custom scheduler.



File is located at /root/nginx-pod.yaml
```bash
kubectl get events
LAST SEEN   TYPE     REASON                    OBJECT              MESSAGE
...
73s         Normal   Scheduled                 pod/nginx           Successfully assigned default/nginx to controlplane
72s         Normal   Pulling                   pod/nginx           Pulling image "nginx"
64s         Normal   Pulled                    pod/nginx           Successfully pulled image "nginx" in 8.644480658s
63s         Normal   Created                   pod/nginx           Created container nginx
62s         Normal   Started                   pod/nginx           Started container nginx
```