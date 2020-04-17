# CONFIGURACIÓN - Seguridad

## **1. Security Context**
---

Ref: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

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
---

Acceso al API de Kubernetes utilizando un token.

`kubectl create serviceaccount dashboard-sa`

`kubectl get serviceaccount`

`kubectl describe serviceaccount dashboard-sa`

Al crearlo el token se genera automáticamente a través de un secret. El nombre del secret está enel apartado "Tokens" de los datos del Service Account creado.

`kubectl describe secret dashboard-sa-token-kbbdm`

_Nota: Los permisos de acceso del Service Account quedan fuera del CKAD (role based access control, RBAC)_

Si la app que requiere el token generado está dentro del clúster en un Pod, puede acceder al token directamente montando un volumen con el secret del service account:

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

_Nota: Todos los Pods tienen asignado por defecto el services account 'default', pero se puede desactivar con la propiedad `automountServiceAccountToken: false`_
