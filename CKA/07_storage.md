# Storage

## Storage in Docker

When you install docker it'll create the `/var/lib/docker` this is where docker stores all of its data by default. It has sub-folders such as `aufs`, `containers`, `image` and `volumes`.

When we run `docker volume create data_vol` it'll create a folder under `/var/lib/docker/volume/data_vol`

Then we can mount this volume using `-v` option in run command `docker run -v data_vol:/var/lib/mysql mysql`

## Container Storage Interface (CSI)

In the past kubernetes used docker alone as the container runtime engine, and all the code to work with docker was embedded within kubernetes source code. Later kubernetes introduced CRI (Container Runtime Interface) in order not to be dependant on docker. It is an standard that defines how an orchestration solution like kubernetes would communicate with a containerization solution like docker or rkt. Similarly in order to deal with variety of networking vendors CNI (Container Networking Interface) was introduced. An the CSI (Container Storage Interface) was developed to interact with variety of storage solutions. 

CSI is not specific to kubernetes and it is a universal standard.


## Volumes

Docker containers are meant to be transient in nature, which means they are meant to last only for the short period of time. They are call upon to process and terminate when the process is finished. It is the same with their data, the data is destroyed along with the container. To persist data we attach a volume to the container when they are created. The data processed by the container is now placed in these volumes. Even if the container is terminated, the data generated and processed is persisted. 

Just like docker, the pods created in kubernetes are transient in nature.

Let's take a look at a pod that has a volume mounted to it. 

The `hostPath` option to configure a directory on the host:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
  - name: alpine
    image: alpine
    command: ["/bin/sh", "-c"]
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
    volumeMounts:
    - mountPath: /opt
      name: data-volume

  volumes:
  - name: data-volume
    hostPath:
      path: /data
      type: Directory
```

The random will be now written in the `/opt` in the container which happens to be in the `data-volume` which is in fact in the `/data` in the host. This is not a recommended approach in a multi-node cluster, the pods will use `/data` directory in all nodes and expect them to be the same and have same data. 

Kubernetes support various different storage solutions such as `NFS`, `GlusterFS`, `Flocker`, `AWS EBS`, `AzureDisk`, `Google Persistent Disk`

For example to configure an AWS Elastic Blok Store volume:

```yaml
  volumes:
  - name: data-volume
    awsElasticBlokStore:
      volumeID: <volume-id>
      fsType: ext4
```

## Persistent Volumes

In the large environment when a lot of users deploy a lot pods managing volumes becomes cumbersome. In order to manage the volumes centrally and let the users use it we have to use persistent volume. It is a cluster-wide pool of storage volumes configured by an administrator to be used by  thee users deploying applications on thee  cluster. The users can now select storage from this pool using persistent volume claims. 

The manifest file for persistent volume is as follows:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /tmp/data # Do not use this in production environment
```
`accessMode` defines how a volume should be mounted on the host whether in `ReadOnlyMany`, `ReadWriteOnce` or `ReadWriteMany`

```bash
kubectl get persistentvolume
```
## Persistent Volumes Claims

To make a storage available to a node we have to use persistent volume claims. Once PVCs are created, kubernetes binds the persistent volumes to PVCs based on the request and properties set on the volumes. 

Every persistent volume claim is bound to a single persistent volume. During thee binding process kubernetes tries to find a persistent volume that has sufficient capacity that is request by the claim. As well as any other request properties such as `accessMode`, `volumeMode`, `storageClass` etc., However if there are multiple matches available and we'd like to specifically use a volume  we could still use labels and selector to bind to a specific volume.

It could be that smaller claim binds to a larger volume if there are no other criteria matches and there are no better options. No other claim can utilize the remaining capacity since there's 1:1 relation between claims and volumes. 

The manifest file for persistent volume claims is as follows:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

Whe we get the claims we can see the volume bindings:

```bash
kubectl get persistentvolumeclaims
```

When a PVC is deleted  we can choose the destiny of the volume. By default the `persistentVolumeClaimPolicy` is set to `Retain` meaning the persistent volume will remain until it is manually deleted. A claim can be also deleted automatically if we set it to `Delete`. A third option would be `Recycle` in this case the data in the volume will be scrubbed before making it available to other claims. 


