# OBSERVABILITY - Verificación del estado de pods y nodos

## **1. Readiness Probes**
---

Se utilizan para comprobar que la aplicación dentro de un Pod está lista para recibir peticiones. Realizando un test a un API, obtener un HTML, lanzar un comando contra BD a través de un script interno del contenedor, ...

Cuando un Pod se ejecuta está "Ready" y los servicios pueden empezar a mandarles peticiones, pero puede que la aplicación dentro del Pod tarde unos minutos en levantarse por lo que se estará accediendo a un Pod que realmente no está listo.

Ejemplo:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
      readinessProbe:
        httpGet:
          path: /api/ready
          port: 8080
```

Hasta que el `readinessProbe` no está OK, el Pod no pasa al estado "Ready".

Tipos de test:
- HTTP Test:
    ```yaml
    readinessProbe:
      httpGet:
        path: /api/ready
        port: 8080
    ```
- TCP Test (util para BDs):
    ```yaml
    readinessProbe:
      tcpSocket:
        port: 3306
    ```
- Exec Command:
    ```yaml
    readinessProbe:
      exec:
        command:
          - cat
          - /app/is_ready
    ```

Parámetros de configuración del test:

```yaml
readinessProbe:
  httpGet:
    path: /api/ready
    port: 8080
  initialDelaySeconds: 10 # tiempo de espera inicial
  periodSeconds: 5 # cada cuanto hacer el test
  failureThreshold: 8 # numero de reintentos si falla el test, 3 intentos por defecto
```

## **2. Liveness Probes**
---

Se utilizan para comprobar que la aplicación dentro de un Pod está en marcha, es un health check, y puede seguir recibiendo peticiones. Realizando un test a un API, obtener un HTML, lanzar un comando contra BD a través de un script interno del contenedor, ...

Si la aplicación cae, el contenedor no se entera y sigue "Ready", provocando que la aplicación no responda y sigan llegando peticiones.

Ejemplo:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
      livenessProbe:
        httpGet:
          path: /api/healthy
          port: 8080
```

Mientras el `livenessProbe` está OK, el Pod seguirá en el estado "Ready".

Tipos de test:
- HTTP Test:
    ```yaml
    livenessProbe:
      httpGet:
        path: /api/healthy
        port: 8080
    ```
- TCP Test (util para BDs):
    ```yaml
    livenessProbe:
      tcpSocket:
        port: 3306
    ```
- Exec Command:
    ```yaml
    livenessProbe:
      exec:
        command:
          - cat
          - /app/is_healthy
    ```

Parámetros de configuración del test:

```yaml
livenessProbe:
  httpGet:
    path: /api/ready
    port: 8080
  initialDelaySeconds: 10 # tiempo de espera inicial
  periodSeconds: 5 # cada cuanto hacer el test
  failureThreshold: 8 # numero de reintentos si falla el test, 3 intentos por defecto
```

## **3. Logs**
---

Para Pods con un solo contenedor:

`kubectl logs -f my-pod`

Para Pods con múltiples contenedores, se debe especificar el nombre de cada contenedor:

`kubectl logs -f my-pod container-1`


## **4. Monitoring**
---

Se utiliza "Metrics Server" y está disponible a nivel de clúster. Se instala bajo demanda.

Nodo contiene el agente Kubelet (responsables de recibir instrucciones del Kubernetes API y arrancar Pods en los nodos). Kubelet contiene un subcomponente llamado cAdvisor (Component Advisor) que es responsable de recoger métricas de rendimiento de los pods y exponearlas a través del Kubelet API y tenerlas disponibles para el Metrics Server.

Para ver las métricas:
- Nodos: `kubectl top node`
- Pods: `kubectl top pod`

## **5. Debug pods**
---

Básicamente es saber dónde ver la información de lo que ocurre con el Pod en caso de error.

Eso se puede ver en el apartado "Events" dentro de la info completa del Pod con `kubectl describe pod my-pod`.

Para ver todos los eventos ocurridos en el sistema (cada namespace tiene su propio registro):

`kubectl get events`

`kubectl get events -n my-namespace`

`kubectl get events --all-namespaces`

## **6. Exec commands**

Acceso al bash:

`kubectl exec -it my-pod -- /bin/bash`

Ejecutar un comando directamente:

`kubectl exec -it my-pod -- date -s '19 APR 2012 11:14:00'`