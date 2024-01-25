# SERVICES & NETWORK - Services

## **1. Services**

https://kubernetes.io/docs/concepts/services-networking/service/

Expose a Pod application as a network service, so that it can be accessed by other Pods.

Service Types:
* NodePort service: Listens on a node port and redirects requests from that port to the app's Pod port. **Expose a service outside the cluster.**
* Cluster IP: Virtual IP within the cluster to enable **internal** communication between different services such as: From a group of front-end servers to a group of back-end servers. Default value.
* Load balancer: Load balancing. Distribute load between different Pods.

IMPORTANT: https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types

### **1.2. Commands**

Create a service from a YAML file: 

`kubectl create -f service-definition.yaml`

Get available services from current namespace:

`kubectl get services`

#### Create service from command (expose pod or deploy):

`kubectl expose deployment webapp-deployment --type=NodePort --port=80 --name=webapp-service -n webapp-ns --dry-run=client -o yaml > service-webapp.yaml`

                    ^             ^                 ^             ^               ^                 ^           ^
                Expose type   Deploy name      Service type      Port       Service name        Namespace    Dont execute

Important:
* The selector defaults to `name: nginx-ingress`, which is the deployment label
* The namespace is included in the YAML because we specify it with `-n webapp-ns`. If not it doesn't add any namespace spec.
* Fixed NodePort like '30080' will be set manually. If not specified the service will have an available port.

Delete the service:

`kubectl delete service myapp-service`

## **2. NodePort**

The port opened on the node ranges from 30000 to 32767. The port opened on the cluster (worker nodes): `http://192.168.1.2:30008`

Service example:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort      # Default is ClusterIP
  ports:
    - targetPort: 80  # If it is not specified, the same as 'port' is taken
      port: 80        # Mandatory
      nodePort: 30008 # If not specified, it is assigned automatically.
  selector:
    app: myapp        # Pod labels (also specified in the Deployment)
    type: front-end
```

#### Traffic redirection:

By having several Pods that matches the Service selector, it randomly balances the requests that come to it. It has an internal load balancer.

If the Pods are distributed in different nodes of the cluster, Kubernetes creates the service transversal to the workers and it does not matter which worker is accessed so that it balances the requests between all the Pods of all the nodes.

Access to the Pod from the internal network with worker IP:

`curl http://192.168.1.2:30008`

## **2. ClusterIP**

A cluster IP is assigned to the service as an access point and the indicated port is published to the service: `http://10.96.127.123:80`

Service example:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: back-end-service

spec:
  type: ClusterIP     # Default is ClusterIP, no need to specify
  ports:
    - targetPort: 80  # If it is not specified, the same as 'port' is taken
      port: 80        # Mandatory
  selector:
    app: myapp        # Pod labels (also specified in the Deployment)
    type: back-end
```