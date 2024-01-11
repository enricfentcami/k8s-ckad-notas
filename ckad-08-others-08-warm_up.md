# WARM UP

## Create a namespace warmup and set it by default

<details><summary>Show answer</summary>
<p>

`kubectl create ns warmup`

`kubectl config set-context --current --namespace=warmup`

</p>
</details>

## Create a pod nginx with nginx image, expose port 80. With labels app=pod1 and tier=front

<details><summary>Show answer</summary>
<p>

`kubectl run nginx --image=nginx --port=80 -l app=pod1 -l tier=front`

</p>
</details>

## Create a pod temp busybox to call nginx pod and get/print the main page

<details><summary>Show answer</summary>
<p>

`kubectl get po -o wide` -> Get the pod IP

`kubectl run busybox --image=busybox --rm -it -- /bin/sh -c 'wget -O- 10.239.0.31:80'`

</p>
</details>

## Create a service (nginx-service) to pod nginx and publish it on 8080

<details><summary>Show answer</summary>
<p>

`kubectl expose pod nginx --name=nginx-service --port=8080 --target-port=80`

</p>
</details>

## Create a pod temp busybox to call nginx-service pod and get/print the main page

<details><summary>Show answer</summary>
<p>

`kubectl run busybox --image=busybox --rm -it -- /bin/sh -c 'wget -O- nginx-service:8080'`

</p>
</details>

## Create a pod nginx2 with nginx image, expose port 80, resources: request cpu 1 amd memory 256Mb, max cpu 2 and memory 512Mb. Once created get the yaml

<details><summary>Show answer</summary>
<p>

`kubectl run nginx2 --image=nginx --port=80 --requests=cpu=1,memory=256Mi --limits=cpu=2,memory=512Mi`

`kubectl get po nginx2 -o yaml`

</p>
</details>

## Delete pods and service, force

<details><summary>Show answer</summary>
<p>

`kubectl delete pod nginx nginx2 --force --grace-period=0`

</p>
</details>

## Create a deployment nginx-deployment with nginx image with 3 replicas and expose port 80

<details><summary>Show answer</summary>
<p>

`kubectl create deployment nginx-deployment --image=nginx -o yaml --dry-run > dep.yaml`

`vim dep.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 3 # Change this
  selector:
    matchLabels:
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx
        name: nginx
        ports: # Add this
          - containerPort: 80
```

`kubectl create -f dep.yaml`

</p>
</details>

## Create a service (nginx-deployment-service) for the deployment of type NodePort

<details><summary>Show answer</summary>
<p>

`kubectl expose deployment nginx-deployment --name=nginx-deployment-service --type=NodePort`

</p>
</details>

## Scale up deployment to 5 replicas

<details><summary>Show answer</summary>
<p>

`kubectl scale deployment nginx-deployment --replicas=5`

</p>
</details>

## Create a pod temp busybox to call nginx-deployment-service and get/print the main page

<details><summary>Show answer</summary>
<p>

`kubectl get svc` -> Get the port

`kubectl get nodes` -> Get the node name (worker)

`kubectl run busybox --image=busybox --rm -it -- /bin/sh -c 'wget -O- node10031-ckad-sandbox.jelastic.labs.gmv.com:30817'`

</p>
</details>

## Delete deployment and service

<details><summary>Show answer</summary>
<p>

`kubectl delete svc nginx-deployment-service`

`kubectl delete deployment nginx-deployment`

</p>
</details>

## Create a config map "myconfig" with values host=localhost, port=8080

<details><summary>Show answer</summary>
<p>

`kubectl create configmap myconfig --from-literal=host=localhost --from-literal=port=8080`

</p>
</details>

## Create a secre "mysecret" with values user=admin, password=ckad

<details><summary>Show answer</summary>
<p>

`kubectl create secret generic mysecret --from-literal=user=admin --from-literal=password=ckad`

</p>
</details>

## Create a pod busybox to load config map and secret vars in env, and prints environment vars. Check and delete it

<details><summary>Show answer</summary>
<p>

`kubectl run busybox --image=busybox --dry-run -o yaml -- env > podenv.yaml`

`vim podenv.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: busybox
  name: busybox
spec:
  containers:
  - args:
    - env
    image: busybox
    name: busybox
    envFrom:
      - configMapRef:
          name: myconfig
      - secretRef:
          name: mysecret
  restartPolicy: Never
```

`kubectl create -f podenv.yaml`

`kubectl logs busybox`

</p>
</details>

## Create a job with the image busybox that executes the command 'echo hello;sleep 30;echo world'

<details><summary>Show answer</summary>
<p>

`kubectl create job busybox --image=busybox -- /bin/sh -c 'echo hello;sleep 30;echo world'`

</p>
</details>

## Follow the logs for the pod (you'll wait for 30 seconds)

<details><summary>Show answer</summary>
<p>

`kubectl logs busybox-xxxxx -f`

</p>
</details>

## Create a cron job with image busybox that runs on a schedule of "*/1 * * * *" and writes 'date; echo Hello from the Kubernetes cluster' to standard output

<details><summary>Show answer</summary>
<p>

`kubectl create cronjob cron --image=busybox --schedule="*/1 * * * *"  -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'`

</p>
</details>
