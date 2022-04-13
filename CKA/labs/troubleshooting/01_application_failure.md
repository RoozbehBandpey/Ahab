# Application Failure Lab

A simple 2 tier application is deployed in the alpha namespace. It must display a green web page on success. Click on the App tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

Stick to the given architecture. Use the same names and port numbers as given in the below architecture diagram. Feel free to edit, delete or recreate objects as necessary.


```bash
kubectl get pods -n alpha
NAME                            READY   STATUS    RESTARTS   AGE
webapp-mysql-84fbfc644f-5cmw5   1/1     Running   0          89s
mysql                           1/1     Running   0          89s
```
```bash
controlplane ~ ➜  kubectl get services -n alpha
NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
mysql         ClusterIP   10.43.10.164   <none>        3306/TCP         2m23s
web-service   NodePort    10.43.216.10   <none>        8080:30081/TCP   2m22s
```

Or a better command:
```bash
kubectl -n alpha get all
```
The service name used for the MySQL Pod is incorrect. According to the Architecture diagram, it should be mysql-service.
To fix this, let'stake the existing service yaml definition
```bahs
kubectl get service mysql -n alpha -o yaml > mysql-service.yaml
controlplane ~ ➜  ls
mysql-service.yaml
```

delete the current service
```bash
kubectl delete service mysql -n alpha
service "mysql" deleted
```

fix the name in yaml manifest then deploy:

```bash
kubectl apply -f mysql-service.yaml 
service/mysql-service created
```
The same 2 tier application is deployed in the beta namespace. It must display a green web page on success. Click on the App tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

Stick to the given architecture. Use the same names and port numbers as given in the below architecture diagram. Feel free to edit, delete or recreate objects as necessary.
```bash
kubectl get  pods -n beta
NAME                            READY   STATUS    RESTARTS   AGE
webapp-mysql-84fbfc644f-6r9mb   1/1     Running   0          52s
mysql                           1/1     Running   0          52s
```
```bash
controlplane ~ ➜  kubectl get services -n beta
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
mysql-service   ClusterIP   10.43.31.144    <none>        3306/TCP         68s
web-service     NodePort    10.43.123.221   <none>        8080:30081/TCP   68s
controlplane ~ ➜  kubectl describe service mysql-service
Error from server (NotFound): services "mysql-service" not found

controlplane ~ ✖ kubectl describe service -n beta mysql-service
Name:              mysql-service
Namespace:         beta
Labels:            <none>
Annotations:       <none>
Selector:          name=mysql
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.43.31.144
IPs:               10.43.31.144
Port:              <unset>  3306/TCP
TargetPort:        8080/TCP
Endpoints:         10.42.0.12:8080
Session Affinity:  None
Events:            <none>
```

If you inspect the mysql-service in the beta namespace, you will notice that the targetPort used to create this service is incorrect.
Compare this to the Architecture diagram and change it to 3306. Update the mysql-service as per the below YAML:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
  namespace: beta
spec:
    ports:
    - port: 3306
      targetPort: 3306
    selector:
      name: mysql
```

The same 2 tier application is deployed in the gamma namespace. It must display a green web page on success. Click on the App tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

Stick to the given architecture. Use the same names and port numbers as given in the below architecture diagram. Feel free to edit, delete or recreate objects as necessary.

If you inspect the mysql-service, you will see that that the selector used does not match the label on the mysql pod.
```bash
kubectl get pods -n gamma --show-labels
NAME                            READY   STATUS    RESTARTS   AGE     LABELS
mysql                           1/1     Running   0          2m45s   name=mysql
webapp-mysql-84fbfc644f-r9wlw   1/1     Running   0          2m45s   name=webapp-mysql,pod-template-hash=84fbfc644f
```

```bash
kubectl describe svc mysql-service -n gamma 
Name:              mysql-service
Namespace:         gamma
Labels:            <none>
Annotations:       <none>
Selector:          name=sql00001
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.43.154.225
IPs:               10.43.154.225
Port:              <unset>  3306/TCP
TargetPort:        3306/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>
```
As you can see the selector used is name=sql001 whereas it should be name=mysql.
Update the mysql-service to use the correct selector as per the below YAML:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
  namespace: gamma
spec:
    ports:
    - port: 3306
      targetPort: 3306
    selector:
      name: mysql
```

The same 2 tier application is deployed in the delta namespace. It must display a green web page on success. Click on the App tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

Stick to the given architecture. Use the same names and port numbers as given in the below architecture diagram. Feel free to edit, delete or recreate objects as necessary.

According to the architecture diagram, the DB_User should be root but it is set to sql-user in the webapp-mysql deployment.
Use the command kubectl -n delta edit deployments.apps webapp-mysql and update the environment variable as follows:
```yaml
spec:
      containers:
      - env:
        - name: DB_Host
          value: mysql-service
        - name: DB_User
          value: root
        - name: DB_Password
          value: paswrd
```
The same 2 tier application is deployed in the epsilon namespace. It must display a green web page on success. Click on the App tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

Stick to the given architecture. Use the same names and port numbers as given in the below architecture diagram. Feel free to edit, delete or recreate objects as necessary.

If you inspect the environment variable called MYSQL_ROOT_PASSWORD, you will notice that the value is incorrect as compared to the architecture diagram:
```bash
root@controlplane:~# kubectl -n epsilon describe pod mysql  | grep MYSQL_ROOT_PASSWORD 
      MYSQL_ROOT_PASSWORD:  passwooooorrddd
```
Correct this by deleting and recreating the mysql pod with the correct environment variable as follows:
```yaml
spec:
  containers:
  - env:
    - name: MYSQL_ROOT_PASSWORD
      value: paswrd
```
Also edit the webapp-mysql deployment and make sure that the DB_User environment variable is set to root as follows:
```yaml
spec:
      containers:
      - env:
        - name: DB_Host
          value: mysql-service
        - name: DB_User
          value: root
        - name: DB_Password
          value: paswrd
```
Once the objects are recreated, and you should be able to access the application.



The same 2 tier application is deployed in the zeta namespace. It must display a green web page on success. Click on the App tab at the top of your terminal to view your application. It is currently failed. Troubleshoot and fix the issue.

Stick to the given architecture. Use the same names and port numbers as given in the below architecture diagram. Feel free to edit, delete or recreate objects as necessary.


There are a few things wrong in this setup:

1. If you inspect the web-service, you will see that the nodePort used is incorrect.
This service should be exposed on port 30081 and NOT 30088.
```bash
kubectl -n zeta get svc
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
mysql-service   ClusterIP   10.43.73.2     <none>        3306/TCP         69s
web-service     NodePort    10.43.246.27   <none>        8080:30088/TCP   69s
```
To correct this, delete the service and recreate it using the below YAML file:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
  namespace: zeta
spec:
  ports:
  - nodePort: 30081
    port: 8080
    targetPort: 8080
  selector:
    name: webapp-mysql
  type: NodePort
```

2. Also edit the webapp-mysql deployment and make sure that the DB_User environment variable is set to root as follows:
```yaml
spec:
      containers:
      - env:
        - name: DB_Host
          value: mysql-service
        - name: DB_User
          value: root
        - name: DB_Password
          value: paswrd
```
3. The DB_Password used by the mysql pod is incorrect. Delete the current pod and recreate with the correct environment variable as per the snippet below:
```yaml
spec:
  containers:
  - env:
    - name: MYSQL_ROOT_PASSWORD
      value: paswrd
```
Once the objects are recreated, and you should be able to access the application.

