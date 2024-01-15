# SERVICES & NETWORK - Ingress

## **1. Ingress**

https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/

Actúa como proxy para evitar múltiples proxies externos a Kubernetes para las redirecciones de un dominio a los servicios. Unifica el acceso en un único punto de entrada al clúster y enruta las peticiones.

Ingress controller: Aplicación como Nginx o Traefik para gestionar las redirecciones
Ingress resources: Configuración de ingress

No está incluido en Kubernetes por defecto.

### **1.2. Comandos**

`kubectl create -f igress-definition.yaml`

`kubectl get ingress`

Ingress no sale en el `get all`:

`kubectl get ingress --all-namespaces`

`kubectl get ingress -n app-space`

Eliminar el ingress:

`kubectl delete ingress ingress-wear`

OJO: No existe un atajo para generar un ingress por comando, hay que crear el yaml desde 0

## **2. Ingress resources**

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

## **3. FAQ - What is the rewrite-target option?**

https://kubernetes.github.io/ingress-nginx/examples/rewrite/

Different ingress controllers have different options that can be used to customise the way it works. NGINX Ingress controller has many options that can be seen here. I would like to explain one such option that we will use in our labs. The Rewrite target option.

Our watch app displays the video streaming webpage at `http://<watch-service>:<port>/`

Our wear app displays the apparel webpage at `http://<wear-service>:<port>/`

We must configure Ingress to achieve the below. When user visits the URL on the left, his request should be forwarded internally to the URL on the right. Note that the /watch and /wear URL path are what we configure on the ingress controller so we can forwarded users to the appropriate application in the backend. The applications don't have this URL/Path configured on them:

`http://<ingress-service>:<ingress-port>/watch` --> `http://<watch-service>:<port>/`

`http://<ingress-service>:<ingress-port>/wear` --> `http://<wear-service>:<port>/`


Without the rewrite-target option, this is what would happen:

`http://<ingress-service>:<ingress-port>/watch` --> `http://<watch-service>:<port>/watch`

`http://<ingress-service>:<ingress-port>/wear` --> `http://<wear-service>:<port>/wear`


Notice watch and wear at the end of the target URLs. The target applications are not configured with /watch or /wear paths. They are different applications built specifically for their purpose, so they don't expect /watch or /wear in the URLs. And as such the requests would fail and throw a 404 not found error.

To fix that we want to "ReWrite" the URL when the request is passed on to the watch or wear applications. We don't want to pass in the same path that user typed in. So we specify the rewrite-target option. This rewrites the URL by replacing whatever is under rules->http->paths->path which happens to be /pay in this case with the value in rewrite-target. This works just like a search and replace function.

### For example: replace(path, rewrite-target)

In our case: replace("/path","/")

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /pay      # La llamada a "/pay" se redirige internamente a "/" (rewrite-target)
        backend:
          serviceName: pay-service
          servicePort: 8282
```

### In another example given here, this could also be:

replace("/something(/|$)(.*)", "/$2")

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: rewrite
  namespace: default
spec:
  rules:
  - host: rewrite.bar.com
    http:
      paths:
      - backend:
          serviceName: http-svc
          servicePort: 80
        path: /something(/|$)(.*)
```