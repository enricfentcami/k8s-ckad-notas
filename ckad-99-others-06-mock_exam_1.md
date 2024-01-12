# MOCK EXAM 1

## Q1:

Deploy a pod named nginx-448839 using the nginx:alpine image

`kubectl run nginx-448839 --image=nginx:alpine`

## Q2:

Create a namespace named apx-z993845

`kubectl create ns apx-z993845`

## Q3:

Create a new Deployment named httpd-frontend with 3 replicas using image httpd:2.4-alpine

`kubectl create deploy httpd-frontend --image=httpd:2.4-alpine -o yaml --dry-run > q3.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: httpd-frontend
  name: httpd-frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: httpd-frontend
  template:
    metadata:
      labels:
        app: httpd-frontend
    spec:
      containers:
      - image: httpd:2.4-alpine
        name: httpd
```

## Q4:

Deploy a messaging pod using the redis:alpine image with the labels set to tier=msg.

`kubectl run messaging --image=redis:alpine -l tier=msg`

## Q5:

A replicaset rs-d33393 is created. However the pods are not coming up. Identify and fix the issue.

`kubectl describe rs rs-d33393`

`kubectl get rs rs-d33393 -o yaml > q5.yaml`

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-d33393
spec:
  replicas: 4
  selector:
    matchLabels:
      name: busybox-pod
  template:
    metadata:
      labels:
        name: busybox-pod
    spec:
      containers:
      - command:
        - sh
        - -c
        - echo Hello Kubernetes! && sleep 3600
        image: busybox
        imagePullPolicy: IfNotPresent
        name: busybox-container
```

## Q6:

Create a service messaging-service to expose the redis deployment in the marketing namespace within the cluster on port 6379.

`kubectl expose deployment redis -n marketing --name=messaging-service --port=6379`

## Q7:

Update the environment variable on the pod webapp-color to use a green background

`kubectl get po webapp-color -o yaml > q7.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: webapp-color
  name: webapp-color
spec:
  containers:
  - env:
    - name: APP_COLOR
      value: green
    image: kodekloud/webapp-color
    name: webapp-color
```

## Q8:

Create a new ConfigMap named cm-3392845. Use the spec given on the right.

`kubectl create configmap cm-3392845 --from-literal=DB_NAME=SQL3322 --from-literal=DB_HOST=sql322.mycompany.com --from-literal=DB_PORT=3306`

## Q9:

Create a new Secret named db-secret-xxdf with the data given(on the right).

`kubectl create secret generic db-secret-xxdf --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123`

## Q10:

Update pod app-sec-kff3345 to run as Root user and with the SYS_TIME capability.

`kubectl get po app-sec-kff3345 -o yaml > q10.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-sec-kff3345
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    imagePullPolicy: Always
    name: ubuntu
    securityContext:
      capabilities:
        add: ["SYS_TIME"]
```

## Q11:

Export the logs of the e-com-1123 pod to the file /opt/outputs/e-com-1123.logs

`kubectl logs e-com-1123 -n e-commerce > /opt/outputs/e-com-1123.logs`

## Q12:

Create a Persistent Volume with the given specification.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-analytics
spec:
  capacity:
    storage: 100Mi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /pv/data-analytics
```

## Q13:

Create a redis deployment using the image redis:alpine with 1 replica and label app=redis. Expose it via a ClusterIP service called redis on port 6379. Create a new Ingress Type NetworkPolicy called redis-access which allows only the pods with label access=redis to access the deployment.

Ojo: Ya mete la label `app=redis` por defecto, si fuera otra habría que exportarlo a YAML y añadirla.

`kubectl create deploy redis --image=redis:alpine`

`kubectl expose deploy redis --name=redis --port=6379`

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: redis-access
spec:
  podSelector:
    matchLabels:
      app: redis
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: redis
    ports:
    - protocol: TCP
      port: 6379
```

## Q14:

Create a Pod called sega with two containers:

Container 1: Name tails with image busybox and command: sleep 3600.
Container 2: Name sonic with image nginx and Environment variable: NGINX_PORT with the value 8080.

`kubectl run sega --image=busybox --command sleep 3600 --dry-run -o yaml > q14.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: sega
  name: sega
spec:
  containers:
  - command:
    - sleep
    - "3600"
    image: busybox
    name: tails
  - name: sonic
    image: nginx
    env:
    - name: NGINX_PORT
      value: "8080"
```