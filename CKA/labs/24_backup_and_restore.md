# Backup and Restore Lab

What is the version of ETCD running on the cluster?



>Check the ETCD Pod or Process


```bash
kubectl get pods -A        
NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
default       blue-746c87566d-765zf                  1/1     Running   0          2m19s
default       blue-746c87566d-87fsg                  1/1     Running   0          2m19s
default       blue-746c87566d-lklxd                  1/1     Running   0          2m19s
default       red-75f847bf79-8h8mh                   1/1     Running   0          2m19s
default       red-75f847bf79-ttdnm                   1/1     Running   0          2m19s
kube-system   coredns-74ff55c5b-8z6ld                1/1     Running   0          9m36s
kube-system   coredns-74ff55c5b-s6sx9                1/1     Running   0          9m36s
kube-system   etcd-controlplane                      1/1     Running   0          9m48s
kube-system   kube-apiserver-controlplane            1/1     Running   0          9m48s
kube-system   kube-controller-manager-controlplane   1/1     Running   0          9m48s
kube-system   kube-flannel-ds-55rt9                  1/1     Running   0          9m36s
kube-system   kube-proxy-8p695                       1/1     Running   0          9m36s
kube-system   kube-scheduler-controlplane            1/1     Running   0          9m48s
```
```bash
kubectl describe pod etcd-controlplane -n kube-system | grep Image
    Image:         k8s.gcr.io/etcd:3.4.13-0
    Image ID:      docker-pullable://k8s.gcr.io/etcd@sha256:4ad90a11b55313b182afc186b9876c8e891531b8db4c9bf1541953021618d0e2
```
At what address can you reach the ETCD cluster from the controlplane node?



>Check the ETCD Service configuration in the ETCD POD

```bash
kubectl describe pod etcd-controlplane -n kube-system | grep listen-client-urls
      --listen-client-urls=https://127.0.0.1:2379,https://10.56.220.3:2379
```

Where is the ETCD server certificate file located?



>Note this path down as you will need to use it later
```bash
kubectl describe pod etcd-controlplane -n kube-system | grep cert-file         
      --cert-file=/etc/kubernetes/pki/etcd/server.crt
```

Where is the ETCD CA Certificate file located?



>Note this path down as you will need to use it later.
```bash
kubectl describe pod etcd-controlplane -n kube-system | grep trusted-ca-file
      --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
      --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
```

The master nodes in our cluster are planned for a regular maintenance reboot tonight. While we do not anticipate anything to go wrong, we are required to take the necessary backups. Take a snapshot of the ETCD database using the built-in snapshot functionality.



Store the backup file at location /opt/snapshot-pre-boot.db
```bash
ETCDCTL_API=3 etcdctl snapshot save /opt/snapshot-pre-boot.db --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key
Snapshot saved at /opt/snapshot-pre-boot.db
```
Restore the original state of the cluster using the backup file.
```bash
ETCDCTL_API=3 etcdctl snapshot restore /opt/snapshot-pre-boot.db --data-dir /var/lib/etcd-from-backup
2022-03-28 21:35:43.195440 I | mvcc: restore compact to 1636
2022-03-28 21:35:43.203241 I | etcdserver/membership: added member 8e9e05c52164694d [http://localhost:2380] to cluster cdf818194e3a8c32
```
Next, update the /etc/kubernetes/manifests/etcd.yaml:

We have now restored the etcd snapshot to a new path on the controlplane - /var/lib/etcd-from-backup, so, the only change to be made in the YAML file, is to change the hostPath for the volume called etcd-data from old directory (/var/lib/etcd) to the new directory /var/lib/etcd-from-backup.
```yaml
  volumes:
  - hostPath:
      path: /var/lib/etcd-from-backup
      type: DirectoryOrCreate
    name: etcd-data
```
With this change, /var/lib/etcd on the container points to /var/lib/etcd-from-backup on the controlplane (which is what we want)

When this file is updated, the ETCD pod is automatically re-created as this is a static pod placed under the /etc/kubernetes/manifests directory.

Note: as the ETCD pod has changed it will automatically restart, and also kube-controller-manager and kube-scheduler. Wait 1-2 to mins for this pods to restart. You can run a watch "docker ps | grep etcd" command to see when the ETCD pod is restarted.

Note2: If the etcd pod is not getting Ready 1/1, then restart it by kubectl delete pod -n kube-system etcd-controlplane and wait 1 minute.

Note3: This is the simplest way to make sure that ETCD uses the restored data after the ETCD pod is recreated. You don't have to change anything else.

If you do change --data-dir to /var/lib/etcd-from-backup in the YAML file, make sure that the volumeMounts for etcd-data is updated as well, with the mountPath pointing to /var/lib/etcd-from-backup (THIS COMPLETE STEP IS OPTIONAL AND NEED NOT BE DONE FOR COMPLETING THE RESTORE)