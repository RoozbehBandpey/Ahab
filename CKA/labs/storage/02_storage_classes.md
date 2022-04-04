# Storage Classes Lab

How many StorageClasses exist in the cluster right now?

Run the command: `kubectl get sc` and count the number of Storage Classes available in the cl

What is the name of the Storage Class that does not support dynamic volume provisioning?

Look for the storage class name that uses `no-provisioner`

What is the Volume Binding Mode used for this storage class

Is there a `PersistentVolumeClaim` that is consuming the `PersistentVolume` called local-pv?
```bash
kubectl get pvc
No resources found in default namespace.
```
Let's fix that. Create a new PersistentVolumeClaim by the name of local-pvc that should bind to the volume local-pv.

Inspect the pv local-pv for the specs.

Use the below YAML file to create the PersistentVolumeClaim local-pvc:
```bash
kubectl describe pv local-pv
Name:              local-pv
Labels:            <none>
Annotations:       <none>
Finalizers:        [kubernetes.io/pv-protection]
StorageClass:      local-storage
Status:            Available
Claim:             
Reclaim Policy:    Retain
Access Modes:      RWO
VolumeMode:        Filesystem
Capacity:          500Mi
Node Affinity:     
  Required Terms:  
    Term 0:        kubernetes.io/hostname in [controlplane]
Message:           
Source:
    Type:  LocalVolume (a persistent volume backed by local storage on a node)
    Path:  /opt/vol1
Events:    <none>
```
```yaml
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: local-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: local-storage
  resources:
    requests:
      storage: 500Mi
```
Why is the PVC in a pending state despite making a valid request to claim the volume called local-pv?

Inspect the PVC events.


>The `StorageClass` used by the PVC uses `WaitForFirstConsumer` volume binding mode. This means that the persistent volume will not bind to the claim until a pod makes use of the PVC to request storage.
```bash
root@controlplane:~# kubectl describe pvc local-pvc | grep -A3 Events
Events:
  Type    Reason                Age               From                         Message
  ----    ------                ----              ----                         -------
  Normal  WaitForFirstConsumer  1s (x7 over 78s)  persistentvolume-controller  waiting for first consumer to be created before binding
```

Create a new pod called nginx with the image nginx:alpine. The Pod should make use of the PVC local-pvc and mount the volume at the path `/var/www/html`.

The PV local-pv should in a bound state.

Solution manifest file to create a pod called nginx is as follows:
```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    volumeMounts:
      - name: local-persistent-storage
        mountPath: /var/www/html
  volumes:
    - name: local-persistent-storage
      persistentVolumeClaim:
        claimName: local-pvc
```