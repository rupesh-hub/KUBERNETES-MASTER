# Creating namespace, deployment, service
vim namespace.yaml
```yaml
kind: Namespace
apiVersion: v1
metadata:
  name: nginx
```

```yaml
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

```yaml
kubectl apply -f pod.yaml
kubectl get pods -n nginx
kubectl exec -it pod/nginx -n nginx -- bash
kubectl describe pod/nginx -n nginx
```

```yaml
Events:
  Type    Reason     Age    From               Message
  Normal  Scheduled  2m21s  default-scheduler  Successfully assigned nginx/nginx to alfarays-cluster-worker2
  Normal  Pulling    2m20s  kubelet            Pulling image "nginx:latest"
  Normal  Pulled     2m4s   kubelet            Successfully pulled image "nginx:latest" in 16.198s (16.198s including waiting). Image size: 59795293 bytes.
  Normal  Created    2m4s   kubelet            Created container nginx
  Normal  Started    2m4s   kubelet            Started container nginx
```

