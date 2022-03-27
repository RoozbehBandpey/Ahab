# Monitoring Lab

Let us deploy metrics-server to monitor the PODs and Nodes. Pull the git repository for the deployment files.




Run: git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git


```bash
git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git
Cloning into 'kubernetes-metrics-server'...
remote: Enumerating objects: 24, done.
remote: Counting objects: 100% (12/12), done.
remote: Compressing objects: 100% (12/12), done.
remote: Total 24 (delta 4), reused 0 (delta 0), pack-reused 12
Unpacking objects: 100% (24/24), done.
root@controlplane:~# ls
kubernetes-metrics-server  sample.yaml
root@controlplane:~# kubectl apply -f kubernetes-metrics-server/
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
serviceaccount/metrics-server created
deployment.apps/metrics-server created
service/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
```
```bash
kubectl top nodes
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
controlplane   395m         1%     1249Mi          0%        
node01         59m          0%     338Mi           0%      
```
```bash
kubectl top pods
NAME       CPU(cores)   MEMORY(bytes)   
elephant   22m          32Mi            
lion       1m           18Mi            
rabbit     111m         253Mi          
```
