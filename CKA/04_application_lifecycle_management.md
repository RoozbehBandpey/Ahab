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

