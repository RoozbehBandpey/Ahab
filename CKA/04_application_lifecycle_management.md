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
When we have a lot of pods it'll become difficult to manage environment data. We can take this information out of the pod definition file and manage it centrally using configuration maps. ConfigMaps are used to store configuration data in form of key-value pairs. When the pod is created inject the config map into the pod so the key-value pairs are available as environment variables for the applications hosted in the pods. 

ConfigMaps can be created with imperative and declarative ways.
* `kubectl create configmap`
    * `kubectl create configmap <config-name> --from-literal=<key>=<value>`
    * `kubectl create configmap app-config --from-literal=APP_COLOR=blue`
    * For more values we can specify the `--from-literal` option multiple times
    * Or we can use a config file: `kubectl create configmap <config-name> --from-file=<path-to-file>`
    * `kubectl create configmap app-config --from-file=app_config.properties`
* `kubectl create -f <config-map-file>`
    * We create a definition file same as any other object but instead of `spec` we have `data`
    * Create by running `kubectl create -f config-map.yaml`
```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

ConfigMaps can be viewed by:

```bash
kubectl get configmap
```

To configure a pod to use ConfigMap we can use `envFrom`, this property is a list so we can pass as many environment variables as required. Each value corresponds to a ConfigMap item. 
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
    envFrom:
      - configMapRef:
          name: app-config
```
ConfigMap can also be passed to pod with `valueFrom` property.
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
        valueFrom:
          configMapKeyRef:
            name: app-config
            key: APP_COLOR
```
### Secrets
Secrets are used to store sensitive information like passwords or keys. They are similar to ConfigMaps except that they are stored in encoded format. 

Secrets can be created with imperative and declarative ways.
* `kubectl create secret generic`
    * `kubectl create secret generic <secret-name> --from-literal=<key>=<value>`
    * `kubectl create secret generic app-secret --from-literal=DB_Host=mysql`
    * For more values we can specify the `--from-literal` option multiple times
    * Or we can use a config file: `kubectl create secret generic <secret-name> --from-file=<path-to-file>`
    * `kubectl create secret generic app-secret --from-file=app_secret.properties`
* `kubectl create -f <secret-file>`
    * We create a definition file same as any other object but instead of `spec` we have `data`
    * Create by running `kubectl create -f secret.yaml`
```yml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: mysql
  DB_User: root
  DB_Password: password
```

However the secrets created from manifest files does not seem to be really secure, because they are stored in YAML format. When we create the secrets with manifest files we must specify the data un the hashed format. 

In order to convert the data to encode format, on the linus host run
```bash
echo -n 'mysql' | base64
echo -n 'root' | base64
echo -n 'password' | base64
```

To view secrets run

```bash
kubectl get secrets
```

To get more details run
```bash
kubectl describe secrets
```

To view the secret values, run
```bash
kubectl get secrets app-secret -o yaml
```

In order to decode the hashed values run the following:

```bash
echo -n 'bX3k4jhb' | base64 --decode
echo -n 'jh354HHGD' | base64 --decode
echo -n '45Ghjhgr4' | base64 --decode
```

To configure a pod to use Secret we can use `envFrom`, this property is a list so we can pass as many secrets as required. Each value corresponds to a Secret item. 
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
    envFrom:
      - secretRef:
          name: app-secret
```

The secrets will be accessible as environment variables within the application. There are other ways to inject secrets into pods, such as single environment variables or inject the whole secret as files in a volume. 

If we were to mount the secret as a file into the volume, each attribute in the secret is created as a file with the value of the secret as its content.

Secrets encode data in base64 format. Anyone with the base64 encoded secret can easily decode it. As such the secrets can be considered as not very safe.

The concept of safety of the Secrets is a bit confusing in Kubernetes. The kubernetes documentation page and a lot of blogs out there refer to secrets as a "safer option" to store sensitive data. They are safer than storing in plain text as they reduce the risk of accidentally exposing passwords and other sensitive data. It's not the secret itself that is safe, it is the practices around it. 

Secrets are not encrypted, so it is not safer in that sense. However, some best practices around using secrets make it safer. As in best practices like:

* Not checking-in secret object definition files to source code repositories.

* [Enabling Encryption at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) for Secrets so they are stored encrypted in ETCD. 



Also the way kubernetes handles secrets. Such as:

* A secret is only sent to a node if a pod on that node requires it.

* Kubelet stores the secret into a tmpfs so that the secret is not written to disk storage.

* Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.

Read about the protections and risks of using secrets here



Having said that, there are other better ways of handling sensitive data like passwords in Kubernetes, such as using tools like Helm Secrets, HashiCorp Vault. 

## Multi Container PODs
The idea of decoupling a large monolith application into sub-components known as microservices enable us to develop and deploy a set of small and reusable code. This architecture helps us to scale up and down as each service requires, instead of scaling the entire application. However at time we may need two service to work together such as a web-server and a logging-service.

We don't want to merge the code as each of them target different functionalities but we still want them to be developed and deployed separately. We need one logging agent per web server paired together that can be scaled together. 

Multi-container pods share the same life-cycle, meaning they are created together and destroyed together. They share the same network space meaning they can refer to each other as localhost and they have access to the same storage volumes. 

To create a multi-container pod we can add the new container definition to the pod:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
      - containerPort: 8080
  - name: log-agent
    image: log-agent
```
## Multi-container PODs Design Patterns
There are 3 common patterns, when it comes to designing multi-container PODs. The first and what we just saw with the logging service example is known as a side car pattern. The others are the adapter and the ambassador pattern.
## InitContainers
In a multi-container pod, each container is expected to run a process that stays alive as long as the POD's lifecycle. For example in the multi-container pod that we saw earlier that has a web application and logging agent, both the containers are expected to stay alive at all times. The process running in the log agent container is expected to stay alive as long as the web application is running. If any of them fails, the POD restarts.



But at times you may want to run a process that runs to completion in a container. For example a process that pulls a code or binary from a repository that will be used by the main web application. That is a task that will be run only one time when the pod is first created. Or a process that waits for an external service or database to be up before the actual application starts. That's where `initContainers` comes in.



An `initContainer` is configured in a pod like all other containers, except that it is specified inside a initContainers section,  like this:


```yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ; done;']
```

When a POD is first created the initContainer is run, and the process in the initContainer must run to a completion before the real container hosting the application starts. 

You can configure multiple such initContainers as well, like how we did for multi-pod containers. In that case each init container is run one at a time in sequential order.

If any of the initContainers fail to complete, Kubernetes restarts the Pod repeatedly until the Init Container succeeds.
```yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

## Self Healing Applications
