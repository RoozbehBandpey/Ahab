# Replicasets practice test

How many ReplicaSets exist on the system?
```bash
kubectl get replicasets
No resources found in default namespace.
```

How many PODs are DESIRED in the new-replica-set?
```bash
kubectl get replicaset new-replica-set
NAME              DESIRED   CURRENT   READY   AGE
new-replica-set   4         4         0       85s
```

What is the image used to create the pods in the new-replica-set?
```bash
kubectl get replicaset new-replica-set -o wide
NAME              DESIRED   CURRENT   READY   AGE     CONTAINERS          IMAGES       SELECTOR
new-replica-set   4         4         0       2m54s   busybox-container   busybox777   name=busybox-pod
```
Why do you think the PODs are not ready?

Run the command: kubectl describe pods and look at under the Events section.
```bash
kubectl describe pods new-replica-set-xkthk

#Output
...
Events:
  Type     Reason     Age                    From               Message
  ----     ------     ----                   ----               -------
  Normal   Scheduled  4m38s                  default-scheduler  Successfully assigned default/new-replica-set-xkthk to controlplane
  Normal   Pulling    2m52s (x4 over 4m19s)  kubelet            Pulling image "busybox777"
  Warning  Failed     2m51s (x4 over 4m19s)  kubelet            Failed to pull image "busybox777": rpc error: code = Unknown desc = failed to pull and unpack image "docker.io/library/busybox777:latest": failed to resolve reference "docker.io/library/busybox777:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
  Warning  Failed     2m51s (x4 over 4m19s)  kubelet            Error: ErrImagePull
  Warning  Failed     2m41s (x6 over 4m19s)  kubelet            Error: ImagePullBackOff
  Normal   BackOff    2m29s (x7 over 4m19s)  kubelet            Back-off pulling image "busybox777"
```


Why are there still 4 PODs, even after you deleted one?

ReplicaSet ensures that desired number of PODs always run


Create a ReplicaSet using the replicaset-definition-1.yaml file located at /root/.

>There is an issue with the file, so try to fix it.
```bash
ls

vi replicaset-definition-1.yaml
```
apiVersion is wrong, must be set to apps/v1
```bash
kubectl create -f replicaset-definition-1.yaml
```

Fix the original replica set new-replica-set to use the correct busybox image.

Either delete and recreate the ReplicaSet or Update the existing ReplicaSet and then delete all PODs, so new ones with the correct image will be created.

```bash
kubectl edit replicasets new-replica-set
```

Scale the ReplicaSet to 5 PODs.

Use kubectl scale command or edit the replicaset using kubectl edit replicaset.

```bash
kubectl scale --replicas=5 replicaset new-replica-set
replicaset.apps/new-replica-set scaled
```