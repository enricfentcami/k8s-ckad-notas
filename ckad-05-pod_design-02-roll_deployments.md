# POD DESIGN - Deployments: Rolling updates and rollbacks

## **1. Rollout and Versioning**

Rollout is launched every time a Deployment is created or updated. This generates a review of the changes made and which can be used for rollback.

Obtain the rollout status of a Deployment:

`kubectl rollout status deployment/myapp-deployment`

List the revisions and history of a Deployment:

`kubectl rollout history deployment/myapp-deployment`

### **1.1. Deployment Strategy**

- Recreate: When updating, it deletes the Pods and recreates them at the same time, causing loss of service.
  - Internally scales the replicas of the old Deployment to 0 and resets them for the update in the new Deployment.
- Rolling update (default): Delete and create Pods one by one to avoid loss of service.
  - Internally it scales the replicas of the new (+) and old (-) Deployment one by one.

### **1.2. Update Deployment**

If some Deployment parameter is modified, such as the version of a container, number of replicas, ... And the following command is executed, it creates a new rollout:

`kubectl apply -f deployment-def.yaml`

To update the image directly you can use the following command, but we will have to update our YAML definition file for future uses:

`kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1`

`kubectl set image deploy myapp-deployment nginx=nginx:1.9.1`

Note: `nginx=nginx:1.9.1` is the name of the container and the new image with its version

### **1.3. Upgrades**

Internally, it creates a new ReplicaSet where it assigns the Pods and the old one eliminates them. The old ReplicaSet will remain at 0 in number of replicas.

`kubectl get replicasets`

## **2. Rollback**

Return a Deployment to the previous state.

`kubectl rollout undo deployment/myapp-deployment`

It will destroy the pods of the new ReplicaSet and relaunch the pods of the old one.

Return to a specific version:

`kubectl rollout undo deployment/nginx-deployment --to-revision=2`

## **3. Commands**

Commands to work with deployments:
- Create from file: `kubectl create -f deployment-def.yaml` or `kubectl apply -f deployment-def.yaml`
- Get: `kubectl get deployments` or `kubectl get deploy`
- Update: `kubectl apply -f deployment-def.yaml` (Create if not exists)
- Status: `kubectl rollout status deployment/myapp-deployment`
- History: `kubectl rollout history deployment/myapp-deployment`
- Show revision history: `kubectl rollout history deploy myapp-deployment --revision=2`
- Rollback to previous version: `kubectl rollout undo deployment/myapp-deployment`
- Rollback to older version: `kubectl rollout undo deploy nginx-deployment --to-revision=2`

## **4. Examples: Updating a Deployment**

Here are some handy examples related to updating a Kubernetes Deployment:

### **4.1. Creating a deployment, checking the rollout status and history:**

In the example below, we will first create a simple deployment and inspect the rollout status and the rollout history:

```console
master $ kubectl create deployment nginx --image=nginx:1.16
deployment.apps/nginx created
  
master $ kubectl rollout status deployment nginx
Waiting for deployment "nginx" rollout to finish: 0 of 1 updated replicas are available...
deployment "nginx" successfully rolled out
  
master $
  
master $ kubectl rollout history deployment nginx
deployment.extensions/nginx
REVISION CHANGE-CAUSE
1     <none>
  
master $
```

### **4.2. Using the --revision flag:**

Here the revision 1 is the first version where the deployment was created.

You can check the status of each revision individually by using the --revision flag:

```console
master $ kubectl rollout history deployment nginx --revision=1
deployment.extensions/nginx with revision #1
  
Pod Template:
  Labels:    app=nginx    pod-template-hash=6454457cdb
  Containers:  nginx:  Image:   nginx:1.16
  Port:    <none>
  Host Port: <none>
  Environment:    <none>
  Mounts:   <none>
  Volumes:   <none>
master $ 
```

### **4.3. Using the --record flag:**

You would have noticed that the "change-cause" field is empty in the rollout history output. We can use the --record flag to save the command used to create/update a deployment against the revision number.

```console
master $ kubectl set image deployment nginx nginx=nginx:1.17 --record
deployment.extensions/nginx image updated
master $master $
  
master $ kubectl rollout history deployment nginx
deployment.extensions/nginx
  
REVISION CHANGE-CAUSE
1     <none>
2     kubectl set image deployment nginx nginx=nginx:1.17 --record=true
master $
```

You can now see that the change-cause is recorded for the revision 2 of this deployment.

Let's make some more changes. In the example below, we are editing the deployment and changing the image from nginx:1.17 to nginx:latest while making use of the --record flag.

```console
master $ kubectl edit deployments. nginx --record
deployment.extensions/nginx edited
  
master $ kubectl rollout history deployment nginx
REVISION CHANGE-CAUSE
1     <none>
2     kubectl set image deployment nginx nginx=nginx:1.17 --record=true
3     kubectl edit deployments. nginx --record=true
  
master $ kubectl rollout history deployment nginx --revision=3
deployment.extensions/nginx with revision #3
  
Pod Template: Labels:    app=nginx
    pod-template-hash=df6487dc Annotations: kubernetes.io/change-cause: kubectl edit deployments. nginx --record=true
  
  Containers:
  nginx:
  Image:   nginx:latest
  Port:    <none>
  Host Port: <none>
  Environment:    <none>
  Mounts:   <none>
  Volumes:   <none>
  
master $
```

### **4.3. Undo a change:**

Lets now rollback to the previous revision:

```console
master $ kubectl rollout undo deployment nginx
deployment.extensions/nginx rolled back
  
master $ kubectl rollout history deployment nginx
deployment.extensions/nginxREVISION CHANGE-CAUSE
1     <none>
3     kubectl edit deployments. nginx --record=true
4     kubectl set image deployment nginx nginx=nginx:1.17 --record=true

master $ kubectl rollout history deployment nginx --revision=4
deployment.extensions/nginx with revision #4Pod Template:
  Labels:    app=nginx    pod-template-hash=b99b98f9
  Annotations: kubernetes.io/change-cause: kubectl set image deployment nginx nginx=nginx:1.17 --record=true
  Containers:
  nginx:
  Image:   nginx:1.17
  Port:    <none>
  Host Port: <none>
  Environment:    <none>
  Mounts:   <none>
  Volumes:   <none>
  
master $ kubectl describe deployments. nginx | grep -i image:
  Image:    nginx:1.17

master $
```

With this, we have rolled back to the previous version of the deployment with the image = nginx:1.17.