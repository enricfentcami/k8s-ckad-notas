# MOCK EXAM 2


   34  
   35  vi q5.yaml
   36  kubectl create -f q5.yaml
   37  kubectl delete -f q5.yaml
   38  kubectl create -f q5.yaml
   39  kubectl get po
   40  kubectl run nginx1401 --image:nginx -o yaml --dry-run > q6.yaml
   41  
   42  vi q6.yaml
   43  kubectl create -f q6.yaml
   44  kubectl get po
   45  
   46  kubectl get po
   47  vi q6.yaml
   48  kubectl get po
   49  vi q6.yaml
   50  kubectl create job
   51  kubectl create job -h
   52  kubectl create job whalesay --image=docker/whalesay -- "cowsay I am going to ace CKAD!" -o yaml --dry-run > q7.yaml
   53  vi q7.yaml
   54  kubectl get job
   55  kubectl delete job whalesay
   56  kubectl create job whalesay --image=docker/whalesay --command="cowsay I am going to ace CKAD!" -o yaml --dry-run > q7.yaml
   57  
   58  vi q7.yaml
   59  kubectl create -f q7.yaml
   60  kubectl get job
   61  kubectl get po
   62  kubectl describe pod whalesay-ngms2
   63  kubectl delete -f q7.yaml
   64  vi q7.yaml
   65  kubectl create -f q7.yaml
   66  kubectl get po
   67  kubectl get job
   68  kubectl run multi-pod --image=nginx --command=sleep 4800 --env=type=planet -o yaml --dry-run > q8.yaml
   69  kubectl run multi-pod --image=nginx --command="sleep 4800" --env=type=planet -o yaml --dry-run > q8.yaml
   70  kubectl run multi-pod --image=nginx --command "sleep 4800" --env=type=planet -o yaml --dry-run > q8.yaml
   71  vi q8.yaml
   72  
   73  kubectl create -f q8.yaml
   74  ls
   75  kubectl get po
   76  kubectl describe multi-pod
   77  kubectl describe pod multi-pod
   78  vi q8.yaml
   79  kubectl delete -f q8.yaml
   80  kubectl create -f q8.yaml
   81  kubectl get po
   82  vi q9.yaml
   83  kubectl create -f q9.yaml
   84  vi q9.yaml
   85  kubectl create -f q9.yaml
   86  kubectl get pv
   87  clear
   88  history




## Q1:

Create a deployment called my-webapp with image: nginx, label tier:frontend and 2 replicas. Expose the deployment as a NodePort service with name front-end-service , port: 80 and NodePort: 30083

`kubectl create deploy my-webapp --image=nginx -o yaml --dry-run > q1-dep.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: my-webapp
    tier: frontend
  name: my-webapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-webapp
      tier: frontend
  template:
    metadata:
      labels:
        app: my-webapp
        tier: frontend
    spec:
      containers:
      - image: nginx
        name: nginx
```

`kubectl expose deploy my-webapp --name=front-end-service --type=NodePort --port=80 --dry-run -o yaml > q1-svc.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: my-webapp
    tier: frontend
  name: front-end-service
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30083
  selector:
    app: my-webapp
    tier: frontend
  type: NodePort
```

## Q2:

Add a taint to the node node01 of the cluster. Use the specification below:

`kubectl taint nodes node01 app_type=alpha:NoSchedule`

`kubectl run alpha --image=redis -o yaml --dry-run > q2.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: alpha
  name: alpha
spec:
  containers:
  - image: redis
    name: alpha
  restartPolicy: Always
  tolerations:
  - key: "app_type"
    operator: "Equal"
    value: "alpha"
    effect: "NoSchedule"
```

## Q3:

Apply a label app_type=beta to node node02. Create a new deployment called beta-apps with image:nginx and replicas:3. Set Node Affinity to the deployment to place the PODs on node02 only

`kubectl label nodes node02 app_type=beta`

