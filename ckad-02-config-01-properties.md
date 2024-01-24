# CONFIGURATION - Properties and environment variables

## **1. ConfigMap**

Create ConfigMap from command with literals:

`kubectl create configmap app-config --from-literal=USERNAME=root --from-literal=PASSWORD=root`

Create ConfigMap from command with file:

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

### **1.1. Reference in a Pod, all properties:**

Injection of environment variables from a ConfigMap.

```yaml
envFrom:
  - configMapRef:
      name: app-config
```

### **1.2. Reference a specific property in a Pod:**

Injection of one or more environment variables from a ConfigMap.

```yaml
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_COLOR
```

### **1.3. Reference in a Pod, as a file across a volume:**

Injection of environment variables from a ConfigMap into a file within a volume in the Pod.

```yaml
volumes:
  - name: app-config-volume
    configMap:
        name: app-config
```

A volume will be defined in the Pod:

```yaml
volumeMounts:
  - name: app-config-volume
    mountPath: /etc/config
```

Within `/etc/config` there will be a file for each key defined in the ConfigMap.

NOTE, if the `/etc/config` directory previously exists, its contents will be deleted.

Reference: https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#add-configmap-data-to-a-volume

## **2. Secrets**

Create Secret from command with literals:

`kubectl create secret generic app-secret --from-literal=USERNAME=root --from-literal=PASSWORD=root`

Create Secret from command with file:

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

Data can be added to the YAML in two ways:

* Encrypted as in the previous example (it is not encrypted, it is base64 encoded):

  `echo -n 'root' | base64`

  They can be deciphered in the same way, easily:

  `echo -n 'cm9vdA==' | base64 --decode`

* Unencrypted values, in plain text:

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

### **2.1. Reference in a Pod, all properties:**

Injection of environment variables from a Secret.

```yaml
envFrom:
 - secretRef:
     name: app-secret
```

### **2.2. References in a Pod, a specific property:**

Injection of one or more environment variables from a Secret.

```yaml
env:
  - name: APP_COLOR
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: APP_COLOR
```

### **2.3. Reference in a Pod, as a file across a volume:**

Injection of environment variables from a Secret into a file within a volume in the Pod.

```yaml
volumes:
  - name: app-secret-volume
    secret:
        secretName: app-secret
```

A volume will be defined in the Pod:

```yaml
volumeMounts:
  - name: app-secret-volume
    mountPath: /etc/config
```

Within `/etc/config` there will be a file for each key defined in the Secret.

NOTE, if the `/etc/config` directory previously exists, its contents will be deleted.

Reference: https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-files-from-a-pod
