# CORE - Comandos básicos y atajos

https://kubernetes.io/docs/reference/kubectl/cheatsheet/

https://kubernetes.io/docs/reference/kubectl/conventions/

## **1. Obtener objetos: Pods, Deployments, Services ...**
---

`kubectl get pods`

Con información completa (incluye en qué nodo está y su IP):

`kubectl get pods -o wide`

### **1.1. Obtener información detallada**

`kubectl describe pod my-pod`

### **1.2. Obtener la definición en YAML**

`kubectl get pod my-pod -o yaml > pod.yaml`

## **2. Crear objetos**
---

### **2.1. Crear objetos a partir de un YAML**

`kubectl create -f pod.yaml`

### **2.2. Pod**

`kubectl run --generator=run-pod/v1 nginx --image=nginx`

`kubectl run --generator=run-pod/v1 nginx-pod --image=nginx:alpine`

Generar POD Manifest YAML file "-o yaml". No crearlo en kubernetes con "--dry-run":

`kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml`

`kubectl run --generator=run-pod/v1 webapp-green --image=kodekloud/webapp-color -o yaml --dry-run=true > pod.yaml`

Pod con labels:

`kubectl run --generator=run-pod/v1 redis --image=redis:alpine --labels=tier=db`

YAML Ejemplo:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
```

### **2.3. Deployment**

`kubectl create deployment --image=nginx nginx`

Generar Deployment YAML file "-o yaml". No crearlo en kubernetes con "--dry-run":

`kubectl create deployment --image=nginx nginx --dry-run -o yaml`

**Crear deployment con replicas**

Crear yaml y añadir replicas:

`kubectl create deployment --image=nginx nginx --dry-run -o yaml > nginx-deployment.yaml`

Abrir el fichero YAML y añadir las réplicas necesarias, por defecto es 1. Luego lanzar la creación a partir de fichero.

Crear y escalar:

`kubectl create deployment webapp --image=kodekloud/webapp-color`

`kubectl scale deployment/webapp --replicas=3`

OJO: No funciona poner el parámetro en el comando, las replicas se hacen con "scale" y los labels tampoco se pueden poner por comando

`kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3 --labels=app=kodekloud-webapp-color`

YAML Ejemplo:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-deployment
  labels:
    app: webapp
spec:
  replicas: 3
  #selector:
  #  matchLabels:
  #    app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 8080
```

### **2.4. Service**

Servicio para el pod de redis:

`kubectl expose pod redis --name=redis-service --port=6379 --type=ClusterIP --selector=tier=db`

YAML Ejemplo:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
  labels:
    app: webapp
spec:
  type: NodePort
  ports:
    - port: 8080
  selector:
    app: webapp
```

### **2.5. Namespaces**

`kubectl create namespace test`

Obtener los objetos de un namespace:
- `kubectl get pods -n test`
- `kubeclt get pods --namespace=test (versión larga)`
- `kubectl get pods --all-namespaces`

YAML Ejemplo:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
  labels:
    name: my-namespace
```

## **3. Editar objetos**
---

Se abrirá el editor del sistema "vi" para editar el YAML del objeto.

`kubectl edit pod my-pod`

Si el cambio no se puede realizar en caliente se guardará una copia en un fichero temporal que se indicará. Con este fichero se podrá volver a crear el objeto haciendo un delete previo.

## **4. Eliminar objetos**
---

### Eliminar desde comando

`kubectl delete pod my-pod`

### Eliminar desde YAML

`kubectl delete -f pod.yaml`
