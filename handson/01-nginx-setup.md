# Creating namespace, deployment, service

vim namespace.yaml

```yaml
kind: Namespace
apiVersion: v1
metadata:
  name: nginx
```

```shell
kubectl apply -f namespace.yaml
kubectl get namespaces
```

vim pod.yaml

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: nginx
  namespace: nginx

spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

```shell
kubectl apply -f pod.yaml
kubectl get pods -n nginx
kubectl exec -it pod/nginx -n nginx -- bash
kubectl describe pod/nginx -n nginx
```

Events:
Type Reason Age From Message
Normal Scheduled 2m21s default-scheduler Successfully assigned nginx/nginx to alfarays-cluster-worker2
Normal Pulling 2m20s kubelet Pulling image "nginx:latest"
Normal Pulled 2m4s kubelet Successfully pulled image "nginx:latest" in 16.198s (16.198s including waiting). Image size:
59795293 bytes.
Normal Created 2m4s kubelet Created container nginx
Normal Started 2m4s kubelet Started container nginx

```shell
kubectl delete -f pod.yaml
```

deployment.yaml

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
```

No need to create separate pod
Deployment itself will be a pod

```shell
kubectl apply -f deployment.yaml
kubectl get deployment -n nginx
kubectl get pods -n nginx
kubectl scale deployment/nginx -n nginx --replicas=5
kubectl get pods -n nginx -o wide
kubectl set image deployment/nginx -n nginx nginx=nginx:1.27.1
```

replicaset.yanl
---

```yaml
kind: ReplicaSet
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
```

```shell
kubectl get replicasets -n nginx
```

daemon-set.yaml
---

```
    cp deployment.yaml daemon-set.yaml
```

```yaml
kind: DaemonSet
apiVersion: apps/v1
metadata:
  name: nginx
  namespace: nginx
spec:
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
```

```shell
kubectl apply -f daemon-set.yaml
kubectl get pods -n nginx # pods per worker: or node
kubectl delete -f daemon-set.yaml
```

