config-map
----

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: mysql-config-map
  namespace: mysql
data:
  MYSQL_DATABASE: "devops"

```

```shell
kubectl apply -f config-map.yaml
kubectl get config-map
```

secrets.yaml
---

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: mysql
data:
  MYSQL_ROOT_PASSWORD: base_64_val
```

mysql-stateful-set.yaml
----

```yaml
kind: StatefulSet
apiVersion: apps/v1
metadata:
  name: mysql
  namespace: mysql
spec:
  serviceName: mysql-service
  replicas: 1
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
          
          livenessProbe: # Liveness and readiness probe
            tcpSocket:
              port: 3306
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            tcpSocket:
              port: 3306
            initialDelaySeconds: 5
            periodSeconds: 10

          resources: # Request Quota
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 200m
              memory: 256Mi

          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_ROOT_PASSWORD
            - name: MYSQL_DATABASE
              valueFrom:
                configMapKeyRef:
                  name: mysql-config-map
                  key: MYSQL_DATABASE
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
apiVersion: v1
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

```shell
echo -n "root" | base64
kubectl get pods -n mysql
kubectl exec -it mysql-0 -n mysql -- bash
mysql -u root -p

# Scale down to stop the pod
kubectl scale statefulset mysql --replicas=0 -n mysql

# Delete the Persistent Volume Claim (this deletes the data!)
kubectl delete pvc mysql-data-mysql-0 -n mysql

# Scale back up to trigger a fresh initialization
kubectl scale statefulset mysql --replicas=1 -n mysql
```

```markdown
Probes:
1. Liveness probe
2. Readiness probe
3. Startup Probe

```

```shell
Taints and Tolerance
kubectl get nodes
kubectl taint node alfarays-cluster-worker2 prod=true:NoSchedule #taint
kubectl taint node alfarays-cluster-worker2 prod=true:NoSchedule- #untaint

Tolerance Example
---
apiVersion: v1
metadata:
  name: nginx-pod
  namespace: nginx
spec:
  containers:
    - name: nginx
      image: nginx:latest 
      ports:
      - containerPort: 80
  tolerations:
    - key: "prod"
      operator: "Equal"
      effect: "NoSchedule"

```