# EXAMPLE COMPLETE

## **POD**
---

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-full
  namespace: default
  labels:
    name: webapp-full
    app: nginx
    function: front-end
  annotations:
    version: 1.17
    company: EEBB
    country: Spain
spec:
  containers:
  - name: webapp-full
    image: nginx
    command: ["nginx"]
    args: ["-g", "daemon off;"]
    # Ports
    ports:
    - containerPort: 80
      name: http
      protocol: TCP # Default
    # Env vars (literal, configMap, secret)
    env:
    - name: APP_VAR0
      value: "VALUE_0"
    - name: APP_VAR1
      valueFrom:
        configMapKeyRef:
          name: app-config-1
          key: APP_VAR1
    - name: APP_COLOR
      valueFrom:
        secretKeyRef:
          name: app-secret-1
          key: APP_SEC1
    # Env vars from (configMap, secret)
    envFrom:
    - configMapRef:
      name: app-config-2
    - configMapRef:
      name: app-config-3
    - secretRef:
        name: app-secret-2
    - secretRef:
        name: app-secret-3
    # Volume mounts (hostPath, pvc, configMap)
    volumeMounts:
    - mountPath: /tmp/data
      name: app-data-volume
    - mountPath: /tmp/claim
      name: app-claim-volume
    - mountPath: /tmp/config
      name: app-config-volume
    # Resources (requests, limits)
    resources:
      requests:
        memory: "500Mi"
        cpu: 1
      limits:
        memory: "600Mi"
        cpu: 2
    # Readiness probe (tcpSocket)
    readinessProbe:
      tcpSocket:
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
    # Liveness probe (httpGet)
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 15
      periodSeconds: 20
  # Volumes (hostPath, pvc, configMap)
  volumes:
  - name: app-data-volume
    hostPath:
      path: /data
      type: Directory
  - name: app-claim-volume
    persistentVolumeClaim:
      claimName: app-pvc
  - name: app-config-volume
    configMap:
        name: app-config-4

```

## **REPLICA SET**

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: webapp-rs
  labels:
    name: webapp-rs
    app: nginx
    function: front-end
spec:
  replicas: 3
  # Pod selector by labels
  selector:
    matchLabels:
      name: webapp-rs
      app: nginx
  template: # Pod definition
    metadata:
      labels:
        name: webapp-rs
        app: nginx 
        function: front-end
    spec:
      containers:
        - name: webapp-rs
          image: nginx
```

## **DEPLOYMENT**


## **SERVICE**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: simple-webapp-service
spec:
  selector:
    app: app1 # Para seleccionar qué Pods encajan con este Service
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
