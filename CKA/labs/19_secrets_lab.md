# Secrets Lab

How many secrets are defined in the default-token secret?
```bash
kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-gbth2   kubernetes.io/service-account-token   3      13m

controlplane ~ ➜  kubectl describe secrets default-token-gbth2
Name:         default-token-gbth2
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: 1ca4b6cd-f88e-4c6c-bdf9-ac84cd7126d3

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     566 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6ImVDS1lyVGdjSXhuVWY4cnlGX0tLN29oQ0I0TDhZaW9NdGtWbnZsU1NNY0EifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4tZ2J0aDIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjFjYTRiNmNkLWY4OGUtNGM2Yy1iZGY5LWFjODRjZDcxMjZkMyIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.Ij1W_EPlUdsmzbNFc-_vgYC7yDIWLT6di_gOTKvwzfU6U0nzuytN3TiwS3Edb0xFhBqmiTUp4a9t0h-Ie3ayvThu9DNwm1sP_R7LTLFgkifRdJmo1k_nYF0DexWfW3moYw1055BxFd4zQyOAVsV33u4wOMAPrKiyYMbYmCwFRjqN2oIjjvGScTVdOg22tvxy2ShBESC68_E3vWXAa7HMizUME02wSoMHTLC2p158jdfcS0QXer25P_ylb6mn1_Bwa4ZEVrBcB4HEn1gctDM8ebsyrPoERPNKlW2IBA2DxRWhtGbNS_bLWv08b4VglQTY_PQyKg1Y3pnk5ar5vjmx8A
```
Create a new secret named db-secret with the data given below.



> You may follow any one of the methods discussed in lecture to create the secret.


* Secret Name: db-secret
* Secret 1: DB_Host=sql01
* Secret 2: DB_User=root
* Secret 3: DB_Password=password123
```bash
echo 'sql01' | base64
c3FsMDEK

controlplane ~ ➜  echo 'root' | base64
cm9vdAo=

controlplane ~ ➜  echo 'password123' | base64
cGFzc3dvcmQxMjMK
```
```bash
kubectl create secret generic db-secret --from-literal=DB_Host=c3FsMDEK --from-literal=DB_User=cm9vdAo= --from-literal=DB_Password=cGFzc3dvcmQxMjMK
```
> The test won't pass becasue of base64 encoding
```bash
kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123
```
Configure webapp-pod to load environment variables from the newly created secret.



> Delete and recreate the pod if required.

```bash
kubectl get pods webapp-pod -o yaml > webapp.yaml
controlplane ~ ➜  kubectl delete pods webapp-pod 
pod "webapp-pod" deleted

ls
sample.yaml  webapp.yaml

controlplane ~ ➜  vi webapp.yaml 

controlplane ~ ➜  kubectl apply -f webapp.yaml 
error: error validating "webapp.yaml": error validating data: ValidationError(Pod.spec.containers[0].envFrom[0]): unknown field "name" in io.k8s.api.core.v1.EnvFromSource; if you choose to ignore these errors, turn validation off with --validate=false

controlplane ~ ✖ vi webapp.yaml 

controlplane ~ ➜  kubectl apply -f webapp.yaml 
pod/webapp-pod created
```