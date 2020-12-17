---
title: Prometheus Instance Creation tutorial
description: This tutorial explains how create Instances for Prometheus
---
### Create this CR which will create a Prometheus Instance

```execute
cat <<'EOF' > prometheusInstance.yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: server
  labels:
    prometheus: k8s
  namespace: operators
spec:
  replicas: 1
  serviceAccountName: prometheus-k8s
  securityContext: {}
  serviceMonitorSelector:
    matchLabels:
      app: playground  
  alerting:
    alertmanagers:
      - namespace: operators
        name: alertmanager-main
        port: web  
EOF
```

Execute below command to create Prometheus instance 

```execute
kubectl create -f prometheusInstance.yaml -n operators
```

You will see the following resources created:

```output
prometheus.monitoring.coreos.com/prometheus created
```

Get the associated Pods:

```execute
kubectl get pods -n operators
```
You will be able to see pods:

```output
NAME                                   READY   STATUS    RESTARTS   AGE
prometheus-operator-6f7589ff7f-hswnz   1/1     Running   0          23m
prometheus-prometheus-0                3/3     Running   1          68s
prometheus-prometheus-1                3/3     Running   1          68s
```


- Create the service to access prometheus server


```execute
cat <<'EOF' > prometheus_service.yaml
apiVersion: v1
kind: Service
metadata:
  name: prometheus
spec:
  type: NodePort
  ports:
  - name: web
    nodePort: 30100
    port: 9090
    protocol: TCP
    targetPort: web
  selector:
    app: prometheus
EOF
```

- Execute below command to create Prometheus Service:

```execute
kubectl create -f prometheus_service.yaml -n operators
```

- Access Prometheus dashboard with below url :


Click on <a href="http://##DNS.ip##:30100" target="_blank">http://##DNS.ip##:30100</a> to access Prometheus dashboard from your browser.

You will see the Prometheus dashboard as below :

![prometheus-page](_images/prom.png)


