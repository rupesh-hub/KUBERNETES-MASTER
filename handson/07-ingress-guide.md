# NGINX Ingress Controller on kind (HTTP Only)

This document is a **step-by-step guide** to setting up **NGINX Ingress Controller** on a **kind (Kubernetes in Docker)** cluster using **HTTP only**.

---

## 1. Prerequisites

Ensure the following tools are installed:

* Docker
* kind
* kubectl

Verify installations:

```bash
kind version
kubectl version --client
```

---

## 2. Install NGINX Ingress Controller (kind-specific)

⚠️ Use the **kind provider manifest**.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

Verify installation:

```bash
kubectl get pods -n ingress-nginx
```

Expected output:

```text
ingress-nginx-controller-xxxxx   Running
```

---

## 4. Deploy a Sample Application

### Deployment

Create `hello-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
      - name: hello
        image: hashicorp/http-echo
        args:
          - "-text=Hello from KIND Ingress"
        ports:
          - containerPort: 5678
```

Apply:

```bash
kubectl apply -f hello-deployment.yaml
```

---

### Service

Create `hello-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-service
spec:
  selector:
    app: hello
  ports:
  - port: 80
    targetPort: 5678
```

Apply:

```bash
kubectl apply -f hello-service.yaml
```

---

## 5. Create an Ingress Resource (HTTP Only)

Create `hello-ingress.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: hello.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: hello-service
            port:
              number: 80
```

Apply:

```bash
kubectl apply -f hello-ingress.yaml
```

---

## 6. Configure Hostname Resolution

Add the following entry to your `/etc/hosts` file:

```text
127.0.0.1 hello.local
```

---

## 7. Test HTTP Access

```bash
curl http://hello.local
```

Expected response:

```text
Hello from KIND Ingress
```

You can also open in a browser:

```text
http://hello.local
```

---

## 8. Verify Ingress and Controller

Check ingress:

```bash
kubectl get ingress
kubectl describe ingress hello-ingress
```

Check controller logs:

```bash
kubectl logs -n ingress-nginx deploy/ingress-nginx-controller
```

Check service endpoints:

```bash
kubectl get endpoints hello-service
```

---

## 9. Common Issues

### Ingress not reachable

* Ensure port 80 is mapped in `kind-config.yaml`
* Ensure `/etc/hosts` is configured correctly
* Ensure ingress controller pod is running

### Ingress ignored

* Verify `ingressClassName: nginx`
* Ensure NGINX ingress controller is installed

---

## 10. Architecture Overview

```text
Client (Browser / curl)
        ↓
localhost:80
        ↓
Docker (kind node)
        ↓
NGINX Ingress Controller
        ↓
Kubernetes Service
        ↓
Pod
```

---

## 11. Cleanup

```bash
kind delete cluster --name kind-ingress
```

---

## 12. Notes

* This guide is **HTTP only** (no TLS / HTTPS)
* Suitable for local development and testing
* For production, add TLS and security features

---

✅ **NGINX Ingress Controller with HTTP on kind is now fully configured.**
