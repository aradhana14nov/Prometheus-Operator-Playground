---
title: Prometheus Monitoring
description: This tutorial explains how Prometheus monitors targets
---

### Prometheus Monitoring


Prometheus is designed to monitor targets. Servers, databases, standalone virtual machines and many mores can be monitored with Prometheus.

Prometheus monitoring feature :

- Prometheus is a pull based monitoring system.

- Prometheus actively screens targets in order to retrieve metrics from them.

- In order to monitor systems, Prometheus will periodically scrap them.

- Prometheus expects to retrieve metrics via HTTP calls done to endpoints that are defined in Prometheus configuration.


Lets take the example of MariaDB Server. Here, we will explain the monitoring of MariaDB Sever using Prometheus. 


### Prometheus monitoring steps to MariaDB Server


step1:  Install the MariaDB operator by running the following command:


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


step2: Create below CR which will create a MariaDB database called test-db, along with user credentials.

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

- Execute below command to create MariaDBserver instance :

```execute
kubectl create -f MariaDBserver.yaml -n my-mariadb-operator-app 
```


Steps3 : Lets see how we can access MariaDB Database. 

- Copy below command and execute by adding MariaDB Server Instance podname.


```copycommand
kubectl exec -it <podname> bash -n my-mariadb-operator-app
```


- login through db-user 

```execute
mysql -h ##DNS.ip## -P 30685 -u db-user -pdb-user
```

- check the the created database 

```execute
show databases;
```

- exit in order to login through root user, if you want to create some DB Tables.

```execute
exit
```

- Login through root user


```execute
mysql -h ##DNS.ip## -P 30685 -u root -ppassword
```


- Create database

```execute
create database testdb;
```

- Use already created database in above steps 

```execute
use testdb;
```

- create table 

```execute
create table names (name VARCHAR(100));
```
- Insert data 

```execute
insert into names values('Abhijit');
```
- Retrieve data from table

```execute
select * from names;
```

```execute
exit
```


Step4: Create below CR for MariaDB Monitoring services.

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

Note: The database host and port should be correct for metrics to work.


- Execute below command to Create Instance of Monitoring 

```execute
kubectl create -f MariaDBmonitoring.yaml -n my-mariadb-operator-app
```

This CR will start Prometheus exporter pod and service. 




Step 5: Create below CR which will create Instance of ServiceMonitor to monitor MariaDB Services:


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

Step 6 : Access the Prometheus service using below link. 

```
http://##DNS.ip##:30100
```

On the prometheus UI, you will see the monitoring targets inside Status -> Targets.
You will see the monitoring metric for the MariaDB endpoint at http://##DNS.ip##:30100/metrics




