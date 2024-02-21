Sample questions from: https://k21academy.com/docker-kubernetes/cka-ckad-exam-questions-answers/

## 1. Create a new service account with the name pvviewer. Grant this Service account access to list all PersistentVolumes in the cluster by creating an appropriate cluster role called pvviewer-role and ClusterRoleBinding called pvviewer-role-binding.

Next, create a pod called pvviewer with the image: redis and serviceaccount: pvviewer in the default namespace.

<details><summary>Show answer</summary>
<p>

Create Service account

$ kubectl create serviceaccount pvviewer

Create cluster role

$ kubectl create clusterrole pvviewer-role --verb=list --resource=PersistentVolumes

Create cluster role binding

$ kubectl create clusterrolebinding pvviewer-role-binding --clusterrole=pvviewer-role --serviceaccount=default:pvviewer

Verify

$ kubectl auth can-i list PersistentVolumes –as system:serviceaccount:default:pvviewer

</p>
</details>

## 2. Create a new deployment called nginx-deploy, with image nginx:1.16 and 1 replica. Record the version. Next upgrade the deployment to version 1.17 using rolling update. Make sure that the version upgrade is recorded in the resource annotation.

<details><summary>Show answer</summary>
<p>

$ vim nginx-deployment.yaml
$ kubectl apply -f nginx-deployment.yaml --record
$ kubectl get deployment
$ kubectl rollout history deployment nginx-deploy

$ kubectl set image deployment/nginx-deploy nginx=1.17 --record
$ kubectl rollout history deployment nginx-deploy

$ kubectl describe deployment nginx-deploy

</p>
</details>

## 3. Create a Persistent Volume with the given specification.

Volume Name: pv-analytics, Storage: 100Mi, Access modes: ReadWriteMany, Host Path: /pv/data-analytics

<details><summary>Show answer</summary>
<p>

$ vim pv.yaml

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
    path:  /pv/data-analytics

$ kubectl create -f pv.yaml
$ kubectl get pv

Read More: K8s Persistent Storage

</p>
</details>

## 4. Taint the worker node to be Unschedulable. Once done, create a pod called dev-redis, image redis:alpine to ensure workloads are not scheduled to this worker node. Finally, create a new pod called prod-redis and image redis:alpine with toleration to be scheduled on node01.

key:env_type, value:production, operator: Equal and effect:NoSchedule

<details><summary>Show answer</summary>
<p>

$ kubectl get nodes
$ kubectl taint node node01 env_type=production:NoSchedule
$ kubectl describe nodes node01 | grep -i taint
$ kubectl run dev-redis --image=redis:alpine --dyn-run=client -o yaml > pod-redis.yaml
$ vi prod-redis.yaml

apiVersion: v1 
kind: Pod 
metadata:
  name: prod-redis 
spec:
  containers:
  - name:  prod-redis 
    image:  redis:alpine
  tolerations:
  - effect: Noschedule 
    key: env_type 
    operator: Equal 
    value: prodcution

$ kubectl create -f prod-redis.yaml

Read More: Scheduling in K8s

</p>
</details>

## 5. Create a Pod called non-root-pod , image: redis:alpine

runAsUser: 1000

fsGroup: 2000

<details><summary>Show answer</summary>
<p>

$ vim non-root-pod.yaml
$ kubectl create -f non-root-pod.yaml

apiVersion: v1 
kind: Pod 
metadata:
  name:  non-root-pod 
spec:
  securityContext: 
    runAsUser:  1000
    fsGroup:  2000 
  containers:
  -  name:  non-root-pod

Read More: K8s Pods For Beginners

</p>
</details>

## 6. Create a NetworkPolicy which denies all ingress traffic

<details><summary>Show answer</summary>
<p>

$ vim policy.yaml

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress

$ kubectl create -f policy.yaml

Read More: K8s Network Policy
</p>
</details>