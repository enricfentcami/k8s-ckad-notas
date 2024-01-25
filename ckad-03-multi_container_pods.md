# MULTI-CONTAINER PODS

Within a Pod there can be multiple containers, it is not usual, but there are cases where it is necessary for two "dependent" applications to be managed together and in this way, for example, they will share data volume.

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

## **Design patterns**

![Design Patterns](./resources/images/multi-container-pod-design.png)

### **Sidecar**

Example, deploy a log-server next to the webapp to send the logs to a central log server.

### **Adapter**

If that central server receives from several apps, the log format will surely be different. The Adapter container processes the logs before sending them to the central server for homogenization.

### **Ambassador**

If the application communicates with different DBs during development (dev, pre, pro) this must be managed. So the Ambassador container will act as a proxy and all connections will go to localhost and it will be responsible for redirecting them to the relevant DB.

#### Sidecar example

The containers share volume so that the sidecar can read the application logs and process them.

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
