# LIGHTNING LAB 2

## Q1:

We have deployed a few pods in this cluster in various namespaces. Inspect them and identify the pod which is not in a Ready state. Troubleshoot and fix the issue.

Next, add a check to restart the container on the same pod if the command ls /var/www/html/file_check fails. This check should start after a delay of 10 seconds and run every 60 seconds.

`kubectl get pod nginx1401 -n dev1401 -o yaml > q1.yaml`

Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx1401
  namespace: dev1401
spec:
  containers:
  - image: kodekloud/nginx
    name: nginx
    ports:
    - containerPort: 9080
      protocol: TCP
    readinessProbe:
      failureThreshold: 3
      httpGet:
        path: /
        port: 9080
        scheme: HTTP
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 1
    livenessProbe:
      exec:
        command:
        - ls /var/www/html/file_check
      initialDelaySeconds: 10
      periodSeconds: 60
```

## Q2:

Create a cronjob called dice that runs every one minute. Use the Pod template located at /root/throw-a-dice. The image throw-dice randomly returns a value between 1 and 6. The result of 6 is considered success and all others are failure.
The job should be non-parallel and complete the task once. Use a backoffLimit of 25.
If the task is not completed within 20 seconds the job should fail and pods should be terminated.

`kubectl create cronjob dice --image=kodekloud/throw-dice --schedule="*/1 * * * *" --dry-run -o yaml > q2.yaml`

Cron Job:
```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: dice
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      completions: 1
      backoffLimit: 25
      activeDeadlineSeconds: 20
      template:
        spec:
          containers:
          -  image: kodekloud/throw-dice
             name: throw-dice
          restartPolicy: OnFailure
```

## Q3:

Create a pod called my-busybox in the dev2406 namespace using the busybox image. The container should be called secret and should sleep for 3600 seconds.

The container should mount a read-only secret volume called secret-volume at the path /etc/secret-volume. The secret being mounted has already been created for you and is called dotfile-secret.

Make sure that the pod is scheduled on master and no other node in the cluster.

`kubectl run my-busybox --image=busybox --command sleep 3600 --dry-run -o yaml > q3.yaml`

Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: my-busybox
  name: my-busybox
  namespace: dev2406
spec:
  containers:
  - image: busybox
    name: secret
    command:
    - /bin/sh
    - -c
    - sleep 3600
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secret-volume
      readOnly: true
  volumes:
    - name: secret-volume
      secret:
        secretName: dotfile-secret
  nodeSelector:
    kubernetes.io/hostname: master
  #nodeName: master
  restartPolicy: Always
```

Si utilizamos `nodeSelector` (recomendado) hay que eliminar el taint de `master`:

`kubectl taint nodes master node-role.kubernetes.io/master:NoSchedule-`

Si utilizamos `nodeName` va directo a ese nodo sin tener en cuenta los taint.

## Q4:

Create a single ingress resource called ingress-vh-routing. The resource should route HTTP traffic to multiple hostnames as specified below:

The service video-service should be accessible on http://watch.ecom-store.com:30093/video

The service apparels-service should be accessible on http://apparels.ecom-store.com:30093/wear

Here 30093 is the port used by the Ingress Controller

Ingress:
```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ingress-vh-routing
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: watch.ecom-store.com
    http:
      paths:
      - path: /video
        backend:
          serviceName: video-service
          servicePort: 8080
  - host: apparels.ecom-store.com
    http:
      paths:
      - path: /wear
        backend:
          serviceName: apparels-service
          servicePort: 8080
```

`curl http://watch.ecom-store.com:30093/video`

`curl http://apparels.ecom-store.com:30093/wear`

## Q5:

A pod called dev-pod-dind-878516 has been deployed in the default namespace. Inspect the logs for the container called log-x and redirect the warnings to /opt/dind-878516_logs.txt on the master node

`kubectl logs dev-pod-dind-878516 -c log-x`

`kubectl logs dev-pod-dind-878516 -c log-x | grep WARN > /opt/dind-878516_logs.txt`
