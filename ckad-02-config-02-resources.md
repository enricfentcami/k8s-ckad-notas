# CONFIGURATION - System resources

## **1.Required system resources (Resource Request)**

For CKAD you only need to know how to specify the resources.

Define the resource requests in the Pod level:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-color
spec:
  containers:
    - name: webapp-color
      image: webapp-color
  resources:
    requests:
      memory: "1Gi"
      cpu: 1
    limits:
      memory: "2Gi"
      cpu: 2
```

What happens if the limit is exceeded?
- The CPU consumed by the Pod cannot exceed the limits.
- The memory can exceed, but if it is constant the Pod will be terminated.

### **CPU values**

- m = Milli (Millicore)
- 1 CPU = 1000m
  - 1 AWS vCPU
  - 1 GCP Core
  - 1 Azure Core
  - 1 Hyperthread

### **Memory values**

- Gi = Gibibyte
- G = Gigabyte
- Mi = Mebibyte
- M = Megabyte
- Ki = Kibibyte = 1024 bytes
- K = Kilobyte = 1000 bytes

## Note on default resource requirements and limits

In order to have a default CPU request and memory you must have first set those as default values for request and limit by creating a LimitRange in that namespace.

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
```

https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
  - default:
      cpu: 1
    defaultRequest:
      cpu: 0.5
    type: Container
```

https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/


References:

https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource