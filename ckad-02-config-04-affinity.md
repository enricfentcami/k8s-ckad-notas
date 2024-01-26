# CONFIGURATION - Mapping pods to nodes (affinity)

Explanation of taints and affinity: https://medium.com/@betz.mark/herding-pods-taints-tolerations-and-affinity-in-kubernetes-2279cef1f982

## **1. Taints and tolerations**

Used so that certain nodes can only contain pods with certain characteristics (repels pods). It does not mean that pods that "fit" the node will necessarily be deployed to that node.
- Taint = node
- Toleration = pod

Get available nodes:

`kubectl get nodes`

Get characteristics of a node:

`kubectl describe node node01`

### **1.1. Taint (node)**

Taint indicates that the node will only accept pods that in its `toleration` meet the key=value condition. And it will be intolerant to any other type of pod.

`kubectl taint nodes node-name key=value:taint-effect`

`kubectl taint nodes node1 app=blue:NoSchedule`
> No pod will be scheduled on node1 unless it has a matching `toleration`.

Types of 'taint-effect', indicating the behavior against intolerant pods:
- NoSchedule: A pod will not be included if it does not meet the condition
- PreferNoSchedule: Preferably a pod will not be included if it does not meet the condition
- NoExecute: A pod will not be added if it does not meet the condition and those that already exist that do not meet it will be expelled (evicted)

#### Master node as a worker node

Remove the taint from master so that it can be used as a worker:

`kubectl taint nodes master node-role.kubernetes.io/master:NoSchedule-`

Note: The `-` in `NoSchedule` indicates that this taint will be removed.

### **1.2. Toleration (pod)**

The condition is created to meet a taint, but it does not mean that a pod necessarily goes to that node.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-color
spec:
  containers:
    - name: webapp-color
      image: webapp-color
  tolerations:
    - key: "app"
      operator: "Equal"
      value: "blue"
      effect: NoSchedule
```

## **2. Label nodes**

The nodes are tagged to be able to use these selection or affinity mechanisms:

`kubectl label nodes <node-name> <label-key>=<label-value>`

`kubectl label nodes node-1 size=Large`

## **3. Node selectors**

Tags such as `size: Large` are used to indicate that a Pod is deployed to a specific node.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-color
spec:
  containers:
    - name: webapp-color
      image: webapp-color
  nodeSelector:
    size: Large
```

But be careful, conditions like `size = large OR size = medium` or `size != small` are not allowed.

## **4. Node affinity (pod)**

Improves the operation of node selectors allowing more complex operations.

https://kubernetes.io/docs/concepts/configuration/assign-pod-node/

Example, deploying a Pod on a 'large' or 'medium' node:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-color
spec:
  containers:
    - name: webapp-color
      image: webapp-color
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: In
                values:
                  - Large
                  - Medium
```

Example, deploy a Pod on a node other than 'small':
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-color
spec:
  containers:
    - name: webapp-color
      image: webapp-color
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: NotIn
                values:
                  - Small
```

The `operator: Exists` operator does not require values, it only checks that the node has the label indicating the pod in `key`.

More info about operators: https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#operators

### **4.1. Affinity types**

There are two types of node affinity:
- `requiredDuringSchedulingIgnoredDuringExecution`: The scheduler can't schedule the Pod unless the rule is met. This functions like nodeSelector, but with a more expressive syntax.
- `preferredDuringSchedulingIgnoredDuringExecution`: The scheduler tries to find a node that meets the rule. If a matching node is not available, the scheduler still schedules the Pod.

They indicate whether or not the Pod can be deployed to a node. It may be required at the time of deployment or preferably deployed to the indicated node. This value is always ignored if the Pod is already running on the node that takes the new value (tag) and they stop matching.

## **5. Taints & Tolerations VS Node Affinity**

With "Taints & Tolerations" the node is protected so that only Pods that meet the taint condition in their tolerances can be deployed, but it does NOT ensure that the Pods that meet the condition will only be deployed on that node.

With "Node Affinity" the Pods are forced to deploy to the node with which they are related, but it does NOT ensure that only those related nodes can be deployed, other Pods that do not have that type of configuration can enter.

SOLUTION: Combine both strategies. We protect the nodes with "taints" and the Pods with "affinity", and in this way we can have nodes dedicated exclusively to certain applications.