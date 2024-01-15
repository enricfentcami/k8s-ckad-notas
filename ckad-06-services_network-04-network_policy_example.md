# EXAMPLE NETWORK POLICY

![Design Patterns](./resources/images/ckad-networkpolicy.png)

## **PODS**

`kubectl run nginx-1 --image=nginx`

`kubectl run nginx-2 --image=nginx`

`kubectl run nginx-3 --image=nginx`

`kubectl run nginx-4 --image=nginx`

## **SERVICES**

`kubectl expose pod nginx-1 --name=nginx-service-1 --port=80 --type=NodePort`

`kubectl expose pod nginx-2 --name=nginx-service-2 --port=80`

`kubectl expose pod nginx-3 --name=nginx-service-3 --port=80`

`kubectl expose pod nginx-4 --name=nginx-service-4 --port=80`

## **NETWORK POLICIES**

Network policy del pod/service 1:
* Acceso desde el exterior del nodo
* Solo puede acceder a pod/service 2 por puerto 80

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: nginx-1-policy
spec:
  podSelector:
    matchLabels:
      run: nginx-1

  # Un policyType sin configuración provoca la denegación de ese tipo de tráfico
  policyTypes:
  - Ingress
  - Egress

  # Para permitir todo el tráfico ingress, si ponemos el policyType
  # Es lo mismo que no poner el policyType ni esta config
  ingress: 
  - {}

  egress:
  - to:
    - podSelector:
        matchLabels:
          run: nginx-2
    ports:
    - protocol: TCP
      port: 80
```

Network policy del pod/service 2:
* Acceso solo desde pod/service 1 por el puerto 80
* Solo puede acceder a pod/service 3 por puerto 80

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: nginx-2-policy
spec:
  podSelector:
    matchLabels:
      run: nginx-2

  policyTypes:
  - Ingress
  - Egress

  ingress:
  - from:
    - podSelector:
        matchLabels:
          run: nginx-1
    ports:
    - protocol: TCP
      port: 80

  egress:
  - to:
    - podSelector:
        matchLabels:
          run: nginx-3
    ports:
    - protocol: TCP
      port: 80
```

Network policy del pod/service 3:
* Acceso solo desde pod/service 2 por el puerto 80
* No puede acceder a otros Pods

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: nginx-3-policy
spec:
  podSelector:
    matchLabels:
      run: nginx-3

  policyTypes:
  - Ingress
  - Egress # Egress sin config deniega el tráfico de salida

  ingress:
  - from:
    - podSelector:
        matchLabels:
          run: nginx-2
    ports:
    - protocol: TCP
      port: 80
```

## **PRUEBAS**

1. Se crean todos los Pods, services y network policy.

2. Listado de los services:
```
NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)
nginx-service-1       NodePort    10.244.172.27    <none>        80:31517/TCP
nginx-service-2       ClusterIP   10.244.247.5     <none>        80/TCP
nginx-service-3       ClusterIP   10.244.199.185   <none>        80/TCP
nginx-service-4       ClusterIP   10.244.96.152    <none>        80/TCP
```

3. Acceso a cada uno de los Pods para verificar el tráfico
   
   `kubectl exec -it nginx-1 -- /bin/bash`

4. Ejecución de los CURL para verificar el acceso:
   1. Pod 1: `curl 10.244.172.27`
   2. Pod 2: `curl 10.244.247.5`
   3. Pod 3: `curl 10.244.199.185`
   4. Pod 4: `curl 10.244.96.152`

   