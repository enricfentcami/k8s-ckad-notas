# POD DESIGN - Labels & selectors

## **1. Labels, selectors & annotations**
---

https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/

- Labels = Etiquetas que indican características
- Selectors = Selección según características
- Annotations = Metadatos para enriquecer los datos del objeto (solo informativos)

Ejemplo de Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels: # Etiquetas del Pod
    name: simple-webapp
    app: app1
    function: front-end
  annotations:
    version: 1.2.3
    company: ACME Corp.
    country: Spain
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
```

Crear Pods con labels (o deployments):

`kubectl run --generator=run-pod/v1 test1 --image nginx --labels tier=front,app=nginx`

`kubectl run --generator=run-pod/v1 test2 --image nginx -l tier=front,app=nginx`

Seleccionar Pods por label con kubectl:

`kubectl get pods --selector app=app1`

`kubectl get pods -l app=app1`

Seleccionar Pods por varias labels:

`kubectl get pods --selector bu=finance,env=prod,tier=frontend`

`kubectl get pods -l bu=finance,env=prod,tier=frontend`

`kubectl get pods -l bu=finance,env=prod,tier!=frontend`

### Ejemplos:

Ejemplo de ReplicaSet:
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-webapp
  labels: # Etiquetas del ReplicaSet
    name: simple-webapp
    app: app1
    function: front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app1 # Para seleccionar qué Pods encajan con este RS
  template:
    metadata:
      labels:
        app: app1 # Hará match con el selector del RS
        function: front-end
    spec:
      containers:
        - name: simple-webapp
          image: simple-webapp
```
Ejemplo de Service, para el Pod anterior:
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
