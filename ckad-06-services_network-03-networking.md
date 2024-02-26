# SERVICES & NETWORK - Networking

https://kubernetes.io/docs/concepts/services-networking/network-policies/

NetworkPolicies are an application-centric construct which allow you to specify how a pod is allowed to communicate with various network "entities" (we use the word "entity" here to avoid overloading the more common terms such as "endpoints" and "services", which have specific Kubernetes connotations) over the network. NetworkPolicies apply to a connection with a pod on one or both ends, and are not relevant to other connections.

## **1. Ingress & Egress rules**

* Ingress: Incoming traffic to a service
* Egress: Outcoming traffic from the service to another service

The service response does not influence the concepts.

USER ---(ingress:80)-> WEB -(egress:5000)---(ingress:5000)-> BACKEND -(egress:3306)---(ingress:3306)-> DB

You can see how it works in [full example](ckad-06-services_network-04-network_policy_example.md). It's important to know:
* A policyType without specific configuration causes that type of traffic to be denied
* To allow all ingress/egress traffic, we either do not set the policyType or leave its configuration empty `-{}`

## **2. Network security / policy**

Kubernetes is configured by default to allow all traffic from any pod to other pods or services in the cluster.

Network policies allow you to create rules to allow or deny access to a pod from other pods:
* DB Pod: Allows ingress traffic from BACKEND through port 3306
* BACKEND Pod: Does not allow traffic from WEB

### Selectors and labels come into action, through rules

DB Pod defines its label:
```yaml
labels:
  role: db
```

Network policy selects the pod by the defined label:
```yaml
podSelector:
  matchLabels:
    role: db
```

The rules are defined to allow access from the BACKEND Pod (API):
```yaml
policyTypes:
 - Ingress
ingress:
- from:
  - podSelector:
      matchLabels:
        name: api-pod
  ports:
  - protocol: TCP
    port: 3306
```

_IMPORTANT: If the port is not entered in the ingress or egress config, any port is accepted._

### Examples

#### Complete example of network policy for DB Pod, allow access from BACKEND(API):

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
    ports:
    - protocol: TCP
      port: 3306
```

#### Full example of 'internal'

* Input connection from 'web' using port 8080 with 'ingress'
* Output connection to 'payroll' by 8080 and 'mysql' by 3306 with 'egress'

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
spec:
  podSelector:
    matchLabels:
      name: internal

  policyTypes:
  - Ingress
  - Egress
  
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: web
    ports:
    - protocol: TCP
      port: 8080
  
  egress:
  - to:
    - podSelector:
        matchLabels:
          name: payroll
    ports:
    - protocol: TCP
      port: 8080
  - to:
    - podSelector:
        matchLabels:
          name: mysql
    ports:
    - protocol: TCP
      port: 3306
```

#### Egress example by namespace and some port to everything

Example that restricts all Pods in Namespace space1 to only have outgoing traffic to Pods in Namespace space2 on port 80. Incoming traffic not affected.
And still allow outgoing DNS traffic on port 53 TCP and UDP, to everything.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np
  namespace: space1
spec:
  podSelector: {}
  policyTypes:
    - Egress
  egress:
    - ports:
        - port: 53
          protocol: TCP
        - port: 53
          protocol: UDP
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: space2
      ports:
        - port: 80
```

### **2.1. Comamnds**

`kubectl create -f network-policy-definition.yaml`

`kubectl get networkpolicy`

`kubectl get networkpolicy payroll-policy`

`kubectl describe networkpolicy payroll-policy`

Network policy does not appear in the `get all`:

`kubectl get networkpolicy --all-namespaces`

`kubectl get networkpolicy -n app-space`

Delete the network policy:

`kubectl delete networkpolicy payroll-policy`

NOTE: There is no shortcut to generate a networkpolicy by command, you have to create the yaml from scratch
