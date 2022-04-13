# Networking

## Routing & Switches

List and modify interfaces on the host

```bash
ip link
```

See ip addresses assigned to those interfaces

```bash
ip addr
```

Set ip addresses on the interfaces
```bash
ip addr add 192.168.1.10/24 dev eth0
```

>Changes made by these commands are only valid until next restart

>To persist  the changes we must set them in the etc network interface file

View the routing table
```bash
route

or

ip route
```

Adding entries into the routing table

```bash
ip route add 192.168.1.10/24 via 192.168.2.1
```

Finally to check if ip forwarding is enabled on the host

```
cat /proc/sys/net/ipv4/ip_forward

1
```

## DNS on Linux

DNS name resolution is handled via a DNS server, the host will bee pointed to the server to look up the DNS names. Every host has a DNS configuration file at `etc/resolve.conf` with an entry into it specifying the address of the DNS server. 

```bash
cat /etc/resolve.conf

nameserver 192.168.1.100
```

Once this is set up across all hosts, every time that a name comes up that host does not know about it, it'll look it up from the DNS server.

the local  `etc/hosts` file has priority over the DNS server, meaning  if we have the same name in the host file and DNS server the file in the `etc/hosts` file is taken into account. 

The order is defined in `etc/nsswitch.conf` and can  be changed

```bash
cat /etc/nsswitch.conf

...
hosts:      files   dns
...
```

### Core DNS

CoreDNS binaries can be downloaded from their Github releases page or as a docker image. Let’s go the traditional route. Download the binary using curl or wget. And extract it. You get the coredns executable.

Run the executable to start a DNS server. It by default listens on port `53`, which is the default port for a DNS server.



Now we haven’t specified the IP to hostname mappings. For that you need to provide some configurations. There are multiple ways to do that. We will look at one. First we put all of the entries into the DNS servers `/etc/hosts` file.

And then we configure CoreDNS to use that file. CoreDNS loads it’s configuration from a file named Corefile. Here is a simple configuration that instructs CoreDNS to fetch the IP to hostname mappings from the file /etc/hosts. When the DNS server is run, it now picks the Ips and names from the /etc/hosts file on the server.


## Network Namespaces

Network namespaces are used by containers like docker to implement network isolation. Containers are separated from underlying host with namespaces. This can be seen when we list processes from within a container or as a root user on the host:

```bash
ps aux
```
When it comes to networking the host has its own interfaces when connects to LAN. as well as route tables. When we create a container on the host, the container will have its  own network namespace therefore it has no visibility to the networking components of the host. Within the network namespace a container can have its own virtual interface, routing and  ARP table. 

To  create a new network namespace run:

```bash
ip netns add red
```

To list the network namespaces run:

```bash
ip netns

red
```

To list the interfaces on the host
```bash
ip link
```
To list the interfaces on the `red` container:

```bash
ip netns exec red ip link
```
or
```bash
ip -n red link
```

To view the ARP table inside the container:

```bash
ip netns exec red arp
```
To view the routing table inside the container:

```bash
ip netns exec red route
```

To establish connectivity between namespaces themselves, just like how we would connect two physical machines together using a cable  to an ethernet interface, we can connect two namespaces using a virtual ethernet pair or a virtual cable. It is often refer to as a pipe.

To create the cable run the:

```bash
ip link add veth-red type veth peer name veth-blue
```
Next we have  to attach each  interface to the appropriate namespace:

```bash
ip link set veth-red netns red
```
```bash
ip link set veth-blue netns blue
```
Then we can assign ip addresses to each of these namespaces
```bash
ip  -n red addr  192.168.15.1 dev veth-red
```
```bash
ip  -n blue addr  192.168.15.2 dev veth-blue
```
Then we bring up each interfaces

```bash
ip -n red link set veth-red up
```
```bash
ip -n blue link set veth-blue up
```

For testing try to ping the ip of the blue from the red namespace

```bash
ip netns exec red ping 192.168.15.2
```

If we have various namespaces we have create  a virtual network inside the host, to create a virtual switch we can use `Linux Bridge` or `Open vSwitch` 

