# POD DESIGN - Deployments: Rolling updates and rollbacks

## **1. Rollout and Versioning**
---

Rollout se lanza cada vez que se crea o actualiza un Deployment. Esto genera una revisión de los cambios realizados y que puede ser utilizada para hacer rollback.

Obtener el estado de rollout de un Deployment:

`kubectl rollout status deployment/myapp-deployment`

Listar las revisiones e historial de un Deployment:

`kubectl rollout history deployment/myapp-deployment`

### **1.1. Deployment Strategy**

- Recreate: Al actualizar elimina los Pods y los vuelve a crear a la vez, provocando pérdida de servicio.
  - Internamente escala a 0 las replicas del viejo Deployment y las vuelve a establecer para la actualización en el nuevo Deployment.
- Rolling update (default): Elimina y crea los Pods uno a uno para evitar pérdida de servicio.
  - Internamente escala una a una las replicas del nuevo (+) y viejo (-) Deployment.

### **1.2. Actualizar Deployment**

Se modifica algún parámetro del Deployment, como la versión de un contenedor, número de réplicas, ... Y se ejecuta el comando, creando un nuevo rollout:

`kubectl apply -f deployment-def.yaml`

Para actualizar la imagen directamente se puede utilizar el siguiente comando, pero tendremos que actualizar nuestro fichero de definición YAML para futuros usos:

`kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1`

Ojo: `nginx=nginx:1.9.1` es el nombre del contenedor y la nueva imagen con su versión

### **1.3. Upgrades**

Internamente crea un ReplicaSet nuevo donde asigna los Pods y el viejo los va eliminando. El ReplicaSet viejo se quedará a 0 en número de replicas.

`kubectl get replicasets`

## **2. Rollback**

Volver un Deployment al estado anterior.

`kubectl rollout undo deployment/myapp-deployment`

Destuirá los pods del ReplicaSet nuevo y relanzará los del viejo.

## **3. Resumen de comandos**

- Crear: `kubectl create -f deployment-def.yaml`
- Get: `kubectl get deployments`
- Update: `kubectl apply -f deployment-def.yaml` (Lo crea si no existe)
- Status: `kubectl rollout status deployment/myapp-deployment`
- History: `kubectl rollout history deployment/myapp-deployment`
- Rollback: `kubectl rollout undo deployment/myapp-deployment`

## **4. Ejemplos: Updating a Deployment**

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