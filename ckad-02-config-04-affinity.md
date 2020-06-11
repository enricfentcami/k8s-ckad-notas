# CONFIGURACIÓN - Asignación de pods a nodos (afinidad)

Explicación de taints y affinity: https://medium.com/@betz.mark/herding-pods-taints-tolerations-and-affinity-in-kubernetes-2279cef1f982

## **1. Taints and tolerations**
---

Se utiliza para que ciertos nodos solo puedan contener pods con ciertas características (repele pods). No significa que los pods que "encajan" en el nodo vayan a desplegarse en ese nodo obligatoriamente.
- Taint = nodo
- Toleration = pod

Obtener nodos disponibles:

`kubectl get nodes`

Obtener características de un nodo:

`kubectl describe node node01`

### **1.1. Taint (node)**

Taint indica que el nodo solo aceptará pods que en su `toleration` cumpla con la condición key=value. Y será intolerante a cualquer otro tipo de pod.

`kubectl taint nodes node-name key=value:taint-effect`

`kubectl taint nodes node1 app=blue:NoSchedule`
> Ningún pod se meterá (no schedule) en el node1 a no ser que tenga `toleration` que coincida.

Tipos de 'taint-effect', indica el comportamiento ante pods intolerantes:
- NoSchedule: No se meterá un pod si no cumple la condición
- PreferNoSchedule: Preferiblemente no se meterá un pod si no cumple la condición
- NoExecute: No se meterá un pod si no cumple la condición y los que ya existan que no la cumplen serán expulsados (evicted)

Eliminar el taint de master para que se pueda utilizar como worker:

`kubectl taint nodes master node-role.kubernetes.io/master:NoSchedule-`

Ojo: El `-` de `NoSchedule` indica que se va a eliminar ese taint.

### **1.2. Toleration (pod)**

Se crea la condición para cumplir un taint, pero no significa que obligatoriamente un pod vaya a dicho nodo

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-color
spec:
  containers:
    - name: webapp-color
      image: webapp-color
  tolerations:
    - key: "app"
      operator: "Equal"
      value: "blue"
      effect: NoSchedule
```

## **2. Etiquetar nodos**
---

Los nodos se etiquetan para poder utilizar estos mecanismos de selección o afinidad:

`kubectl label nodes <node-name> <label-key>=<label-value>`

`kubectl label nodes node-1 size=Large`


## **3. Node selectors**
---

Se utilizan etiquetas como '`size: Large`' para indicar que un Pod se despliegue en un nodo concreto.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-color
spec:
  containers:
    - name: webapp-color
      image: webapp-color
  nodeSelector:
    size: Large
```

Pero ojo, que no se permiten condiciones como 'size = large OR size = medium' o 'size != small'

## **4. Node affinity (pod)**
---

Mejora el funcionamiento de los node selectors permitiendo operaciones más complejas.

https://kubernetes.io/docs/concepts/configuration/assign-pod-node/

Ejemplo, desplegar un Pod en un nodo 'large' o 'medium':
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-color
spec:
  containers:
    - name: webapp-color
      image: webapp-color
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: In
                values:
                  - Large
                  - Medium
```

Ejemplo, desplegar un Pod en un nodo diferente a 'small':
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-color
spec:
  containers:
    - name: webapp-color
      image: webapp-color
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: NotIn
                values:
                  - Small
```

El operador `operator: Exists` no requiere valores, solo comprueba que el nodo tenga la label que indica el pod en `key`.

### **4.1. Tipos de afinity**

Disponibles actualmente:
- `requitedDuringSchedulingIgnoredDuringExecution`
- `preferredDuringSchedulingIgnoredDuringExecution`

Indican si el Pod puede desplegarse o no en un nodo. Puede ser requerido en el momento de desplegar o preferible que se despliegue en el nodo indicado. Siempre se ignora este valor si el Pod ya está ejecutándose en el nodo que coge el nuevo valor (etiqueta) y dejan de encajar.

## **5. Taints & Tolerations VS Node Affinity**

Con "Taints & Tolerations" se blinda el nodo para que solo se puedan desplegar Pods que complen la condición de taint en sus tolerances, pero NO asegura que los Pods que las cumplen se vayan a desplegar en dicho nodo.

Con "Node Affinity" se obliga a los Pods a desplegarse en el nodo con el que son afines, pero NO asegura que solo se puedan desplegar esos nodos afines, pueden entrar otros nodos que no tienen ese tipo de configuración.

SOLUCIÓN: Combinar ambas estrategias. Protegemos los nodos con "taints" y los Pods con "affinity", y de este modo podemos tener nodos dedicados exclusivamente a determinadas aplicaciones.