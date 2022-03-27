# Logging and Monitoring

## Monitoring Cluster Components

For monitoring resource consumption on kubernetes we would like to know:
* Node-level metrics: such as the number of nodes in the cluster, how meany of them are healthy as well as cpu, memory, disk and network utilization metrics.
* Pod-level metrics: such as number of pods, performance metrics of each pods like cpu and memory consumptions

Kubernetes does not come with full feature built-in monitoring solution, but there are number of open source solution available:
* Metrics Server: Is the newer version of Heapster (deprecated) we can have one metrics server per kubernetes cluster, it retrieves metrics from each of the kubernetes nodes and pods, aggregates them and stores them in memory.
    * This is an in-memory-only monitoring solution and does not store the metrics on the disk. Therefore we cannot see historical performance data.
    * The kubelet is responsible for receiving instruction from kube-api server and running pods on the nodes, the kunelet also has a sub-component known as cAdvisor (or container advisor) it is responsible for collecting performance metrics from the pods and expose them through kubelet api make them available for metrics server
    * If you're using minikube run the command `minikube addons enable metrics-server`, for other environments it has to be installed. once deployed git it some time to collect data then view the metrics by running `kubectl top node` or `kubectl top pod`
* Prometheus
* Elastic Stack
* DataDog
* Dynatrace