`kubectl create deploy beta-apps --image=nginx -o yaml --dry-run > q3.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: beta-apps
  name: beta-apps
spec:
  replicas: 3
  selector:
    matchLabels:
      app: beta-apps
  template:
    metadata:
      labels:
        app: beta-apps
    spec:
      containers:
      - image: nginx
        name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: app_type
                operator: In
                values:
                - beta
```

## Q4:

Create a new Ingress Resource for the service: my-video-service to be made available at the URL: http://ckad-mock-exam-solution.com:30093/video.

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: my-video-service
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: ckad-mock-exam-solution.com
    http:
      paths:
      - path: /video
        backend:
          serviceName: my-video-service
          servicePort: 8080
```

`curl http://ckad-mock-exam-solution.com:30093/video`

## Q5:

We have deployed a new pod called pod-with-rprobe. This Pod has an initial delay before it is Ready. Update the newly created pod pod-with-rprobe with a readinessProbe using the given spec

`kubectl get po pod-with-rprobe -o yaml > q5.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: pod-with-rprobe
  name: pod-with-rprobe
spec:
  containers:
  - env:
    - name: APP_START_DELAY
      value: "180"
    image: kodekloud/webapp-delayed-start
    imagePullPolicy: Always
    name: pod-with-rprobe
    ports:
    - containerPort: 8080
      protocol: TCP
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
```

## Q6:

Create a new pod called nginx1401 in the default namespace with the image nginx. Add a livenessProbe to the container to restart it if the command ls /var/www/html/probe fails. This check should start after a delay of 10 seconds and run every 60 seconds.

`kubectl run nginx1401 --image=nginx -o yaml --dry-run > q6.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx1401
  name: nginx1401
  namespace: default
spec:
  containers:
  - image: nginx
    name: nginx1401
    livenessProbe:
      exec:
        command:
        - ls
        - /var/www/html/probe
      initialDelaySeconds: 10
      periodSeconds: 60
```

## Q7:

Create a job called whalesay with image docker/whalesay and command "cowsay I am going to ace CKAD!".
completions: 10
backoffLimit: 6
restartPolicy: Never

`kubectl create job whalesay --image=docker/whalesay  -o yaml --dry-run > q7.yaml`

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: whalesay
spec:
  backoffLimit: 6
  completions: 10
  template:
    spec:
      containers:
      - image: docker/whalesay
        name: whalesay
        #command: ["/bin/sh", "-c", "cowsay I am going to ace CKAD!"]
        command: ["cowsay", "I am going to ace CKAD!"]
        #command: ["cowsay"]
        #args: ["I am going to ace CKAD!"]
      restartPolicy: Never
```

`kubectl logs whalesay-9vrgn`

```console
 _________________________
< I am going to ace CKAD! >
 -------------------------
    \
     \
      \
                    ##        .
              ## ## ##       ==
           ## ## ## ##      ===
       /""""""""""""""""___/ ===
  ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
       \______ o          __/
        \    \        __/
          \____\______/
```

## Q8:

Create a pod called multi-pod with two containers.
Container 1: name: jupiter, image: nginx
Container 2: europa, image: busybox
  command: sleep 4800

Environment Variables:
* Container 1: type: planet
* Container 2: type: moon

`kubectl run multi-pod --image=busybox --command "sleep 4800" --env=type=moon -o yaml --dry-run > q8.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: multi-pod
  name: multi-pod
spec:
  containers:
  - env:
    - name: type
      value: planet
    image: nginx
    name: jupiter
  - name: europa
    image: busybox
    command:
    - /bin/sh
    - -c
    - sleep 4800
    env:
    - name: type
      value: moon
```

## Q9:

Create a PersistentVolume called custom-volume with size: 50MiB reclaim policy:retain, Access Modes: ReadWriteMany and hostPath: /opt/data

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: custom-volume
spec:
  capacity:
    storage: 50Mi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /opt/data
```