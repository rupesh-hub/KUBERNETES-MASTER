vim service.yaml
---

```yaml
kind: Service
apiVersion: v1
metadata:
  name: nginx
  namespace: nginx
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP

```

```shell
kubectl apply -f service.yaml
kubectl get all -n nginx
kubectl port-forward service/nginx -n nginx 80:808 --address=0.0.0.0
```