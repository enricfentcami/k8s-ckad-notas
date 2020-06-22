# LIGHTNING LAB 1

## Q1:

Create a Persistent Volume called log-volume. It should make use of a storage class name manual. It should use RWX as the access mode and have a size of 1Gi. The volume should use the hostPath /opt/volume/nginx

Next, create a PVC called log-claim requesting a minimum of 200Mi of storage. This PVC should bind to log-volume.

Mount this in a pod called logger at the location /var/www/nginx. This pod should use the image nginx:alpine.

### Solution:

Persistent volume:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: log-volume
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  storageClassName: manual
  hostPath:
    path: /opt/volume/nginx
```

Persistent volume claim:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: log-claim
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: manual
```

Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: logger
spec:
  containers:
    - name: logger
      image: nginx:alpine
      volumeMounts:
      - mountPath: "/var/www/ngix"
        name: log-volume
  volumes:
    - name: log-volume
      persistentVolumeClaim:
        claimName: log-claim
```

## Q2:

We have deployed a new pod called secure-pod and a service called secure-service. Incoming or Outgoing connections to this pod are not working.
Troubleshoot why this is happening.

Make sure that incoming connection from the pod webapp-color are successful.

Important: Don't delete any current objects deployed.

### Solution:

Network policy:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: secure-network-policy
spec:
  podSelector:
    matchLabels:
      run: secure-pod
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: webapp-color
    ports:
    - protocol: TCP
      port: 80
```

## Q3:

Create a pod called time-check in the dvl1987 namespace. This pod should run a container called time-check that uses the busybox image.
1. Create a config map called time-config with the data TIME_FREQ=10 in the same namespace.
2. The time-check container should run the command: while true; do date; sleep $TIME_FREQ;done and write the result to the location /opt/time/time-check.log.
3. The path /opt/time on the pod should mount a volume that lasts the lifetime of this pod.

### Solution:

`kubectl create ns dvl1987`

`kubectl create configmap time-config --from-literal=TIME_FREQ=10 -n dvl1987`

Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: time-check
  name: time-check
  namespace: dvl1987
spec:
  containers:
  - image: busybox
    name: time-check
    command:
    - /bin/sh
    - -c
    - while true; do date; sleep $TIME_FREQ;done > /opt/time/time-check.log
    env:
      - name: TIME_FREQ
        valueFrom:
          configMapKeyRef:
            name: time-config
            key: TIME_FREQ
    volumeMounts:
      - name: logs
        mountPath: /opt/time
  volumes:
    - name: logs
      emptyDir: {}
      # SE PIDE QUE LA VIDA DEL VOLUMEN SEA LA DEL POD
      #hostPath:
      #  path: /tmp/time
      #  type: DirectoryOrCreate

  restartPolicy: Never
```

## Q4:

Create a new deployment called nginx-deploy, with one signle container called nginx, image nginx:1.16 and 4 replicas. The deployment should use RollingUpdate strategy with maxSurge=1, and maxUnavailable=2.

Next upgrade the deployment to version 1.17 using rolling update.

Finally, once all pods are updated, undo the update and go back to the previous version.

### Solution:

Deployment:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deploy
  name: nginx-deploy
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx-deploy
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 2
      maxSurge: 1
  template:
    metadata:
      labels:
        app: nginx-deploy
    spec:
      containers:
      - image: nginx:1.16
        name: nginx
```

`kubectl set image deployment/nginx-deploy nginx=nginx:1.17`

`kubectl rollout status deployment/nginx-deploy`

`kubectl rollout undo deployment/nginx-deploy`

`kubectl rollout history deployment/nginx-deploy`


## Q5:

Create a redis deployment with the following parameters:

Name of the deployment should be redis using the redis:alpine image. It should have exactly 1 replica.

The container should request for .2 CPU. It should use the label app=redis.

Make sure that the pod is scheduled on master node.

It should mount exactly 2 volumes:
* An Empty directory volume called data at path /redis-master-data.
* A configmap volume called redis-config at path /redis-master.
* The container should expose the port 6379.

The configmap has already been created.

### Solution:

`kubectl taint nodes master node-role.kubernetes.io/master:NoSchedule-`

Deployment:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: redis
  name: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - image: redis:alpine
        name: redis
        ports:
          - name: default
            containerPort: 6379
        volumeMounts:
          - name: data
            mountPath: /redis-master-data
          - name: redis-config
            mountPath: /redis-master
        resources:
         requests:
           cpu: "0.2"
      nodeSelector:
        kubernetes.io/hostname: master
      volumes:
        - name: data
          emptyDir: {}
        - name: redis-config
          configMap:
            name: redis-config
```