With `Linux Bridge` we have to first add a new interface to the host with thee type set tto `bridge`:

```bash
ip link add v-net-0 type bridge
```

## Networking in Docker

A docker host (a server  with docker installed on it) has an ethernet interface `eth0` that connects to the LAN. When you run a container you have different networking options to choose from `docker run --network none nginx`

* `none`: The docker container is not attached to any network, the container cannot reach  the outside world and no one from outside world can reach the container. 
* `host`: The container is attached to the host network, if you deploy an application on the port `80` of  the container then the container is reachable on the port `80` of the host without any additional port mapping. If we try to run another instance of the container that listens on the same port, it won't work as they share the host networking.
* `bridge`: An internal private network get created which the containers and docker host attach to it. Each device connecting to this bridge get their own private IP address. 

When docker is installed on a host, it'll create a virtual private network called `bridge` you can see this when you run `docker network ls` command. On the host the network is created by the name `docker0` you can see this in the output of `ip link` command.

Whenever a container is created, docker creates a network namespace for it. To list the namespaces run `ip netns` 

## Container Networking Interface (CNI)

The CNI is set of standards that defines how programs should be developed to solve the networking challenges, in a container runtime environment. The program is referred to as a plugin, in this case the `bridge` is a plugin for CNI. CNI defines a set of responsibilities for container runtimes and pliugins. For container runtime CNI specifies that it is responsible for creating a namespace for each container.  


## Cluster Networking

The kubernetes cluster consist of master and worker nodes, each node must have at least one network interface connected to the private network. Each interface must have an address configured. The host must have a unique hostname set as well as a unique mac address. 

There are some ports that need to be opened as well, The master should accept connections on the `6443` for the api-server, The kubelet on master and worker nodes listen on port `10250` the kube-scheduler requires the `10251`, the kube-controller-manager requires the `10252`.

The worker nodes exposes services for external access on ports `30000` to `32767`

The ETCD server is on its own port `2379`


## Pod Networking

Kubernetes expects every pod to get its own unique IP address. Every pod should be able to reach every other pod within the same node using IP addresses. Every pod should be able to reach every other pod on other nodes as well. 

There are many networking solution out there that implement this model. 

## CNI in Kubernetes

The CNI plugin must be invoked by components within kubernetes responsible for creating containers, because that component must invoke the appropriate network plugin after the container is created. The CNI plugin is configured within kubelet service within each node in the cluster. If you look at the kubelet service file `kubelet.service` you'll see an option set to CNI `--network-plugin=cni` You can also view this option in kubelet service `ps aux | grep kubelet`.

## WeaveWorks

## IP Address Management - Weave

CNI says it is the responsibility of the CNI plugin (the network solution provider) to take care of assigning IPs to the containers. 

Kubernetes does not care how we assign IPs, we just need to make sure there's no duplicate IPs

An easy way to do it is to store the list of IPs in a file, and make sure we have the necessary code in our script to manage this file properly. This file will be placed on each host and manages the IP of the pods on each node. 

CNI comes with two built-in plugins which we can outsource this task to.
* `DHCP`
* `host-local`

We have to invoke this plugin in our script,  the CNI specification file has a section called `IPAM` which specify the type of plugin to be used. `cat /etc/cni/net.d/net-script.conf`

This details can be red from our script, to use the right plugin instead of hardcoding it. 

Weave by default allocate the IP range `10.32.0.0/12` for the entire network. 

## Service Networking

In practice you'd rarely configure pods to connect to each other, if you want a pod to access another pod you would use a service. 

In order to make orange pod accessible to the blue pod, we have to make an orange service. The orange service gets an IP address and a name assign to it, the blue pod can now access orange pod through orange service's IP or its name. 

When a service is created it is accessible from all pods in the cluster, irrespective of what nodes the pods are on. 

A clusterIP service is only accessible from within the cluster. It's useful for services such as databases. 

A nodePort service is accessible from outside of the cluster, It's useful for services such as web applications. It works just like clusterIP but in addition it exposes the application on all nodes in the cluster.

