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

`kubectl run nginx --image=nginx` -> Crea un pod

Añadir argumentos `args` al contendor, que se ejecutarán. Depués del `--` son los argumentos:
`kubectl run pod1 --image=bash -- bash -c "hostname >> /tmp/hostname && sleep 1d"` 

Generar POD Manifest YAML file "-o yaml". No crearlo en kubernetes con "--dry-run=client":

`kubectl run nginx --image=nginx --dry-run=client -o yaml`

`kubectl run webapp-green --image=kodekloud/webapp-color -o yaml --dry-run=client > pod.yaml`

*Restart Never*:

`kubectl run nginx --image=nginx --restart=Never --dry-run=client -o yaml > pod.yaml` -> K8s 1.18+

_Importante: con `--restart=Never` en versiones antiguas genera el Pod, sin eso genera un Deployment (DEPRECATED)_


Pod con labels:

`kubectl run redis --image=redis:alpine --labels=tier=db`

YAML Ejemplo:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
  labels:
    tier: frontend
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
```

Si no se especifica una label tendrá por defecto `run=nginx` (nombre del pod)

### **2.3. ReplicaSet**

ReplicaSet no se puede generar por comando, hay que utilizar YAML.

Escalar y modificar el RS por comando:

`kubectl scale replicaset webapp --replicas=3`

YAML Ejemplo:
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-1
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
```

OJO: Si tenemos PODs creados anteriormente que cumplen con el `selector` se eliminarán los que "sobren" para cumplir con el número de replicas. Si hay menos que el número de replicas se crearán solo los necesarios para cumplir la condición.

### **2.3. Deployment**

`kubectl create deployment --image=nginx nginx`

Generar Deployment YAML file "-o yaml". No crearlo en kubernetes con "--dry-run=client":

`kubectl create deployment --image=nginx nginx --dry-run=client -o yaml`

**Crear deployment con replicas**

Crear yaml y añadir replicas:

`kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml`

Abrir el fichero YAML y añadir las réplicas necesarias, por defecto es 1. Luego lanzar la creación a partir de fichero.

Crear con replicas:

`kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3`

Crear y escalar:

`kubectl create deployment webapp --image=kodekloud/webapp-color`

`kubectl scale deployment/webapp --replicas=3`

OJO: Los labels no se pueden poner por comando, esto no fuciona:

`kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3 --labels=app=kodekloud-webapp-color` ->  `unknown flag: --labels`

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
  selector:
    matchLabels:
      app: webapp
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

Ejemplo de Pod y Servicio por comando:

`kubectl run test-web --image=nginx -l app=front` 

`kubectl expose pod test-web --name=test-web-service --port=80 --selector=app=front`

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: front
  name: test-web
  namespace: eebb
spec:
  containers:
  - image: nginx
    name: test-web

---

apiVersion: v1
kind: Service
metadata:
  labels:
    app: front
  name: test-web-service
  namespace: eebb
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: front
```

### **2.5. Namespaces**

`kubectl create namespace test`

Obtener los objetos de un namespace:
- `kubectl get pods -n test`
- `kubeclt get pods --namespace=test` (versión larga)
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

Forzar el borrado:

`kubectl delete pod my-pod --force`

### Eliminar desde YAML

`kubectl delete -f pod.yaml`

## **5. Contexto y namespaces**

Ver datos de contexto y namespace por defecto:

`kubectl config get-contexts`

Establecer el contexto por defecto, cambio de clúster:

`kubectl config use-context my-cluster-name`

Establecer un nuevo namespace por defecto para el contexto (clúster) actual:

`kubectl config set-context --current --namespace=ggckad-s2`

IMPORTANTE: Muy útil en el examen ya que las preguntas tienen su propio namespace
