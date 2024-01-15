# SERVICES & NETWORK - Services

## **1. Services**

https://kubernetes.io/docs/concepts/services-networking/service/

Exponer una aplicación de un Pod como un servicio de red, para que pueda ser accedido por otros Pods.

Service Types:
* NodePort service: Escucha en un puerto del nodo y redirige las peticiones de ese puerto al puerto del Pod de la app. **Exponer hacia fuera un servicio.**
* Cluster IP: IP virtual dentro del clúster para habilitar la comunicación **interna** entre diferentes servicios como: De un grupo de front-end servers a un grupo de back-end servers. Valor por defecto.
* Load balancer: Balanceo de carga. Distribuir carga entre distintos Pods.

IMPORTANTE: https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types

### **1.2. Comandos**

`kubectl create -f service-definition.yaml`

`kubectl get services`

#### Crear servicio desde comando (exponer pod o deployment):

`kubectl expose deployment webapp-deployment --type=NodePort --port=80 --name=webapp-service -n webapp-ns --dry-run -o yaml > service-webapp.yaml`

                    ^             ^                 ^             ^               ^                 ^           ^
                Qué tipo     Nombre deploy.      Tipo serv.     Puerto       Nombre serv.       Namespace     No ejecutarlo

Importante:
* El selector pone por defecto el `name: nginx-ingress` que es el label del deployment
* El namespace no lo incluye en el YAML, debe hacerse manualmente
* El NodePort fijo '30080' se pone manualmente


Eliminar el service:

`kubectl delete service myapp-service`

## **2. NodePort**

El puerto que se abre en el nodo va del 30000 al 32767. El puerto se abre en el clúster (nodos worker): `http://192.168.1.2:30008`

Ejemplo de Service:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort      # Default es ClusterIP
  ports:
    - targetPort: 80  # Si no se especifica se coge el mismo que 'port'
      port: 80        # Obligatorio
      nodePort: 30008 # Si no se especifica se asigna automáticamente
  selector:
    app: myapp        # Labels del Pod
    type: front-end
```

#### Redirección del tráfico:

Al tener varios Pods que encajan en el selector del Service, balancea aleatoriamente las peticiones que le llegan. Tiene un load balancer interno.

Si los Pods están distribuidos en diferentes nodos del clúster, Kubernetes crea el servicio transversal a los workers y es indiferente a qué worker se acceda para que balancee las peticiones entre todos los Pods de todos los nodos.

Acceso al Pod desde la red interna con IP del worker:

`curl http://192.168.1.2:30008`

## **2. ClusterIP**

Se asigna una IP del clúster al servicio como punto de acceso y se publica el puerto indicado en el servicio: `http://10.96.127.123:80`

Ejemplo de Service:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: back-end-service

spec:
  type: ClusterIP     # Default es ClusterIP, no hace falta especificarlo
  ports:
    - targetPort: 80  # Si no se especifica se coge el mismo que 'port'
      port: 80        # Obligatorio
  selector:
    app: myapp        # Labels del Pod
    type: back-end
```