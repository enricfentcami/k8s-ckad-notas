Sample questions from: https://tanzu.vmware.com/developer/blog/ckad-practice-questions-sept-21/

# Application Design and Build (20%)

## 1. We have a specific job (say, image “busybox”) that needs to run once a day. How do we do that?

<details><summary>Show answer</summary>
<p>

The following would run the job once a day at 2am. If you’re not familiar with crontab syntax, check out crontab.guru for a refresher (but keep in mind that you wouldn’t be able to use crontab.guru during the exam).

k create cronjob test-job --image=busybox --schedule="0 2 * * *"

</p>
</details>

## 2. Let’s assume that the job typically takes an hour. Sometimes however it remains stuck forever. How can we avoid that?

<details><summary>Show answer</summary>
<p>

We can use a liveness probe.

From the Kubernetes docs: “Many applications running for long periods of time eventually transition to broken states, and cannot recover except by being restarted. Kubernetes provides liveness probes to detect and remedy such situations.”

</p>
</details>

## 3. Provide YAML for a deployment running NGINX with the official NGINX image.

<details><summary>Show answer</summary>
<p>

k create deploy nginx --image=nginx -o yaml --dry-run=client

If you want to test it out you can dump it into a file and run it, e.g.:

k create deploy nginx --image=nginx -o yaml --dry-run=client > ckad-nginx.yaml

k apply -f ckad-nginx.yaml

Then you can use port-forward

k port-forward deploy/nginx 1234:80

Go to your browser for http://localhost:1234. You should see an Welcome to NGINX webpage.

And run ctrl-c to terminate the port-forward.
</p>
</details>

## 4. The official NGINX image serves files from /usr/share/nginx/html/; provide a deployment YAML that runs NGINX but serves files downloaded from a git repo (hint: use a volume and an initContainer).

<details><summary>Show answer</summary>
<p>

If we follow the provided hint, we will want to put the files of the git repo into the volume. The volume will be mounted into the NGINX container (in the /usr/share/nginx/html directory), and it will also be mounted into an initContainer that will take care of downloading the files (e.g. by cloning the repo).

Since we need a git repo to test the whole thing, I’m using one of my GitHub repositories in the example below. My GitHub repository has a few static HTML files for the purpose of this example. The easiest way to produce the YAML is probably to start with the NGINX YAML from the previous question (and use kubectl apply to try it out and iterate until we get it right). Alternatively, you could also create a Deployment, and then tweak it with kubectl edit. We need to make three changes in the Deployment manifest:

    Add the volume
    Mount the volume in the NGINX container, at /usr/share/nginx/html/
    Add the initContainer that will also mount the volume, and clone the git repo

The mount path in the initContainer doesn’t matter, as long as the git command is set to clone the repo in that same path.

When we put all these changes together, we get a YAML manifest looking like the one below:

apiVersion: apps/v1
kind: Deployment
metadata:
 name: nginx-github
spec:
 selector:
   matchLabels:
     app: nginx-github
 replicas: 1
 template:
   metadata:
     labels:
       app: nginx-github
   spec:
     containers:
     - name: nginx
       image: nginx
       ports:
       - containerPort: 80
       volumeMounts:
       - name: www-data
         mountPath: /usr/share/nginx/html
     # These containers are run during pod initialization
     initContainers:
     - name: git
       image: alpine/git
       command:
       - git
       - clone
       - https://github.com/tiffanyfay/space-app.git
       - /data
       volumeMounts:
       - name: www-data
         mountPath: /data # You can choose a different name if you want
     volumes:
     - name: www-data
       emptyDir: {}
</p>
</details>

# Application Deployment (20%)

## 1. Provide YAML for deployment called “blue” running NGINX

<details><summary>Show answer</summary>
<p>

k create deploy blue --image=nginx -o yaml --dry-run=client > nginx-blue.yaml

k apply -f nginx-blue.yaml

</p>
</details>

## 2. Provide YAML for deployment called “green” running NGINX

<details><summary>Show answer</summary>
<p>

k create deploy green --image=nginx -o yaml --dry-run=client > nginx-green.yaml

k apply -f nginx-green.yaml

</p>
</details>

## 3. Provide YAML for an internal service called “prod” sending traffic to deployment “blue”

<details><summary>Show answer</summary>
<p>

We can either use k expose or k create svc here. With expose we don’t have to do anything afterwards since it is associated with an existing resource so it knows what selector to use.

k expose deployment blue --name=prod --port=80 > -o yaml --dry-run=client > prod-svc.yaml

k apply -f prod-svc.yaml

Another way to do this is with k create svc clusterip prod and then change the selector to app: blue. If you don’t have a resource you’re creating a service for already you’ll have to use create svc instead of expose.

We can check this worked by getting the IP addresses for blue and green and seeing what endpoints we have for our service.

k get pods -l 'app in (blue,green)' -o wide

The endpoint listed for the prod service should be <blue-IP>:80.

k get ep prod

</p>
</details>

## 4. Switch the traffic to mix of blue+green

<details><summary>Show answer</summary>
<p>

To switch traffic we can add a new label to blue and green and have the service selector use this new label. For instance this could be svc: prod.

We can add svc: prod at the level of spec.template.metadata.labels on a line next to app: <green or blue> and do a k apply -f on the files again.

Solely for the purpose of testing, you could just label those pods for now. This doesn’t work in the long term though because if the pods terminate and new ones create they won’t have the label.

kubectl label pods --selector app=blue svc=prod
kubectl label pods --selector app=green svc=prod

Edit the service YAML to change the selector from app: blue to svc: prod then apply it:

kubectl apply -f 

We can see that this worked by checking the endpoints again. There should be two now.

k get ep prod

