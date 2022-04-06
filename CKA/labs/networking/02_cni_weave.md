# CNI Weave

Inspect the kubelet service and identify the network plugin configured for Kubernetes.
```bash
ps -aux | grep kubelet | grep --color network-plugin= 
root      4668  0.0  0.0 4150044 107036 ?      Ssl  20:57   0:26 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.2
```
What is the path configured with all binaries of CNI supported plugins?

The CNI binaries are located under `/opt/cni/bin` by default.


Identify which of the below plugins is not available in the list of available CNI plugins on this host?
```bash
ls /opt/cni/bin
bandwidth  dhcp      flannel      host-local  loopback  portmap  sbr     tuning
bridge     firewall  host-device  ipvlan      macvlan   ptp      static  vlan
```
What is the CNI plugin configured to be used on this kubernetes cluster?
```bash
ls /etc/cni/net.d/
10-flannel.conflist
```
What binary executable file will be run by kubelet after a container and its associated namespace are created.
```bash
cat /etc/cni/net.d/10-flannel.conflist
{
  "name": "cbr0",
  "cniVersion": "0.3.1",
  "plugins": [
    {
      "type": "flannel",
      "delegate": {
        "hairpinMode": true,
        "isDefaultGateway": true
      }
    },
    {
      "type": "portmap",
      "capabilities": {
        "portMappings": true
      }
    }
  ]
}
```