# CONFIGURACIÓN - Recursos de sistema

## **1. Recursos de sistema requeridos (Resource Request)**
---

Para CKAD solo se necesita saber cómo especificar los recursos

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-color
spec:
  containers:
    - name: webapp-color
      image: webapp-color
  resources:
    requests:
      memory: "1Gi"
      cpu: 1
    limits:
      memory: "2Gi"
      cpu: 2
```
¿Qué pasa si se excede el límite?
- La CPU que consuma el Pod no puede exceder los límites.
- La memoria sí puede exceder, pero si es constante el Pod será finalizado.

### **Valores de CPU**

- m = Milli (Millicore)
- 1 CPU = 1000m
  - 1 AWS vCPU
  - 1 GCP Core
  - 1 Azure Core
  - 1 Hyperthread

### **Valores de Memoria**

- Gi = Gibibyte
- G = Gigabyte
- Mi = Mebibyte
- M = Megabyte
- Ki = Kibibyte = 1024 bytes
- K = Kilobyte = 1000 bytes
