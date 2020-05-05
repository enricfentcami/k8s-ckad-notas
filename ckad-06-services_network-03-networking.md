# SERVICES & NETWORK - Networking

https://kubernetes.io/docs/concepts/services-networking/network-policies/

## **1. Ingress & Egress rules**
---

* Ingress: Tráfico de entrada a un servicio
* Egress: Tráfico de salida del servicio hacia otro servicio

La respuesta del servicio no influyen en los conceptos.


USUARIO ---(ingress:80)-> WEB -(egress:5000)---(ingress:5000)-> BACKEND -(egress:3306)---(ingress:3306)-> DB

## **2. Network security / policy**
---

Kubernetes está configurado por defecto para permitir todo el tráfico ("all allow") desde cualquier pod a otros pods o servicios en el clúster.

Network policies permite crear reglas para permitir o denegar el acceso a un pod desde otros pods:
* DB Pod: Permite tráfico ingress desde BACKEND a través del puerto 3306
* DB Pod: No permite tráfico desde WEB

### Selectores y labels entran en acción, mediante reglas

DB Pod define su label:
```yaml
labels:
  role: db
```

Network policy selecciona el pod por la label definida:
```yaml
podSelector:
  matchLabels:
    role: db
```

Se definen las reglas para permitir acceso desde el Pod de BACKEND(API):
```yaml
policyTypes:
 - Ingress
ingress:
- from:
  - podSelector:
      matchLabels:
        name: api-pod
  ports:
  - protocol: TCP
    port: 3306
```

Ejemplo completo de network policy para DB Pod, permitir acceso desde BACKEND(API):
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
    ports:
    - protocol: TCP
      port: 3306
```

Ejemplo completo de 'internal':
* Acceso entrada desde 'web' por 8080 con 'ingress'
* Acceso salida 'payroll' por 8080 y 'mysql' por 3306 con 'egress'
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
spec:
  podSelector:
    matchLabels:
      name: internal

  policyTypes:
  - Ingress
  - Egress
  
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: web
    ports:
    - protocol: TCP
      port: 8080
  
  egress:
  - to:
    - podSelector:
        matchLabels:
          name: payroll
    ports:
    - protocol: TCP
      port: 8080
  - to:
    - podSelector:
        matchLabels:
          name: mysql
    ports:
    - protocol: TCP
      port: 3306

```

### **2.1. Comandos**

`kubectl create -f network-policy-definition.yaml`

`kubectl get networkpolicy`

`kubectl get networkpolicy payroll-policy`

`kubectl describe networkpolicy payroll-policy`

Network policy no sale en el `get all`:

`kubectl get networkpolicy --all-namespaces`

`kubectl get networkpolicy -n app-space`

Eliminar el network policy:

`kubectl delete networkpolicy payroll-policy`

OJO: No existe un atajo para generar un networkpolicy por comando, hay que crear el yaml desde 0


