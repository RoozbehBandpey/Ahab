# Persistent Volume Claims

The application stores logs at location `/log/app.log`. View the logs.




You can `exec` in to the container and open the file:
`kubectl exec webapp -- cat /log/app.log`


If the POD was to get deleted now, would you be able to view these logs.

Use the command `kubectl delete` to delete a webapp pod and try to view those logs again.

>The logs are stored in the Container's file system that lives only as long as the Container does. Once the pod is destroyed, you cannot view the logs again.



Configure a volume to store these logs at `/var/log/webapp` on the host.



Use the spec provided below.

* Name: webapp
* Image Name: kodekloud/event-simulator
* Volume HostPath: /var/log/webapp
* Volume Mount: /log
```bash
kubectl run webapp --image=kodekloud/event-simulator --dry-run=client -o yaml > webapp.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: event-simulator
    image: kodekloud/event-simulator
    env:
    - name: LOG_HANDLERS
      value: file
    volumeMounts:
    - mountPath: /log
      name: log-volume

  volumes:
  - name: log-volume
    hostPath:
      # directory location on host
      path: /var/log/webapp
      # this field is optional
      type: Directory
```


Create a Persistent Volume with the given specification.

* Volume Name: pv-log
* Storage: 100Mi
* Access Modes: ReadWriteMany
* Host Path: /pv/log
* Reclaim Policy: Retain

Use the following manifest file to create a pv-log persistent volume:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  persistentVolumeReclaimPolicy: Retain
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 100Mi
  hostPath:
    path: /pv/log
```
Let us claim some of that storage for our application. Create a Persistent Volume Claim with the given specification.



* Volume Name: claim-log-1
* Storage Request: 50Mi
* Access Modes: ReadWriteOnce

Solution manifest file to create a claim-log-1 PVC with given properties as follows:
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
```
What is the state of the Persistent Volume Claim?

```bash
kubectl get persistentvolumeclaims
NAME          STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
claim-log-1   Pending       

kubectl get persistentvolumes     
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv-log   100Mi      RWX            Retain           Available                                   4m42s
```
Why is the claim not bound to the available Persistent Volume?


Run the command: `kubectl get pv,pvc` and look under the Access Modes section. The Access Modes set on the PV and the PVC do not match.


Update the Access Mode on the claim to bind it to the PV.



>Delete and recreate the claim-log-1.
```bash
kubectl delete pvc claim-log-1 
persistentvolumeclaim "claim-log-1" deleted
root@controlplane:~# vi  claim-log-1.yaml           
root@controlplane:~# kubectl apply -f claim-log-1.yaml 
persistentvolumeclaim/claim-log-1 created
root@controlplane:~# kubectl get pvc                
NAME          STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
claim-log-1   Bound    pv-log   100Mi      RWX                           21s
```

Update the webapp pod to use the persistent volume claim as its storage.



Replace `hostPath` configured earlier with the newly created `PersistentVolumeClaim`.
```bash
kubectl delete pod webapp --force
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: event-simulator
    image: kodekloud/event-simulator
    env:
    - name: LOG_HANDLERS
      value: file
    volumeMounts:
    - mountPath: /log
      name: log-volume

  volumes:
  - name: log-volume
    persistentVolumeClaim:
      claimName: claim-log-1
```
What is the Reclaim Policy set on the Persistent Volume pv-log?
```bash
kubectl get pv
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   REASON   AGE
pv-log   100Mi      RWX            Retain           Bound    default/claim-log-1                           13m
```
Why is the PVC stuck in Terminating state?


The PVC was still being used by the webapp pod when we issued the delete command. Until the pod is deleted, the PVC will remain in a terminating state.
```bash
kubectl get pv 
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM                 STORAGECLASS   REASON   AGE
pv-log   100Mi      RWX            Retain           Released   default/claim-log-1                           17m
```