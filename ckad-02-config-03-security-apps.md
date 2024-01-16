# CONFIGURATION - Application Security

## **1. Security Context**

Defines privileges and access control for Pods or containers.

Ref: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

Pod-level security context:

* `runAsUser` specifies for all containers in the Pod that all processes run with the indicated user (user ID).
* `runAsGroup` specifies the primary group (group ID) for all processes in any container in the Pod.
   *  If this attribute is not specified, the group ID will be that of root(0).
   *  Any file generated will have the specified userID and groupID.
* `fsGroup` specifies the group that owns the data volumes and the files created on them.

Container-level security context:

* The operation is the same as for the Pod
* Overrides Pod-level attributes
* Provides additional attributes such as:
  * `allowPrivilegeEscalation`: To control whether a process can obtain more privileges than its parent process.
    * `allowPrivilegeEscalation: false`
  * `capabilities`: Allow certain privileges to a process without granting full root privileges.
    * `add: ["NET_ADMIN", "SYS_TIME"]`

### **1.1. Pod level**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
    - name: sec-ctx-demo
      image: busybox
```

### **1.2. Containter level**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo-2
spec:
  securityContext:
    runAsUser: 1000
  containers:
    - name: sec-ctx-demo-2
      image: busybox
      securityContext:
        runAsUser: 2000
        allowPrivilegeEscalation: false
```

### **1.3. Capabilities (container level)**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo-3
spec:
  securityContext:
    runAsUser: 1000
  containers:
    - name: sec-ctx-demo-3
      image: busybox
      securityContext:
        capabilities:
          add: ["NET_ADMIN", "SYS_TIME"]
```

## **2. Service Account**

https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

Access to the Kubernetes API using a token.

`kubectl create serviceaccount dashboard-sa`

`kubectl get serviceaccount`

Abbreviated: `kubectl get sa`

`kubectl describe serviceaccount dashboard-sa`

When created, the token is automatically generated via a secret. The name of the secret is in the "Tokens" section of the created Service Account data.

`kubectl describe secret dashboard-sa-token-kbbdm`

If the app that requires the generated token is within the cluster on a Pod, you can access the token directly by mounting a volume with the service account secret:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: service-account-pod
spec:
  containers:
    - name: service-account-pod
      image: busybox
  serviceAccount: dashboard-sa
```

The secret of the service accounts are mounted in /var/run/secrets/kubernetes.io/service/account

_Note: All Pods are assigned the 'default' services account by default, but it can be disabled with the property `automountServiceAccountToken: false`_