Switch traffic back to blue only. You could do this by removing the svc: prod label line from the green pods or change the service selector back to app: blue using kubectl edit
</p>
</details>

# Application Observability and Maintenance (15%)

## 1. What is the difference between liveness and readiness probes?

<details><summary>Show answer</summary>
<p>

Readiness: Detect if the thing is not ready, e.g. if it’s overloaded, busy. It’s like putting a little sign on saying “I am busy!” or “Gone for a break!”, and we leave it alone; we don’t send requests to it until it’s ready again.

Liveness: Detect if the thing is dead. When it’s dead it won’t come back so we need to replace it with a new one (=restart it) because Kubernetes is not the walking dead.
</p>
</details>

## 2. Provide YAML for deployment running NGINX, with HTTP readiness probe

<details><summary>Show answer</summary>
<p>

k create deploy nginx-readiness --image=nginx
k edit deploy nginx-readiness

Add the following under the template.spec.containers level. So this should be in line with name, image, etc.

       readinessProbe:
         httpGet:
           path: /index.html
           port: 80
         initialDelaySeconds: 5
         periodSeconds: 5 # how long to wait after first try

Watch for the deployment to become ready:

k get deploy nginx-readiness -w

Run ctrl-c to stop the watch.
</p>
</details>

# Application Environment, Configuration and Security (25%)

## 1. Create deployment “purple” running NGINX**

<details><summary>Show answer</summary>
<p>

k create deploy purple --image=nginx --port=80

</p>
</details>

## 2. Create service account named “scaler”

<details><summary>Show answer</summary>
<p>

k create sa scaler

</p>
</details>

## 3. Create a pod named scaler with the following requirements:
- it should use that scaler serviceaccount
- it should be possible to obtain an interactive shell in it
- it should have kubectl installed (install it manually somehow or use e.g. nixery.dev/shell/kubectl)

<details><summary>Show answer</summary>
<p>

The following will run forever so we can get an interactive shell, otherwise the pod will just terminate. And we don’t have a flag for serviceaccount so we need to use an override or you could add the serviceaccount with kubectl edit or do -o yaml --dry-run=client > some file name, add it, and then kubectl apply -f.

k run kubectl --image=nixery.dev/shell/kubectl --overrides='{ "spec": { "serviceAccount": "scaler" } }' -- /bin/bash -c "while true; do sleep 30; done;"

</p>
</details>

## 4. Create a role/rolebinding such as serviceaccount scaler can scale up/down the deployment purple but cannot do anything else (not delete the deployment, not edit it other than scaling, not view/touch other resources)

<details><summary>Show answer</summary>
<p>

Unfortunately kubectl create role doesn’t let you have multiple API groups so either you need to create two separate ones and merge them manually e.g.

k create role scaler --verb=get --resource=deployments --resource-name=purple -o yaml --dry-run=client > scaler-get.yaml

k create role scaler --verb=patch --resource=deployments/scale --resource-name=purple -o yaml --dry-run=client > scaler-patch.yaml

Or just create a YAML file:

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
 name: scaler
rules:
- apiGroups: ["apps"]
 resources: ["deployments"]
 verbs: ["get"]
 resourceNames: ["purple"]
- apiGroups: ["apps"]
 resources: ["deployments/scale"]
 verbs: ["patch"]
 resourceNames: ["purple"]

Create the rolebinding

k create rolebinding scaler --serviceaccount=ckad:scaler --role=scaler

Now let’s open an interactive shell:

k exec -it kubectl -- sh

Let’s see what we have permissions to do:

kubectl auth can-i --list

We should see the following two:

deployments.apps                                []                                    [purple]         [get]
deployments.apps/scale                          []                                    [purple]         [patch]

Now let’s scale up to 2 replicas:

kubectl scale deploy purple --replicas=2
kubectl get deploy purple

Run exit to exit the container.
</p>
</details>

# Services and Networking (20%)

## 1. Create deployment “yellow” running nginx:alpine

<details><summary>Show answer</summary>
<p>

k create deploy yellow --image=nginx:alpine

</p>
</details>

## 2. Create deployment “orange” running nginx:alpine

<details><summary>Show answer</summary>
<p>

k create deploy orange --image=nginx:alpine

</p>
</details>

## 3. Block access to pods of deployment “yellow”

<details><summary>Show answer</summary>
<p>

This is where we involve network policies. We can block all traffic into (ingress) the yellow pods. Unfortunately at this time we can’t use kubectl to do this so we need YAML.

k apply -f the following as a file.

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
 name: block-to-yellow
spec:
 podSelector:
   matchLabels:
     app: yellow
 policyTypes:
 - Ingress
 ingress: []

Now let’s test this.

First, get our pods’ IP addresses.

k get pods -l 'app in (yellow,orange)' -o wide

First let’s see that we can ping orange.

kubectl run --rm -it ping --image=alpine --restart=Never -- ping <orange-ip>

You should see something like:

If you don't see a command prompt, try pressing enter.
64 bytes from 10.47.0.1: seq=1 ttl=64 time=1.301 ms
64 bytes from 10.47.0.1: seq=2 ttl=64 time=1.264 ms

Now try with yellow. You shouldn’t see any traffic.

kubectl run --rm -it ping --image=alpine --restart=Never -- ping <yellow-ip>

Run ctrl-c to terminate it.

Allow pods of “orange” to ping pods of “yellow” Now we need a new network policy that allows ingress traffic from orange to yellow. Create the following YAML and k apply -f <file>.

kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-orange-to-yellow
spec:
  podSelector:
    matchLabels:
      app: yellow
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: orange

This time we need to enter the orange pod and to ping yellow. We will need to use yellow’s IP. And we should see traffic.

k exec -it <orange-pod-name> -- ping <yellow-ip>
</p>
</details>
