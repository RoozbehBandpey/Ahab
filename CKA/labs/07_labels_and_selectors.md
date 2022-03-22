# Labels and Selectors

We have deployed a number of PODs. They are labelled with tier, env and bu. How many PODs exist in the dev environment?
```bash
kubectl get pods --selector env=dev --show-labels
or
kubectl get pods --selector env=dev --no-headers | wc -l
```
How many PODs are in the finance business unit (bu)?
```bash
kubectl get pods --selector bu=finance --no-headers | wc -l
```

How many objects are in the prod environment including PODs, ReplicaSets and any other objects?
```bash
kubectl get all --selector env=prod --no-headers | wc -l
```

Identify the POD which is part of the prod environment, the finance BU and of frontend tier?

```bash
kubectl get pods --selector env=prod,bu=finance,tier=frontend

NAME          READY   STATUS    RESTARTS   AGE
app-1-zzxdf   1/1     Running   0          6m17s
```

A ReplicaSet definition file is given replicaset-definition-1.yaml. Try to create the replicaset. There is an issue with the file. Try to fix it.

```bash
kubectl apply  -f replicaset-definition-1.yaml
The ReplicaSet "replicaset-1" is invalid: spec.template.metadata.labels: Invalid value: map[string]string{"tier":"nginx"}: `selector` does not match template `labels`
```

set the replicaset label to pod label from template section