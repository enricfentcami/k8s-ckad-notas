# EXAMPLE COMPLETE

## **POD**
---

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-full
  namespace: eebb
  labels:
    name: webapp-full
    app: nginx
    function: front-end
  annotations:
    version: "1.17"
    company: "EEBB"
    country: "Spain"
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
    - name: APP_SEC1
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
    # Volume mounts (hostPath, temporal, pvc, configMap, secret)
    volumeMounts:
    - mountPath: /tmp/data
      name: app-data-volume
    - mountPath: /tmp/temporal
      name: app-temporal-volume
    - mountPath: /tmp/claim
      name: app-claim-volume
    - mountPath: /tmp/config
      name: app-config-volume
    - mountPath: /tmp/secret
      name: app-secret-volume
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
  # Volumes (hostPath, pvc, configMap, secret)
  volumes:
  - name: app-data-volume
    hostPath:
      path: /data
      type: Directory
  - name: app-temporal-volume
    emptyDir: {}
  - name: app-claim-volume
    persistentVolumeClaim:
      claimName: app-claim-volume
  - name: app-config-volume
    configMap:
        name: app-config-4
  - name: app-secret-volume
    secret:
        name: app-secret-4
```

## **REPLICA SET**

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: webapp-rs
  namespace: eebb
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

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-deployment
  namespace: eebb
  labels:
    name: webapp
    app: nginx
    function: front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      name: webapp
      app: nginx
  template:
    metadata:
      labels:
        name: webapp
        app: nginx
        function: front-end
    spec:
      containers:
      - name: webapp-dep
        image: nginx
        # Ports
        ports:
        - containerPort: 80
          name: http
          protocol: TCP # Default
      [...]
```

## **SERVICE**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
  namespace: eebb
spec:
  selector:  # Para seleccionar qué Pods o Deployment encajan con este Service
    name: webapp
    app: nginx
    function: front-end
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

## **CONFIG MAP**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config-1
  namespace: eebb
data:
  APP_VAR1: VALUE_1
```

Comandos para el Pod:

`kubectl create configmap app-config-1 --from-literal=APP_VAR1=VALUE_1 -n eebb`

`kubectl create configmap app-config-2 --from-literal=APP_VAR2=VALUE_2 -n eebb`

`kubectl create configmap app-config-3 --from-literal=APP_VAR3=VALUE_3 -n eebb`

`kubectl create configmap app-config-4 --from-literal=APP_VAR4=VALUE_4 -n eebb`

## **SECRET**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret-1
  namespace: eebb
data:
  APP_SEC1: U0VDUkVUXzE=
```

Comandos para el Pod:

`kubectl create secret generic app-secret-1 --from-literal=APP_SEC1=SECRET_1 -n eebb`

`kubectl create secret generic app-secret-2 --from-literal=APP_SEC2=SECRET_2 -n eebb`

`kubectl create secret generic app-secret-3 --from-literal=APP_SEC3=SECRET_3 -n eebb`

`kubectl create secret generic app-secret-4 --from-literal=APP_SEC4=SECRET_4 -n eebb`

## **PERSISTENT VOLUME**

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: app-volume
spec:
  storageClassName: webapp
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 100Mi
  hostPath: # Path del nodo. No recomendado en producción
    path: /data/webapp
    type: DirectoryOrCreate
```

## **PERSISTENT VOLUME CLAIM**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-claim-volume
  namespace: eebb
spec:
  storageClassName: webapp
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Mi
```
