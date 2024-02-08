# SERVICES & NETWORK - Ingress

## **1. Ingress**

https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/

Acts as a proxy to avoid multiple proxies external to Kubernetes for redirects from a domain to services. It unifies access into a single entry point to the cluster and routes requests.
* Ingress controller: Application like Nginx or Traefik to manage redirects
* Ingress resources: Ingress configuration

It is not included in Kubernetes by default.

### **1.2. Comamnds**

Create object from YAML file:

`kubectl create -f igress-definition.yaml`

Get available objects:

`kubectl get ingress` or `kubectl get ing`

Ingress does not appear in the `kubectl get all` command:

`kubectl get ingress --all-namespaces`

`kubectl get ingress -n app-space`

Delete an ingress:

`kubectl delete ingress ingress-wear`

NOTE: There is no shortcut to generate an ingress by command, you have to create the yaml from 0 (using the official documentation)

## **2. Ingress resources**

### **2.1. Basic Ingress**

No rules are needed if there is only one backend in the system.

All incoming traffic is sent directly to the configured service.

Ingress example:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  ingressClassName: nginx
  backend:
    service:
      name: wear-service
      port:
        number: 80
```

### **2.2. Rules**

Routes the traffic depending on the path that reaches the ingress, sending it to one service or another.

Path type is required and Prefix is commonly used, check documentation for more info: https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types

#### Ingress example within the same domain (by Path):

* `http://www.my-store.com/wear`
* `http://www.my-store.com/watch`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /wear
            pathType: Prefix
            backend:
              service:
                name: wear-service
                port:
                  number: 80
          - path: /watch
            pathType: Prefix
            backend:
              service:
                name: watch-service
                port:
                  number: 80
```

#### Ingress example on different subdomains of the domain (by Host):

* `http://wear.my-store.com`
* `http://watch.my-store.com`

We use the `host` attribute to specify the full domain.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  ingressClassName: nginx
  rules:
    - host: wear.my-store.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: wear-service
                port:
                  number: 80
    - host: watch.my-store.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: watch-service
                port:
                  number: 80
```

#### Example of Ingress on different subdomains of the domain with different paths (by Host + path):

* `http://wear.my-store.com/`
* `http://wear.my-store.com/support`
* `http://watch.my-store.com/`
* `http://watch.my-store.com/movies`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  ingressClassName: nginx
  rules:
    - host: wear.my-store.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: wear-service
                port:
                  number: 80
          - path: /support
            pathType: Prefix
            backend:
              service:
                name: wear-support-service
                port:
                  number: 80
    - host: watch.my-store.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: watch-service
                port:
                  number: 80
          - path: /movies
            pathType: Prefix
            backend:
              service:
                name: watch-movies-service
                port:
                  number: 80
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
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rewrite
  annotations:
    # The call to "/pay" is internally redirected to "/" (rewrite-target)
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /pay
            pathType: Prefix
            backend:
              service:
                name: pay-service
                port:
                  number: 8282
```

### In another example given here, this could also be:

replace("/something(/|$)(.*)", "/$2")

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rewrite
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  ingressClassName: nginx
  rules:
    - host: rewrite.bar.com
      http:
        paths:
          - path: /something(/|$)(.*)
            pathType: Prefix
            backend:
              service:
                name: http-svc
                port:
                  number: 80
```