# POD DESIGN - Labels & selectors

## **1. Labels, selectors & annotations**

https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/

- Labels = Labels that indicate characteristics
- Selectors = Selection according to characteristics
- Annotations = Metadata to enrich the object data (extend configuration of some objects)

Pod example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels: # Pod labels
    name: simple-webapp
    app: app1
    function: front-end
  annotations:
    version: 1.2.3
    company: ACME Corp.
    country: Spain
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
```

Create Pods with labels:

`kubectl run test1 --image nginx --labels tier=front,app=nginx`

`kubectl run test2 --image nginx -l tier=front,app=nginx`

Select Pods by label with kubectl:

`kubectl get pods --selector app=app1`

`kubectl get pods -l app=app1`

Select Pods by various labels:

`kubectl get pods --selector bu=finance,env=prod,tier=frontend`

`kubectl get pods -l bu=finance,env=prod,tier=frontend`

`kubectl get pods -l bu=finance,env=prod,tier!=frontend`

### Examples:

ReplicaSet example:
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-webapp
  labels: # Labels of the ReplicaSet
    name: simple-webapp
    app: app1
    function: front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app1 # To select which Pods matches this RS
  template:
    metadata:
      labels:
        app: app1 # Will match the RS selector
        function: front-end
    spec:
      containers:
        - name: simple-webapp
          image: simple-webapp
```

Example of Service, for the previous Pod:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: simple-webapp-service
spec:
  selector:
    app: app1 # To select which Pods matches with this Service
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
