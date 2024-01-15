# API DEPRECTAION

Understanding the API deprectation policy: https://kubernetes.io/docs/reference/using-api/deprecation-policy/

## **1. Deprecation Policy Rules**

* Rule #1: API elements may only be removed by incrementing the version of the API group.
* Rule #2: API objects must be able to round-trip between API versions in a given release without information loss, with the exception of whole REST resources that do not exist in some versions.
* Rule #3: An API version in a given track may not be deprecated in favor of a less stable API version.
* Rule #4a: API lifetime is determined by the API stability level
* Rule #4b: The "preferred" API version and the "storage version" for a given group may not advance until after a release has been made that supports both the new version and the previous version

List API objects: `kubectl api-resources`

List only API versions: `kubectl api-versions`

## Migrate from old version to latest version

We need to install the convert plugin: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-convert-plugin

Old ingress version:
```yaml
# Deprecated API version
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ingress-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /video-service
        pathType: Prefix
        backend:
          serviceName: ingress-svc
          servicePort: 80
```

Convert with the command `kubectl convert -f ingress-old.yaml > ingress-new.yaml`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /video-service
        pathType: Prefix
        backend:
          service:
            name: ingress-svc
            port:
              number: 80
```
