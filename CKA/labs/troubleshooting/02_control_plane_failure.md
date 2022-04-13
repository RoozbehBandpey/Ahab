# Control Plane Failure

The cluster is broken again. We tried deploying an application but it's not working. Troubleshoot and fix the issue.



Start looking at the deployments.
```bash
kubectl get all   
NAME                       READY   STATUS    RESTARTS   AGE
pod/app-586bddbc54-phdhh   0/1     Pending   0          87s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   10m

NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/app   0/1     1            0           88s

NAME                             DESIRED   CURRENT   READY   AGE
replicaset.apps/app-586bddbc54   1         1         0       87s
```
```bash
root@controlplane:~# kubectl get pods -n kube-system
NAME                                   READY   STATUS             RESTARTS   AGE
coredns-74ff55c5b-gd8qn                1/1     Running            0          11m
coredns-74ff55c5b-pvxgm                1/1     Running            0          11m
etcd-controlplane                      1/1     Running            0          11m
kube-apiserver-controlplane            1/1     Running            0          11m
kube-controller-manager-controlplane   1/1     Running            0          11m
kube-flannel-ds-9lxqn                  1/1     Running            0          11m
kube-proxy-4m6tm                       1/1     Running            0          11m
kube-scheduler-controlplane            0/1     CrashLoopBackOff   4          2m43s
```
```bash
kubectl describe pods kube-scheduler-controlplane -n kube-system
Name:                 kube-scheduler-controlplane
...
Events:
  Type     Reason   Age                    From     Message
  ----     ------   ----                   ----     -------
  Normal   Pulled   5m34s (x5 over 6m55s)  kubelet  Container image "k8s.gcr.io/kube-scheduler:v1.20.0" already present on machine
  Normal   Created  5m34s (x5 over 6m55s)  kubelet  Created container kube-scheduler
  Warning  Failed   5m33s (x5 over 6m54s)  kubelet  Error: failed to start container "kube-scheduler": Error response from daemon: OCI runtime create failed: container_linux.go:367: starting container process caused: exec: "kube-schedulerrrr": executable file not found in $PATH: unknown
  Warning  BackOff  108s (x26 over 6m53s)  kubelet  Back-off restarting failed container

  The command run by the scheduler pod is incorrect. Here is a snippet of the YAML file.
```
```yaml
spec:
  containers:
  - command:
    - kube-scheduler
    - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
    - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
    - --bind-address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=true
    - --port=0
```
```bash
cat /var/lib/kubelet/config.yaml |  grep staticPod
staticPodPath: /etc/kubernetes/manifests
```
```bash
ls /etc/kubernetes/manifests
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
root@controlplane:~# vi /etc/kubernetes/manifests/kube-scheduler.yaml
```
Scale the deployment app to 2 pods.
```bash
kubectl scale deployments app --replicas=2
deployment.apps/app scaled
root@controlplane:~# kubectl get deployments
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
app    1/2     1            1           21m
root@controlplane:~# kubectl get pods
NAME                   READY   STATUS    RESTARTS   AGE
app-586bddbc54-phdhh   1/1     Running   0          22m
```

Even though the deployment was scaled to 2, the number of PODs does not seem to increase. Investigate and fix the issue.



Inspect the component responsible for managing deployments and replicasets.
```bash
kubectl get pods -n kube-system
]NAME                                   READY   STATUS             RESTARTS   AGE
coredns-74ff55c5b-gd8qn                1/1     Running            0          31m
coredns-74ff55c5b-pvxgm                1/1     Running            0          31m
etcd-controlplane                      1/1     Running            0          31m
kube-apiserver-controlplane            1/1     Running            0          31m
kube-controller-manager-controlplane   0/1     CrashLoopBackOff   4          2m33s
kube-flannel-ds-9lxqn                  1/1     Running            0          31m
kube-proxy-4m6tm                       1/1     Running            0          31m
kube-scheduler-controlplane            1/1     Running            0          3m9s
```
```bash
kubectl logs kube-controller-manager-controlplane -n kube-system
Flag --port has been deprecated, see --secure-port instead.
I0413 19:12:18.319994       1 serving.go:331] Generated self-signed cert in-memory
stat /etc/kubernetes/controller-manager-XXXX.conf: no such file or directory
```
The configuration file specified (/etc/kubernetes/controller-manager-XXXX.conf) does not exist.
Correct the path: /etc/kubernetes/controller-manager.conf


Something is wrong with scaling again. We just tried scaling the deployment to 3 replicas. But it's not happening.



Investigate and fix the issue.
```bash
kubectl get pods -n kube-system
NAME                                   READY   STATUS             RESTARTS   AGE
coredns-74ff55c5b-gd8qn                1/1     Running            0          38m
coredns-74ff55c5b-pvxgm                1/1     Running            0          38m
etcd-controlplane                      1/1     Running            0          38m
kube-apiserver-controlplane            1/1     Running            0          38m
kube-controller-manager-controlplane   0/1     CrashLoopBackOff   3          48s
kube-flannel-ds-9lxqn                  1/1     Running            0          38m
kube-proxy-4m6tm                       1/1     Running            0          38m
kube-scheduler-controlplane            1/1     Running            0          9m47s
root@controlplane:~# kubectl logs kube-controller-manager-controlplane -n kube-system
Flag --port has been deprecated, see --secure-port instead.
I0413 19:18:22.971555       1 serving.go:331] Generated self-signed cert in-memory
unable to load client CA file "/etc/kubernetes/pki/ca.crt": open /etc/kubernetes/pki/ca.crt: no such file or directory
```
It appears the path /etc/kubernetes/pki is not mounted from the controlplane to the kube-controller-manager pod. If we inspect the pod manifest file, we can see that the incorrect hostPath is used for the volume:

