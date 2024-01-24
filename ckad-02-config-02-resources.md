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

By command:

`kubectl run nginx --image=nginx --requests='cpu=1,memory=1Gi' --limits='cpu=2,memory=2Gi'`

`kubectl run nginx --image=nginx --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi'`

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
