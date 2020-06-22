# MULTI-CONTAINER PODS

Dentro de un Pod pueden haber mútiples contenedores, no es lo habitual, pero hay casos que es necesario que dos aplicaciones "dependientes" se gestionen juntas y de ese modo, por ejemplo, compartirán volumen de datos.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
    - name: log-agent
      image: log-agent
```

https://matthewpalmer.net/kubernetes-app-developer/articles/multi-container-pod-design-patterns.html

## **Patrones de diseño**

![Design Patterns](./resources/images/multi-container-pod-design.png)

### **Sidecar**

Ejemplo, desplegar un log-server junto al webapp para mandar los logs a un servidor central de logs.

### **Adapter**

Si ese servidor central recibe de varias apps, seguramente el formato de logs sea diferente. El contenedor Adapter procesa los logs antes de enviarlos al servidor central para homogeneizar.

### **Ambassador**

Si la aplicación se comunica con diferentes BDs durante el desarrollo (dev, pre, pro) hay que gestionar esto. Por lo que el contenedor Ambassador hará de proxy y todas las conexiones irán a localhost y se encargará de redirigirlas a la BD pertinente.

#### Ejemplo Sidecar

Los contenedores comparte volumen para que el sidecar pueda leer los logs de la aplicación y procesarlos.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: counter
spec:
  containers:
  - name: count
    image: busybox
    args:
    - /bin/sh
    - -c
    - >
      i=0;
      while true;
      do
        echo "$i: $(date)" >> /var/log/1.log;
        echo "$(date) INFO $i" >> /var/log/2.log;
        i=$((i+1));
        sleep 1;
      done
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  - name: count-agent
    image: k8s.gcr.io/fluentd-gcp:1.30
    env:
    - name: FLUENTD_ARGS
      value: -c /etc/fluentd-config/fluentd.conf
    volumeMounts:
    - name: varlog
      mountPath: /var/log
    - name: config-volume
      mountPath: /etc/fluentd-config
  volumes:
  - name: varlog
    emptyDir: {}
  - name: config-volume
    configMap:
      name: fluentd-config
```
