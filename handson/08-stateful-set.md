stateful set
----
mysql
---
namespace.yaml
---
```yaml
kind: Namespace
apiVersion: v1
metadata:
  name: mysql
```

```shell
kubectl apply -f namespace.yaml
```


mysql-stateful-set.yaml
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  namespace: mysql
spec:
  serviceName: mysql-service
  replicas: 1   # ⚠️ use 1 unless replication is configured
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8.0
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: root-password
            - name: MYSQL_DATABASE
              value: devops
          volumeMounts:
            - name: mysql-data
              mountPath: /var/lib/mysql
  volumeClaimTemplates:
    - metadata:
        name: mysql-data
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 1Gi
```

```yaml
kind: Service
apiVersion: v1s
metadata:
  name: mysql-service
  namespace: mysql
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
    - port: 3306
      name: mysql
      targetPort: 3306
      protocol: TCP
```