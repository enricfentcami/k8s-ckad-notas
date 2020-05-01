# POD DESIGN - Jobs & CronJobs

## **1. Jobs**
---

https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/

Por defecto, Kubernetes al ejecutar un pod que ejecuta una acción y termina lo ejecuta una y otra vez debido a la política `restartPolicy` que por defecto es `Always`. Para hacer que se ejecute una vez se haría con la policy `Never`.

Ejemplo de Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: math-pod
spec:
  containers:
    - name: math-pod
      image: ubuntu
      command: ['expr','3','+','2']
    restartPolicy: Never
```

### **1.1. Definir un Job**

Se necesita un manager que pueda crear tantos Pods como sean requeridos para realizar una tarea y que finalice, por ejemplo en procesamiento paralelo de datos.

Ejemplo de Job:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  template:
    spec:
      containers:
        - name: math-pod
          image: ubuntu
          command: ['expr','3','+','2']
        restartPolicy: Never
```

### **1.2. Comandos**

`kubectl create -f job-definition.yaml`

`kubectl get jobs`

Se puede ver el Pod creado con el STATUS `completed` con 0 RESTARTS

`kubectl get pods`

En este caso el resultado está en el log del Pod:

`kubectl logs math-add-job-xxxxx`

> `5`

Eliminar el job:

`kubectl delete job math-add-job`

#### Ejemplos en la vida real

Procesamiento de imágenes, reports + envío de email...

### **1.3. Multiple Pods**

Se indica el número de SUCCESSFUL que se quiere tener. Los pods se generan secuencialmente cuando el anterior ha finalizado.

Ejemplo de Job:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  completions: 3
  template:
    spec:
      containers:
        - name: math-pod
          image: ubuntu
          command: ['expr','3','+','2']
        restartPolicy: Never
```

Cuando un job falla se ejecuta otro hasta que se completan los SUCCESSFUL que se habían definido con `completions`

### **1.4. Multiple Pods en paralelo**

Se indica el número de Pods a crear en paralelo con `parallelism`.

Ejemplo de Job:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  completions: 3
  parallelism: 3
  template:
    spec:
      containers:
        - name: math-pod
          image: ubuntu
          command: ['expr','3','+','2']
        restartPolicy: Never
```

Creará 3 Pods a la vez y si alguno falla volverá a crearlo hasta que se completen los indicados en `completions`.

### **1.5. Limitar el número de intentos**

Se permite máximo el número de fallos indicados por `backoffLimit`.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  completions: 3
  parallelism: 3
  backoffLimit: 6
  template:
    spec:
      containers:
        - name: math-pod
          image: ubuntu
          command: ['expr','3','+','2']
        restartPolicy: Never
```

## **2. CronJobs**
---

Jobs programados como 'crontab'.

Ejemplo de CronJob:
```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: reporting-cron-job
spec:
  schedule: "*/1 * * * *" # Cron expression
  jobTemplate:
    spec:
      completions: 3
      parallelism: 3
      template:
        spec:
          containers:
            - name: reporting-tool
              image: reporting-tool
            restartPolicy: Never
```

### **2.2. Comandos**

`kubectl create -f cron-job-definition.yaml`

`kubectl get cronjob`

Eliminar el cron job:

`kubectl delete cronjob reporting-cron-job`