Kubelet is responsible for creating pods on each node, kubelet watches the changes in the cluster through the kube api-server. After creating pods it will invoke the CNI plugin to configure networking for that pod. Kube-proxy watches the changes in the cluster through the kube api-server and every time a new service is to be created, kube-proxy gets into action. Unlike pods, services are not limited to nodes they are a cluster-wide concept. 

When we create a service object in kubernetes, it is assigned an IP address from a predefined range, the kube-proxy gets that IP address, and create forwarding rules on each node in the cluster. Saying any traffic coming to this IP should go to the IP of the pod.  Whenever a service is created or deleted, the kube-proxy creates or deletes these rules. 


kube-proxy supports different ways such as `userspace` where it listens on a port for each service and proxies the connection to a pod. `ipvs` rules, or more familiar option is `iptables`. 

The proxy mode can be set while configuring kube-proxy service.

```bash
kube-proxy --proxy-mode [userspace | iptables | ipvs]
```
The default is `iptables` 

You can see the rules created by kube proxy in the IP table NAT table output. 

```bash
iptables -L -t nat | grep <service-name>
```

## DNS

In kubernetes each node has a nodename and IP address  assign to it, these are probably registered in a DNS server in the organization.

We focus on DNS resolution within the cluster, kubernetes deploys a builtin DNS server by default when you set up a cluster. 

Imagine we have two pods and one service, test and web pod, to make the web pod accessible to the test pod we make a service. It gets its own IP. Whenever a service is created, kubernetes DNS server creates a record for the service, it maps the service name to its IP address. So within the cluster, any pod can reach this service using its service name. 

If the web service was in a separate namespace called `apps` then to refer to it form the default namespace we would have to say `web-service.apps`
```bash
curl http://web-service.apps
```

For each namespace we would have a subdomain, all the services are group together in another subdomain called `svc`. Finally all services and pods are grouped together into a root domain for the cluster  which is set to `cluster.local` by default. So the FQDN for the service becomes `http://web-service.apps.svc.cluster.local` that is how services are resolved within the cluster. 

The FQDN for pods are not created by default, but we can enable that explicitly. For each pod kubernetes generates a name by replacing the dots with dashes. Name space will remain the same, the type will be set to `pod` and the root domain will also stay the same. For example `http://10-244-2-5.apps.pod.cluster.local`

## Core DNS

An easy way to make two pods resolve each other is to add an entry into each of their `etc/hosts` file. On the first pod we add the second pod and vice versa.

This is not a suitable solution if we have thousands of pods being created and deleted. 

A better solution is to move these entries into a central DNS server, then point these pods into the DNS server by adding a entry into their `ect/resolve.conf` file specifying the the `nameserver` is at the IP address of the DNS server. Every time a new pod is created we  add a record into the DNS server for that pod so that other pods can access the new pod. And we configure the `enc/resolve.conf` file so the new pods can resolve other pods into the cluster.

Kubernetes creates an entry in the DNS server by generating a pod name with replacing the dots in the IP of the pod with dashes. After `v1.12` the recommended DNS is CoreDNS.

The CoreDNS is deployed as a pod in the kube-system namespace, they are actually deployed as two pods for redundancy as part of a replicaset. CoreDNS requires a configuration file `etc/coredns/CoreFile` within this file we have a number of plugins configured for `health`, `metrics`, etc., the plugin that makes CoreDNS works with kubernetes is called `kubernetes`. `cluster.local` is mentioned there so every record in kubernetes fall under it. 

The `pods insecure` will enable pod name resolution by dash conversion. 

This file is also passed to pods as a `ConfigMap` object 

```bash
kubectl get configmap -n kube-system

coredns
```

For the pods to point to the DNS server, they can point to CoreDNS service,  the service is name `kube-dns` by default, the IP address of this service is configured as `nameserver` on pods. The DNS configuration on pods are done by kubernetes automatically, specifically by `kubelet` if we look at the config file of the kubelet we will see the IP of the DNS server in it:

```bash
cat /var/lib/kubelet/config.yaml

...
clusterDNS:
  - 10.96.0.10
clusterDomain: cluster.local
```

