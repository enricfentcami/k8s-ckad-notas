# CONFIGURACIÓN - Seguridad

## **1. Security Context**
---

Define privilegios y control de acceso para Pods o contenedores.

Ref: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

Security context a nivel de Pod:

* `runAsUser` especifica para todos los contenedores del Pod que todos los procesos se ejecutan con el usuario indicado (user ID).
* `runAsGroup` especifica el grupo principal (group ID) para todos los procesos de cualquier contenedor del Pod.
   *  Si este atributo no se especifica, el group ID será el de root(0).
   *  Cualquier fichero generado tendrá el userID y groupID especificados.
* `fsGroup` especifica el grupo propietario de los volúmenes de datos y de los ficheros creados en ellos.

Security context a nivel de contendor:

* El funcionamiento es el mismo que para el Pod
* Sobrescribe los atributos a nivel de Pod
* Proporciona atributos adicionales como:
  * `allowPrivilegeEscalation`: Para controlar si un proceso puede obtener más privilegios que su proceso padre.
    * `allowPrivilegeEscalation: false`
  * `capabilities`: Permitir ciertos privilegios a un proces sin otorgar todos los privilegios de root.
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
---

Acceso al API de Kubernetes utilizando un token.

`kubectl create serviceaccount dashboard-sa`

`kubectl get serviceaccount`

`kubectl describe serviceaccount dashboard-sa`

Al crearlo el token se genera automáticamente a través de un secret. El nombre del secret está en el apartado "Tokens" de los datos del Service Account creado.

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
