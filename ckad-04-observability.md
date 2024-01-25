# OBSERVABILITY - Verification of the state of pods and nodes

## **1. Probes**

## **1.1. Readiness probes**

They are used to verify that the application within a Pod is ready to receive requests. Performing a test on an API, obtaining an HTML, launching a command against the DB through an internal script of the container, ...

When a Pod is running it is "Ready" and the services can start sending requests to it, but it may take a few minutes for the application inside the Pod to wake up so you will be accessing a Pod that is not really ready.

Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
      readinessProbe:
        httpGet:
          path: /api/ready
          port: 8080
```

Until the `readinessProbe` is OK, the Pod does not go to the "Ready" state.

Test types:
- HTTP Test:
    ```yaml
    readinessProbe:
      httpGet:
        path: /api/ready
        port: 8080
    ```
- TCP Test (useful for DBs):
    ```yaml
    readinessProbe:
      tcpSocket:
        port: 3306
    ```
- Exec Command:
    ```yaml
    readinessProbe:
      exec:
        command:
          - cat
          - /app/is_ready
    ```

Test configuration parameters:

```yaml
readinessProbe:
  httpGet:
    path: /api/ready
    port: 8080
  initialDelaySeconds: 10 # initial waiting time
  periodSeconds: 5 # how often to take the test
  failureThreshold: 8 # number of retries if the test fails, 3 attempts by default
```

## **1.2. Liveness Probes**

They are used to check that the application within a Pod is running, it is a health check, and can continue receiving requests. Performing a test on an API, obtaining an HTML, launching a command against the DB through an internal script of the container, ...

If the application goes down, the container does not know and remains "Ready", causing the application to not respond and requests to continue arriving.

Example:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
      livenessProbe:
        httpGet:
          path: /api/healthy
          port: 8080
```

As long as the `livenessProbe` is OK, the Pod will remain in the "Ready" state.

Test types:
- HTTP Test:
    ```yaml
    livenessProbe:
      httpGet:
        path: /api/healthy
        port: 8080
    ```
- TCP Test (useful for DBs):
    ```yaml
    livenessProbe:
      tcpSocket:
        port: 3306
    ```
- Exec Command:
    ```yaml
    livenessProbe:
      exec:
        command:
          - cat
          - /app/is_healthy
    ```

Test configuration parameters:

```yaml
livenessProbe:
  httpGet:
    path: /api/live
    port: 8080
  initialDelaySeconds: 10 # initial waiting time
  periodSeconds: 5 # how often to take the test
  failureThreshold: 8 # number of retries if the test fails, 3 attempts by default
```

## **3. Logs**

For Pods with a single container:

`kubectl logs -f my-pod`

For Pods with multiple containers, the name of each container must be specified:

`kubectl logs -f my-pod container-1`

`-f` is "follow" to show the logs in "tail" mode

Children logs:
* Job logs: `kubect logs job/busybox-job`
* Logs of a deployment: `kubectl logs deployment/webapp-deployment`

## **4. Monitoring**

"Metrics Server" is used and is available at the cluster level. Installs on demand.

Node contains the Kubelet agent (responsible for receiving instructions from the Kubernetes API and starting Pods on the nodes). Kubelet contains a subcomponent called cAdvisor (Component Advisor) that is responsible for collecting performance metrics from the pods and exposing them through the Kubelet API and making them available to the Metrics Server.

To view metrics:
- Nodes: `kubectl top node`
- Pods: `kubectl top pod`
- 
## **5. Debug pods**

Basically it is knowing where to see the information about what happens with the Pod in case of an error.

This can be seen in the "Events" section within the complete Pod info with `kubectl describe pod my-pod`.

To view all events that have occurred on the system (each namespace has its own log):

`kubectl get events`

`kubectl get events -n my-namespace`

`kubectl get events --all-namespaces`

## **6. Exec commands**

https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/

Access to the bash:

`kubectl exec -it my-pod -- /bin/bash`

Run a command directly:

`kubectl exec -it my-pod -- date -s '19 APR 2012 11:14:00'`