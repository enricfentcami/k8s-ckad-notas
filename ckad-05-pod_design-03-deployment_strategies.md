# POD DESIGN - Deployment strategies: Blue/green and canary

These are not strategies implemented natively in Kubernetes, but you have to know how to carry them out with the available tools.

## **1. Bue/green**
---

Blue/Green will maintain two Deployments with the current version of the application and the new version.
Blue si called the old version and Green the new version, and once Green is available all traffic from Blue will be redirected to Green.

1. Create a Service with tag selection `version: v1`
2. Create a Deployment called `blue` with the tag `version: v1`
3. Create a Deployment called `green` with the tag `version: v2`
4. Incoming traffic goes to the `blue` Deployment because of the `version: v1` tag
5. If we change the Service tag selector to `version: v2` all traffic will be redirected to `green` Deployment

YAML examples:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
  labels:
    app: myapp
    type: front-end
spec:
  replicas: 5
  selector:
    matchLabels:
      verison: v1
  template:
    metadata:
      labels:
        version: v1
    spec:
      containers:
      - name: app-container
        image: myapp-image:1.0
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
  labels:
    app: myapp
    type: front-end
spec:
  replicas: 5
  selector:
    matchLabels:
      verison: v2
  template:
    metadata:
      labels:
        version: v2
    spec:
      containers:
      - name: app-container
        image: myapp-image:2.0
---
apiVersion: v1
kind: Service
metadata:
  name: my-service
  labels:
    app: myapp
spec:
  selector:
    # Version v2 to change from blue to green
    version: v1
```

## **2. Canary**
---

The Canary strategy is based on having the current version published and publishing the new version at the same time, but only redirecting a part of the traffic to the new version to be able to carry out tests with real users.

Once the new application is tested and validated, the new version will be deployed using RollingUpdate and the Deployment Canary will be eliminated.

1. Create a Service with tag selection `version: v1`
2. Create a Deployment called `principal` with the tag `version: v1`
3. Create a Deployment called `canary` with the tag `version: v2`
4. Route a small percentage of traffic to `version: v2` to work with both versions
   1. Add an app label to both Deployments and Service called `app: front-end` and remove the `version` tag from Service
   2. Set a small number of replicas to the `canary` Deployment
5. Once tests are ok, do a RollingUpdate to `primary` Deployment with new version
6. Then delete the `canary` Deployment

YAML examples:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-primary
  labels:
    app: myapp
    type: front-end
spec:
  replicas: 5
  selector:
    matchLabels:
      app: front-end
  template:
    metadata:
      labels:
        version: v1
        app: front-end
    spec:
      containers:
      - name: app-container
        image: myapp-image:1.0
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-canary
  labels:
    app: myapp
    type: front-end
spec:
  replicas: 1
  selector:
    matchLabels:
      app: front-end
  template:
    metadata:
      labels:
        version: v2
        app: front-end
    spec:
      containers:
      - name: app-container
        image: myapp-image:2.0
---
apiVersion: v1
kind: Service
metadata:
  name: my-service
  labels:
    app: myapp
spec:
  selector:
    app: front-end
```
