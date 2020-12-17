---
title: Prometheus Monitoring
description: This tutorial explains how Prometheus monitors targets/endpoints
---

### Prometheus Monitoring


Prometheus is designed to monitor targets. Servers, databases, standalone virtual machines etc can be monitored with Prometheus.

Prometheus monitoring feature :

- Prometheus is a pull based monitoring system.

- Prometheus actively screens targets in order to retrieve metrics from them.

- In order to monitor systems, Prometheus will periodically scrap them.

- Prometheus expects to retrieve metrics via HTTP calls done to endpoints that are defined in Prometheus configuration.


Lets take the example of MariaDB Server. Here, we will explain the monitoring of MariaDB Sever using Prometheus. 

### How to monitor MariaDB server using Prometheus 

Step 1:  Install the MariaDB operator by running the following command:


```execute
kubectl create -f https://operatorhub.io/install/mariadb-operator-app.yaml             
```

- watch your operator come up using below command :


```execute
kubectl get csv -n my-mariadb-operator-app
```

- Get the associated Pods using below command:


```execute
kubectl get pods -n my-mariadb-operator-app
```


Step 2: To create MariaDB database called test-db along with user credentials , create the below yaml definition of the Custom Resource.

```execute
cat <<'EOF' > MariaDBserver.yaml
apiVersion: mariadb.persistentsys/v1alpha1
kind: MariaDB
metadata:
  name: mariadb
spec:
  # Keep this parameter value unchanged.
  size: 1
  
  # Root user password
  rootpwd: password

  # New Database name
  database: test-db
  # Database additional user details (base64 encoded)
  username: db-user 
  password: db-user 

  # Image name with version
  image: "mariadb/server:10.3"

  # Database storage Path
  dataStoragePath: "/mnt/data" 

  # Database storage Size (Ex. 1Gi, 100Mi)
  dataStorageSize: "1Gi"

  # Port number exposed for Database service
  port: 30685
EOF
```

- Execute below command to create an instance of MariaDBserver using the above yaml definition:

```execute
kubectl create -f MariaDBserver.yaml -n my-mariadb-operator-app 
```

- Check pods status :

```execute
kubectl get pods -n my-mariadb-operator-app
```

Step 3 : Access MariaDB Database. 

- Connect to MariaDB Server pod.

 - Copy below command to the terminal,add the podname of MariaDB Server Instance.
    
 ```copycommand
 kubectl exec -it <podname> bash -n my-mariadb-operator-app
 ```


 - Connect to the database using username **db-user** and password **db-user**


 ```execute
 mysql -h ##DNS.ip## -P 30685 -u db-user -pdb-user
 ```


- list database

```execute
show databases;
```


- exit the database.


```execute
exit
```


- To login through root user use below command:


```
mysql -h ##DNS.ip## -P 30685 -u root -ppassword
```





Step 4: Enable monitoring service for MariaDB Server.


- Execute below command to get services of MariadB:
  
  ```execute
  kubectl get svc -n my-mariadb-operator-app
  ```

 Output:
 ```
      NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
 mariadb-operator-metrics   ClusterIP   10.111.158.91    <none>        8383/TCP,8686/TCP   3d4h
 mariadb-service            NodePort    10.106.178.202   <none>        80:30685/TCP        3d4h
 ```
 
- Get the port of mariadb-service :
  
  From above command output, mariadb-service port is 30685 

- To enable monitoring using Prometheus exporter pod and service , create the below yaml definition of the Custom Resource.

```execute
cat <<'EOF'> MariaDBmonitoring.yaml
apiVersion: mariadb.persistentsys/v1alpha1
kind: Monitor
metadata:
  name: mariadb-monitor
spec:
  # Add fields here
  size: 1
  # Database source to connect with for colleting metrics
  # Format: "<db-user>:<db-password>@(<dbhost>:<dbport>)/<dbname>">
  # Make approprite changes 
  dataSourceName: "root:password@(##DNS.ip##:30685)/test-db"
  # Image name with version
  # Refer https://registry.hub.docker.com/r/prom/mysqld-exporter for more details
  image: "prom/mysqld-exporter"
EOF
```

Note: The database host and port should be correct for metrics to work. Host will be cluster IP and the port will be mariadb-service port(see Step 4).


- Execute below command to Create Instance of Monitoring 

```execute
kubectl create -f MariaDBmonitoring.yaml -n my-mariadb-operator-app
```

This will start Prometheus exporter pod and service. 



- Check pods status :

```execute
kubectl get pods -n my-mariadb-operator-app
```



Step 5: Create Instance of ServiceMonitor to monitor MariaDB Services:


```execute
cat <<'EOF' > ServiceMonitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: mariadb-monitor 
  labels:
    app: playground
  namespace: operators
spec:
  namespaceSelector:
    any: true
  selector:
    matchLabels:
      tier: monitor-app 
  endpoints:
   - interval: 20s
     port: monitor    
EOF
```


- Execute below command to create ServiceMonitor instance :


```execute
kubectl create -f ServiceMonitor.yaml -n operators
```

Step 6 : Access the Prometheus dashboard using below link. 

```
http://##DNS.ip##:30100
```

On the prometheus UI, Go to Status tab. Choose option:Targets to see endpoints.





