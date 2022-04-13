# Worker Node Failure Lab

Fix the broken cluster
```bash
kubectl get nodes
NAME           STATUS     ROLES                  AGE   VERSION
controlplane   Ready      control-plane,master   14m   v1.20.0
node01         NotReady   <none>                 14m   v1.20.0
```
SH to node01 and check the status of container runtime (docker, in this case) and the kubelet service.

Since the kubelet is not running, attempt to start it by running:
```bash
systemctl start kubelet
root@node01:~# systemctl status kubelet
```
The cluster is broken again. Investigate and fix the issue.

kubelet has stopped running on node01 again. Since this is a systemd managed system, we can check the kubelet log by running journalctl. Here is a snippet showing the error with kubelet:


There appears to be a mistake path used for the CA certificate in the kubelet configuration. This can be corrected by updating the file /var/lib/kubelet/config.yaml.
Once this is fixed, restart the kubelet service, (like we did in the previous question) and node01 should return back to a working state.

The cluster is broken again. Investigate and fix the issue.

Once again the kubelet service has stopped working. Checking the logs, we can see that this time, it is not able to reach the kube-apiserver.
```bash
journalctl -u kubelet 
```
As we can clearly see, kubelet is trying to connect to the API server on the controlplane node on port 6553. This is incorrect.
To fix, correct the port on the kubeconfig file used by the

To get the right port run:

```bash
kubectl cluster-info
```
```bash
cat /etc/kubernetes/kubelet.conf
```