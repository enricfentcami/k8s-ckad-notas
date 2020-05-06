# OTHERS - Horizontal Pod Autoscaling

https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/

https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#autoscale

## **1. Ejemplo**
---

Desplegar servicio y deployment de prueba (https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#run-expose-php-apache-server):

`kubectl apply -f https://k8s.io/examples/application/php-apache.yaml`

Crear autoescalado (https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#create-horizontal-pod-autoscaler):

`kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10`

_Sacarlo a YAML sin ejecutarlo:_

`kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10 --dry-run -o yaml > autoscaler.yaml`

Comprobarlo:

`kubectl get hpa`

Añadir carga al POD (https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#increase-load):
_(Ejecutar en otra consola)_

`kubectl run --generator=run-pod/v1 -it --rm load-generator --image=busybox /bin/sh`

Ejecutar bucle infinito:

`while true; do wget -q -O- http://php-apache; done`

Tras un minuto se ve que se está escalando el POD:

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

Al parar la ejecución de POD de carga, tras unos minutos, se ve que baja el número de replicas: 

`kubectl get hpa`

```console
NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   0%/50%    1         10        1          150m
```
