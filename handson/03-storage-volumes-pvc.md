Persistent Volume & Persistent Volume Claims

persistent-volume.yaml
----
```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: local-pv
  namespace: nginx
  labels:
    app: local
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  hostPath:
    path: /mnt/data
```

```shell
kubectl apply -f persistent-volume.yaml
kubectl get pv
```

persistent-volume-claim.yaml
----
```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: local-pvc
  namespace: nginx
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-storage
```

```shell
kubectl apply -f persistent-volume-claim.yaml
kubectl get pv
kubectl get pvc
```

Assigning pvc to pod/deployment
---
```shell
kubectl depelet -f deployment.yaml

```

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nginx
  namespace: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
          volumeMounts:
            - mountPath: /var/www/html
              name: my-volume
      volumes:
        - name: my-volume #any
          persistentVolumeClaim:
            claimName: local-pvc
```

```shell
kubectl apply -f deployment.yaml
kubectl get pods -n nginx
kubectl describe pod/...
```