You can get the FQDN of the services or pods by running
```bash
host <service-name>
```


## Ingress

Imagine we're deploying an application that has a online selling service, say at `online-store.come` we build the application as a docker image and deploy it as a pod within a deployment in kubernetes. The application needs a database so we deploy mysql as a pod and create a service as `ClusterIP` called `mysql-service` to make the application accessible to the outside world, we create another service this time of type `NodePort` lets say at port `38080` the users can now access the application at `http://<node-ip>:38080` whenever traffic increases, we increase the number of replicas of the pod to handle the additional load, and the service takes care of splitting the traffic between pods. 

There are many more things involved in addition to splitting the traffic between the pods. For example we do not want that users to type in IP address every time. So we configure our DNS server to point to the IP of the nodes. To take care of the high-number port we bring an additional later between our DNS server and the cluster like a `proxy-server` that proxies the requests from port `80` to port `38080` on the nodes, then we point the DNS to the proxy server. Now users can simply access the `online-store.com`.  This is when our application is hosted on-prem.

If the application was hosted on a public cloud, instead of creating a service of type `NodePort` we could create a `LoadBalancer` in that case kubernetes would do everything just like the node port but in addition it would send a request to the cloud provider to provision a load balancer. On receiving the request the cloud provider will automatically deploy a load balancer configured to route traffic on the service ports on all the nodes. The load balancer has an external IP that can be provided to users to access the application. Then we point the DNS server to this IP. 

Now let's imagine we have additional service in our business, we want the users to be able to access the new service by going to `online-store.come/<new-service>` we would like to make the old application accessible at `online-store.come/<old-service>` The developers have developed the new application as a completely standalone application and it has nothing to do with the old one. We deploy the new application as a separate deployment within the cluster. We create a service as type `LoadBalancer` and kubernetes provisions port `38282` for this service and also provisions a network load balancer on the cloud, the new load balancer has a new IP.

In order to direct traffic to each od these load balancers based on the URL that the user types in, we need yet another proxy or load balancer that can redirect the traffic. Every time we introduce a new service we have to reconfigure the load balancer. Finally we also need to enable SSL for our applications so users can access the application using the HTTPS. This can be configure at many levels, either application level or proxy,load balancer etc., Ideally we'd like to configure this in one place with minimal maintenance. This becomes easily cumbersome to configure and it would be very nice to manage that within the kubernetes cluster, and have all that configurations as just another kubernetes configuration file. That's where ingress comes in.


Ingress helps your users access your application using a single externally accessible URL, hat you can configure to route to different services within your cluster based on the URL path. At the same time, implement SSL security as well. 

Think of ingress as a layer 7 load balancer builtin to the kubernetes cluster, that can be configured using native kubernetes primitives just like any other object in kubernetes. 

Even with ingress we still need to expose it to make it accessible outside of the cluster, so we still have to publish it as node port or with a cloud native load balance, but that is just a one-time configuration. In the future we apply all the load balancing, URL based routing, SSL etc., on the ingress controller. 


Ingress controller is principally a reverse proxy like `Nginx`, `HAPROXY` or `Traefik` with set of rules to configure the ingress. The solution we deploy is called ingress controller and the rules are called ingress resources. A kubernetes cluster does not come with ingress controller by default. 

To deploy a `nginx` ingress controller we start with a deployment definition file:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
    template:
      metadata:
        labels:
          name: nginx-ingress
      spec:
        containers:
          - name: nginx-ingress-controller
            image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
        args:
          - /nginx-ingress-controller
        env:
          - name: POD_NAME
            valueFrom: 
              fieldRef:
                filedPath: metadata.name
          - name: POD_NAMESPACE
            valueFrom: 
              fieldRef:
                filedPath: metadata.namespace
        ports:
          - name: http
            containerPort: 80
          - name: https
            containerPort: 443
