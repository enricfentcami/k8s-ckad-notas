# OTHERS - Horizontal Pod Autoscaling

NOTE: Not included in the CKAD exam

https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/

https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#autoscale

## **1. Example**

Deploy service and test deployment (https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#run-expose-php-apache-server):

`kubectl apply -f https://k8s.io/examples/application/php-apache.yaml`

Create autoscaling (https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#create-horizontal-pod-autoscaler):

`kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10`

_Get it to YAML without executing it:_

`kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10 --dry-run -o yaml > autoscaler.yaml`

Check it:

`kubectl get hpa`

Add work load to the Pod (https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#increase-load):
_(Ejecutar en otra consola)_

`kubectl run -it --rm load-generator --image=busybox /bin/sh`

Run infinite loop:

`while true; do wget -q -O- http://php-apache; done`

After a minute you can see that the Pod is climbing:

`kubectl get hpa`

```console
NAME         REFERENCE               TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   250%/50%   1         10        6          141m
```

`kubectl get deployment php-apache`

```console
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
php-apache   6/6     6            6           147m
```

When you stop the loading Pod execution, after a few minutes, you see that the number of replicas drops:

`kubectl get hpa`

```console
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%    1         10        1          150m
```
