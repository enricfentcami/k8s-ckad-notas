# CONFIGURACIÓN - Propiedades y variables de entorno

## **1. ConfigMap**

Crear ConfigMap desde comando con literales:

`kubectl create configmap app-config --from-literal=USERNAME=root --from-literal=PASSWORD=root`

Crear ConfigMap desde comando con fichero:

`kubectl create configmap app-config --from-file=application.properties`

Sample ConfigMap YAML file:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_VAR1: VALUE_1
```

### **1.1. Referenciar en un Pod, todas las propiedades:**

Inyección de las variables de entorno desde un ConfigMap.

```yaml
envFrom:
  - configMapRef:
      name: app-config
```

### **1.2. Referenciar en un Pod, una propiedad concreta:**

Inyección de una o varias variables de entorno desde un ConfigMap.

```yaml
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_COLOR
```

### **1.3. Referenciar en un Pod, como fichero a través de un volumen:**

Inyección de las variables de entorno desde un ConfigMap en un fichero dentro de un volumen en el Pod.

```yaml
volumes:
  - name: app-config-volume
    configMap:
        name: app-config
```

En el Pod se definirá un volumen:

```yaml
volumeMounts:
  - name: app-config-volume
    mountPath: /etc/config
```

Dentro de `/etc/config` existirá un fichero por cada clave definida en el ConfigMap.

OJO, que si existe previamente el directorio `/etc/config` será eliminado su contenido.

Referencia: https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#add-configmap-data-to-a-volume

## **2. Secrets**

Crear Secret desde comando con literales:

`kubectl create secret generic app-secret --from-literal=USERNAME=root --from-literal=PASSWORD=root`

Crear Secret desde comando con fichero:

`kubectl create secret generic app-secret --from-file=application-secret.properties`

Sample Secret YAML file:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-config-secret
  labels:
    app: database
data:
  # Encoded in Base64
  USER: cm9vdA==
  PASSWORD: ZG9ja2Vy
```

En el YAML se pueden añadir los datos de dos formas:

* Cifrados como en el ejemplo anterior (el cifrado es débil con base64):

  `echo -n 'root' | base64`

  Se pueden descifrar del mismo modo, fácil:

  `echo -n 'cm9vdA==' | base64 --decode`

* En claro:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-config-secret
  labels:
    app: database
stringData:
  USER: root
  PASSWORD: password
```

### **2.1. Referenciar en un Pod, todas las propiedades:**

Inyección de las variables de entorno desde un Secret.

```yaml
envFrom:
 - secretRef:
     name: app-secret
```

### **2.2. Referencias en un Pod, una propiedad concreta:**

Inyección de una o varias variables de entorno desde un Secret.

```yaml
env:
  - name: APP_COLOR
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: APP_COLOR
```

### **2.3. Referenciar en un Pod, como fichero a través de un volumen:**

Inyección de las variables de entorno desde un Secret en un fichero dentro de un volumen en el Pod.

```yaml
volumes:
  - name: app-secret-volume
    secret:
        secretName: app-secret
```

En el Pod se definirá un volumen:

```yaml
volumeMounts:
  - name: app-secret-volume
    mountPath: /etc/config
```

Dentro de `/etc/config` existirá un fichero por cada clave definida en el Secret.

OJO, que si existe previamente el directorio `/etc/config` será eliminado su contenido.

Referencia: https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-files-from-a-pod
