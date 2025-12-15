** Steps To Follow **
Apply the configuration:


ingress.yaml
---

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  namespace: nginx
  annotations: 
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - http:
        paths:
          - path: /nginx
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: notes-app-service
                port:
                  number: 8000
```

```shell
kubectl apply -f ingress.yaml
kubectl get ingress -n nginx
kubectl get all -n nginx
kubectl get service -n ingress-nginx
kubectl port-forward service/ingress-nginx-controller -n ingress-nginx 8080:80 --address=0.0.0.0
```