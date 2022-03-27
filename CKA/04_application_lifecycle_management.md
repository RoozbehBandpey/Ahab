# Application Lifecycle Management

## Rolling Updates and Rollbacks

When we first create a deployment, it creates a rollout, a new rollout creates a new deployment revision. In the future when the application is upgraded and the container version is updated to a newer version, a new rollout and a new deployment revision is created. This helps us to keep track of changes in the deployment and enable us to rollback to  previous version of deployment if necessary. 

We can see the status of rollout by:

```bash
kubectl rollout status deployment/myapp-deployment
```

To see the revision and history of rollout:

```bash
kubectl rollout history deployment/myapp-deployment
```

### Deployment Strategy

For instance we have five replicas of our application instances deployed. 

1. To create a newer version of these is to destroy all and deploy the newer version, meaning first destroy five running instance and then deploy five new instances of newer application version. 
    * The problem is during period of upgrade, the application is down and inaccessible
    * This is known as recreate strategy
2. We don not destroy all of them at once, instead we take down older version and bring up newer version instance by instance
    * This way the upgrade is seamless
    * This is known as rolling update, the kubernetes default strategy

Once we made the necessary changes in the deployment manifest file we run the apply command to deploy the changes. A new rollout is triggered and a new revision of the deployment is created.

Or we could specifically update parts of deployment, for instance we could use:

```bash
kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1
```
To only update the image of deployment, but this way the deployment manifest file will have different configuration than the application.

When we view the deployment in details we can also see the difference between `Recreate` and `Rolling Update` strategies, with describe command. 
* In recreate: the replicaset is first scale down to 0, then scale up to 5
* Rolling update: thr replicaset is scale up and down one at a time

Under the hood when upgrade happens, kubernetes creates a new replicaset and one by one create the pods in the new one while at the same time take down the old pods in the older replicaset. This can be seen while rolling updates by running `kubectl get replicasets`.

If there's something wrong and we would like to rollback to the older version. Run the following:

```bash
kubectl rollout undo deployment/myapp-deployment
```

## Configure Applications
Configuring applications comprises of understanding the following concepts:

* Configuring Command and Arguments on applications

* Configuring Environment Variables

* Configuring Secrets
### Commands
Imagine we were to run a docker container from an ubuntu image, when we run `docker run ubuntu` it runs an instance of ubuntu image and exists immediately. If we list running the container `docker ps` we won't see anything, if we list running container and including those that stopped `docker ps -a` we can see the container. 

There reason that ubuntu stops, is that unlike virtual machines, containers are not meant to host an operating system. They are rather meant to run a specific task or process. Once the task is complete the container exits. Containers only live as long as the process inside it is alive.

If we look at the dockerfile for popular docker images, we'll see an instruction in CMD section that defines the process to be run within the container when it starts. `CMD ["nginx"]`, `CMD ["mysqld"]` but for ubuntu this command is `CMD ["bash"]` bash is not really a process like a webserver or database server, it is a shell that listens for the input from the terminal, if it cannot find the terminal, it exits! By default docker does not attach a terminal to container when it is ran. 

To specify a command to start a container:
1. Append a command to docker run command, that way it'll overwrite the command specified within the image. 
    * `docker run ubuntu [COMMAND]`, for example `docker run ubuntu sleep 5`
    * This way when the container starts it'll run the ubuntu and wait 5 seconds and then it terminates
2. We create our own image from the base ubuntu image and specify the command `CMD sleep 5`
    * Either in shell form `CMD command param1`
    * Or in json array format `CMD ["command", "param1"]`
    * Then build our own image `docker build -t ubuntu-sleeper .`
    * Then we run it `docker run ubuntu-sleeper`
    * If we want to change the sleep time we have to run `docker run ubuntu-sleeper sleep 5`
    * But a better way is entry point instructions, which it'll specify the program ehn the container runs `ENTRYPOINT ["sleep"]` what ever we specify in command-line will be appended to entry point so we can simply say `docker run ubuntu-sleeper 10`
    * In order to prevent error for cases like `docker run ubuntu-sleeper` we have to define a default value for command parameter we can use both entry point and command section in this case the command will be appended to the entry point instruction `ENTRYPOINT ["sleep"] CMD["5"`
    * To modify the entry point during the runtime we can run `docker run --entrypoint sleep2.0 ubuntu-sleeper 10`


### Commands and Arguments
Now let's create a pod using the sleeper image:
```yml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
```
When this pod is created, it'll create a container form a specified image, the container sleeps for 5 seconds and exits.

If we need to change the sleep parameter, anything that is appended in the docker run command will go to the args property of pod definition file, in form of an array.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
    command: ["sleep2.0"]
    args: ["10"]
```

If we need to modify the entrypoint command to a new command, we use the command filed, this will correspond to `ENTRYPOINT` in the dockerfile. 
### Environment Variables
To set an environment variable in docker we ca simply say `docker run -e APP_COLOR=pink simple-webapp-color`. 

In kubernetes while defining a pod in its manifest file we can use the env property to set environment variables:
```yml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
    env:
      - name: APP_COLOR
        value: pink
```
`env` is an array, each item has a name and value property.
### ConfigMaps

### Secrets

## Scale Applications