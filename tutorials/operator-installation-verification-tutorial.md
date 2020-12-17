---
title: Prometheus Operator Installation Verification Tutorial
description: Check Operator Deployment Status
---

### Check the Prometheus Operator 

After installation, verify that your operator got successfully installed by executing the below command.

```execute
kubectl get csv -n operators
```

You should see a similar output as below.

```output
NAME                        DISPLAY               VERSION   REPLACES                    PHASE
prometheusoperator.0.37.0   Prometheus Operator   0.37.0    prometheusoperator.0.32.0   Succeeded
```

**Please wait till `PHASE` status will be `Succeeded` and then proceed further.**

After the installation is successful , you can check your operator pods by executing the below command.

```execute
kubectl get pods -n operators
```

You should see a pod starting with 'prometheus-operator' with Ready value '1/1' and Status 'Running' like the output below.

```output
NAME                                   READY   STATUS    RESTARTS   AGE
prometheus-operator-6f7589ff7f-hswnz   1/1     Running   0          6m48s
```