```

This a special build of nginx specifically to be used as ingress controller.

Nginx has specific set of configurations such as `err-log-path`, `keep-alive` and `ssl-protocols` for these we must create a config map and pass them in.

We must also create two environment variables to carry pod's name and namespace that is deployed to. The nginx service requires this to read the configuration data from within the pod. 


We then need a service to expose the ingress controller to the external world, so we create a service of type `NodePort`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-ingress
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  selector:
    name: nginx-ingress
```

The ingress controllers have additional intelligence builtin to them to monitor kubernetes cluster for ingress resources and configure the underlying nginx server when something is changed. But for ingress controller to do this it requires a service account with the right set of permissions. For that we create a service account with the correct roles and rolebindings.

Next we have to create ingress resources. An ingress resource is a set of rules applied on ingress controller.  We can configure it to say simply route all incoming traffic to a specific application, or route traffic to a different application based on a URL, the configuration is as follows:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-<service-name>
spec:
  backend:
    serviceName: <service-name>
    servicePort: 80
```
After creation we can view the resource by

```bash
kubectl get ingress
```

To create rules we start with a similar definition file, for instance the rules for routing traffic based on URL path is as follows:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-<service-name>
spec:
  rules:
  - http: 
    paths:
    pathType: prefix
    - path: /<new-service>
      backend:
        service:
          name: <new-service-name>
          port:
            number: 80
    - path: /<old-service>
      backend:
        serviceName: <old-service-name>
        servicePort: 80
```

The default backend is when user tries to reach a URL that is not specified in rules.

If we want to route users based on domain names we have to create separate rules:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-<service-name>
spec:
  rules:
  - host: <new-service>.online-store.com
    http:
      paths:
        backend:
          service:
            name: <new-service-name>
            port:
              number: 80
  - host: <old-service>.online-store.com
    http:
      paths:
        backend:
          service:
            name: <new-service-name>
            port:
              number: 80
```

n k8s version 1.20+ we can create an Ingress resource from the imperative way like this:-
```bash
kubectl create ingress <ingress-name> --rule="host/path=service:port"
```

## Ingress - Annotations and rewrite-target
Different ingress controllers have different options that can be used to customize the way it works. NGINX Ingress controller has many options that can be seen here. 

### The Rewrite target option.



Our watch app displays the video streaming webpage at `http://<watch-service>:<port>/`

Our wear app displays the apparel webpage at `http://<wear-service>:<port>/`

We must configure Ingress to achieve the below. When user visits the URL on the left, his request should be forwarded internally to the URL on the right. Note that the `/watch` and `/wear` URL path are what we configure on the ingress controller so we can forwarded users to the appropriate application in the backend. The applications don't have this URL/Path configured on them:

`http://<ingress-service>:<ingress-port>/watch` --> `http://<watch-service>:<port>/`

`http://<ingress-service>:<ingress-port>/wear` --> `http://<wear-service>:<port>/`



Without the rewrite-target option, this is what would happen:

`http://<ingress-service>:<ingress-port>/watch` --> `http://<watch-service>:<port>/watch`

`http://<ingress-service>:<ingress-port>/wear` --> `http://<wear-service>:<port>/wear`



Notice watch and wear at the end of the target URLs. The target applications are not configured with `/watch` or `/wear` paths. They are different applications built specifically for their purpose, so they don't expect `/watch` or `/wear` in the URLs. And as such the requests would fail and throw a `404 not found` error.



To fix that we want to "ReWrite" the URL when the request is passed on to the watch or wear applications. We don't want to pass in the same path that user typed in. So we specify the rewrite-target option. This rewrites the URL by replacing whatever is under `rules->http->paths->path` which happens to be `/pay` in this case with the value in rewrite-target. This works just like a search and replace function.

For example: `replace(path, rewrite-target)`
In our case: `replace("/path","/")`


```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /pay
        backend:
          serviceName: pay-service
          servicePort: 8282
```

In another example given here, this could also be:

`replace("/something(/|$)(.*)", "/$2")`
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: rewrite
  namespace: default
spec:
  rules:
  - host: rewrite.bar.com
    http:
      paths:
      - backend:
          serviceName: http-svc
          servicePort: 80
        path: /something(/|$)(.*)
```