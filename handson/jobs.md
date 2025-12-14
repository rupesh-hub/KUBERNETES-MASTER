Job
---

```yaml
kind: Job
apiVersion: batch/v1
metadata:
  name: demo-job
  namespace: nginx
spec:
  completions: 1
  parallelism: 1
  template:
    metadata:
      name: demo-job
      labels:
        app: demo-job
    spec:
      containers:
        - name: demo-job
          image: busybox:latest
          command: [ "sh","-c", "echo Hello All! && sleep 10" ]
      restartPolicy: Never

```

```shell
kubectl apply -f job.yaml
kubectl get job -n nginx
kubectl logs pod/demo-job -n nginx
kubectl delete -f job.yaml
```

CronJob
--
vim cron-job.yaml

```yaml
kind: CronJob
apiVersion: batch/v1
metadata:
  name: every-second-minute-backup
  namespace: nginx
spec:
  schedule: "*/2 * * * *"  # every 2 minutes
  jobTemplate:
    spec:
      template:
        metadata:
          name: every-second-minute-backup
          labels:
            app: every-second-minute-backup
        spec:
          containers:
            - name: every-second-minute-backup
              image: busybox:latest
              command:
                - sh
                - -c
                - >
                  echo "Backup started..." ;
                  mkdir -p /backups && mkdir -p /data &&
                  cp -r /data /backups &&
                  echo "Backup completed!" ;
              volumeMounts:
                - mountPath: /data
                  name: data-volume
                - name: backup-volume
                  mountPath: /backups
          restartPolicy: OnFailure
          volumes:
            - name: data-volume
              hostPath:
                path: /demo-data
                type: DirectoryOrCreate
            - name: backup-volume
              hostPath:
                path: /backups
                type: DirectoryOrCreate

```

```shell
kubectl apply -f cron-job.yaml
```