# SERVICES & NETWORK - Ingress

## **1. Ingress**
---

https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/

Actúa como proxy para evitar múltiples proxies externos a Kubernetes para las redirecciones de un dominio a los servicios. Unifica el acceso en un único punto de entrada al clúster y enruta las peticiones.

Ingress controller: Aplicación como Nginx o Traefik para gestionar las redirecciones
Ingress resources: Configuración de ingress

No está incluido en Kubernetes por defecto.

### **1.2. Comandos**

`kubectl create -f igress-definition.yaml`

`kubectl get ingress`

Eliminar el ingress:

`kubectl delete ingress ingress-wear`

## **2. Ingress resources**
---

### **2.1. Ingress básico**

No se necesitan reglas si solo hay un backend en el sistema.

Todo el tráfico de entrada se manda directamente al servicio configurado

Ejemplo de Ingress:
```yaml
apiVersion: extensions/v1beta1
kind: Igress
metadata:
  name: ingress-wear
spec:
  backend:
    serviceName: wear-service
    servicePort: 80
```

### **2.2. Reglas**

Enruta el tráfico dependiendo del path que le llega al ingress, lo manda a un servicio u otro

#### Ejemplo de Ingress dentro de un mismo dominio (by Path):

* `http://www.my-store.com/wear`
* `http://www.my-store.com/watch`

```yaml
apiVersion: extensions/v1beta1
kind: Igress
metadata:
  name: ingress-wear-watch
spec:
  rules:
    - http:
        paths:
          - path: /wear
              backend:
                serviceName: wear-service
                servicePort: 80
          - path: /watch
              backend:
                serviceName: watch-service
                servicePort: 80
```

#### Ejemplo de Ingress en diferentes subdominios del dominio (by Host):

* `http://wear.my-store.com`
* `http://watch.my-store.com`

```yaml
apiVersion: extensions/v1beta1
kind: Igress
metadata:
  name: ingress-wear-watch
spec:
  rules:
    - host: wear.my-store.com
      http:
        paths:
          backend:
            serviceName: wear-service
            servicePort: 80
    - host: watch.my-store.com
      http:
        paths:
          backend:
            serviceName: watch-service
            servicePort: 80
```

#### Ejemplo de Ingress en diferentes subdominios del dominio con diferentes paths (by Host + path):

* `http://wear.my-store.com/`
* `http://wear.my-store.com/support`
* `http://watch.my-store.com/`
* `http://watch.my-store.com/movies`

```yaml
apiVersion: extensions/v1beta1
kind: Igress
metadata:
  name: ingress-wear-watch
spec:
  rules:
    - host: wear.my-store.com
      http:
        paths:
          - path: /
              backend:
                serviceName: wear-service
                servicePort: 80
          - path: /support
              backend:
                serviceName: wear-support-service
                servicePort: 80
    - host: watch.my-store.com
      http:
        paths:
          - path: /
              backend:
                serviceName: watch-service
                servicePort: 80
          - path: /movies
              backend:
                serviceName: watch-movies-service
                servicePort: 80
```
