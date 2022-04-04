# Storage

## Storage in Docker

When you install docker it'll create the `/var/lib/docker` this is where docker stores all of its data by default. It has sub-folders such as `aufs`, `containers`, `image` and `volumes`.

When we run `docker volume create data_vol` it'll create a folder under `/var/lib/docker/volume/data_vol`

Then we can mount this volume using `-v` option in run command `docker run -v data_vol:/var/lib/mysql mysql`

## Container Storage Interface (CSI)

In the past kubernetes used docker alone as the container runtime engine, and all the code to work with docker was embedded within kubernetes source code. Later kubernetes introduced CRI (Container Runtime Interface) in order not to be dependant on docker. It is an standard that defines how an orchestration solution like kubernetes would communicate with a containerization solution like docker or rkt. Similarly in order to deal with variety of networking vendors CNI (Container Networking Interface) was introduced. An the CSI (Container Storage Interface) was developed to interact with variety of storage solutions. 

CSI is not specific to kubernetes and it is a universal standard.


## Persistent Volumes

