HPA -> Replication --> Increasing the number of pods 
VPA -> Increasing in Limit no replication - increasing the power of pods

```shell
kubectl top pod
kubectl top node
```

namespace.yaml
---
```yaml
kind: Namespace
apiVersion: v1
metadata:
  name: apache
```

deployment.yaml
--
```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: apache-deployment
  namespace: apache
spec:
  replicas: 1
  selector:
    matchLabels:
      app: apache

  template:
    metadata:
      labels:
        app: apache
      name: apache
    spec:
      containers:
        - name: apache
          image: httpd:latest
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 200m
              memory: 256Mi
```

service.yaml
---
```yaml
kind: Service
apiVersion: v1
metadata:
  name: apache-service
  namespace: apache
spec:
  selector:
    app: apache
  ports:
    - port: 80 # Exposed posed in cluster
      protocol: TCP
      targetPort: 80 # Container port
  type: ClusterIP

```
```shell
kubectl get all -n apache
kubectl port-forward service/apache-service -n apache 82:80 --address=0.0.0.0
sudo -E kubectl port-forward service/apache-service -n apache 82:80 --address=0.0.0.0
kubectl scale deployment apache-deployment -n apache --replicas=3
```

hpa.yaml
--

```yaml
apiVersion: autoscaling/v2 
kind: HorizontalPodAutoscaler
metadata:
  name: apache-hpa
  namespace: apache
spec:
  scaleTargetRef:
    kind: Deployment
    name: apache-deployment
    apiVersion: apps/v1
  minReplicas: 1
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu 
        target:
          type: Utilization
          averageUtilization: 5
```

```shell
kubectl get hpa -n apache
kubectl run -i --tty load-generator --image=busybox -n apache /bin/sh

while true; do wget -q -O- http://apache-service.apache.svc.cluster.local; done
```


Note Affinity
---
```markdown
Node Affinity is a set of rules that tells Kubernetes which nodes your Pods should be scheduled on, based on labels on those nodes. 
Think of it as a more powerful and flexible version of nodeSelector

There are two main types:
- Required (requiredDuringSchedulingIgnoredDuringExecution): The scheduler must find a node that matches the rule. If it can't, the Pod won't run (it stays "Pending").
- Preferred (preferredDuringSchedulingIgnoredDuringExecution): The scheduler will try to find a matching node, but if none are available, it will schedule the Pod anyway.

Example: Nginx on "High-Performance" Nodes
In this example, we want to ensure Nginx only runs on nodes labeled as size: large, and we prefer it if those nodes are in the availability-zone: us-east-1a.
1. Label your nodes
   First, you would label your physical/virtual nodes:
```
```shell
    kubectl label nodes node-01 size=large
    kubectl label nodes node-01 availability-zone=us-east-1a
```

```markdown
2. Nginx Deployment with Affinity
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-affinity
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      affinity:
        nodeAffinity:
          # HARD RULE: Must be on a large node
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: size
                    operator: In
                    values:
                      - large
          # SOFT RULE: Prefer us-east-1a
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 1 # Priority (1-100)
              preference:
                matchExpressions:
                  - key: availability-zone
                    operator: In
                    values:
                      - us-east-1a
      containers:
      - name: nginx
        image: nginx:latest
```

```markdown
Key Operators
Node affinity allows you to use logic beyond just "equals":
In: Label value is in a list (used above).
NotIn: Label value is not in a list (useful for avoiding specific nodes).
Exists: The node must have the label, regardless of its value.
DoesNotExist: The node must NOT have the label.
Gt / Lt: Greater than / Less than (for numeric values).

Note: The "IgnoredDuringExecution" part of the name means that if the node's labels change after the Pod is already
running, Kubernetes won't kick the Pod off. It only checks these rules during the initial scheduling.
```
