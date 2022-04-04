# Storage

## Storage in Docker

When you install docker it'll create the `/var/lib/docker` this is where docker stores all of its data by default. It has sub-folders such as `aufs`, `containers`, `image` and `volumes`.

When we run `docker volume create data_vol` it'll create a folder under `/var/lib/docker/volume/data_vol`

Then we can mount this volume using `-v` option in run command `docker run -v data_vol:/var/lib/mysql mysql`

## Container Storage Interface (CSI)

In the past kubernetes used docker alone as the container runtime engine, and all the code to work with docker was embedded within kubernetes source code. Later kubernetes introduced CRI (Container Runtime Interface) in order not to be dependant on docker. It is an standard that defines how an orchestration solution like kubernetes would communicate with a containerization solution like docker or rkt. Similarly in order to deal with variety of networking vendors CNI (Container Networking Interface) was introduced. An the CSI (Container Storage Interface) was developed to interact with variety of storage solutions. 

CSI is not specific to kubernetes and it is a universal standard.


## Persistent Volumes

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