# Logging Lab
We have deployed a POD hosting an application. Inspect it. Wait for it to start.
```bash
kubectl get pods --watch
NAME       READY   STATUS    RESTARTS   AGE
webapp-1   1/1     Running   0          33s
```
A user - USER5 - has expressed concerns accessing the application. Identify the cause of the issue.



Inspect the logs of the POD
```bash
kubectl logs -f webapp-1 
[2022-03-27 12:12:46,354] INFO in event-simulator: USER2 is viewing page2
[2022-03-27 12:12:47,355] INFO in event-simulator: USER1 logged in
[2022-03-27 12:12:48,356] INFO in event-simulator: USER4 is viewing page1
[2022-03-27 12:12:49,357] INFO in event-simulator: USER3 logged out
[2022-03-27 12:12:50,359] INFO in event-simulator: USER3 is viewing page1
[2022-03-27 12:12:51,360] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-03-27 12:12:51,360] INFO in event-simulator: USER4 is viewing page1
...
```

A user is reporting issues while trying to purchase an item. Identify the user and the cause of the issue.



Inspect the logs of the webapp in the POD
```bash
kubectl logs -f webapp-2
error: a container name must be specified for pod webapp-2, choose one of: [simple-webapp db]
```
```bash
kubectl logs -f webapp-2 simple-webapp
[2022-03-27 12:15:19,452] INFO in event-simulator: USER4 is viewing page1
[2022-03-27 12:15:20,453] INFO in event-simulator: USER1 is viewing page3
[2022-03-27 12:15:21,455] INFO in event-simulator: USER2 logged out
[2022-03-27 12:15:22,456] INFO in event-simulator: USER3 is viewing page2
[2022-03-27 12:15:23,458] INFO in event-simulator: USER3 is viewing page1
[2022-03-27 12:15:24,459] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-03-27 12:15:24,460] INFO in event-simulator: USER1 is viewing page1
[2022-03-27 12:15:25,461] INFO in event-simulator: USER3 logged in
[2022-03-27 12:15:26,463] INFO in event-simulator: USER4 logged out
[2022-03-27 12:15:27,463] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
...
```