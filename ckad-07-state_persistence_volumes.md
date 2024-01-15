# STATE PERSISTENCE - Volumes & Persistent Volumes

El almacenamiento dentro de un Pod es volátil o transient. Se utilizan volúmenes para persistir datos fuera del Pod.

## **1. Volumes & mount**

Montar un volumen dentro del Pod con almacenamiento en el host (solo util en single-node):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
    - name: alpine
      image: alpine
      command: ["/bin/sh", "-c"]
      args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]

      volumeMounts:
      - mountPath: /opt
        name: data-volume

    volumes:
    - name: data-volume
      hostPath:
        path: /data
        type: Directory
```

### **1.1. Volume Types**

Kubernetes dispone de muchos tipos diferentes como: NFS, ceph, AWS EBS, ...

https://kubernetes.io/docs/concepts/storage

Ejemplo con AWS EBS (Elastic Block Store):
```yaml
volumes:
- name: data-volume
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```

## **2. Persistent Volumes**

Pool de volúmenes de almacenamiento para todo el clúster que lo configura un administrador.

Los usuarios pueden seleccionar el almacenamiento de este pool utilizando Persistent Volume Claims (PVC).

Ejemplo Persitent Volume:
```yaml
apiVersion: v1
kind: PersitentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
#  hostPath:              # Path del nodo. No recomendado en producción
#    path: /tmp/data
  awsElasticBlockStore:   # Es mejor utilizar un servicio externo
    volumeID: <volume-id>
    fsType: ext4

```

### **2.1. Comandos**

`kubectl create -f pv-definition.yaml`

`kubectl get persitentvolume`

Access modes:
* ReadOnlyMany
* ReadWriteOnce
* ReadWriteMany

### **2.2. Reclaim Policy**

https://kubernetes.io/docs/concepts/storage/persistent-volumes/#reclaim-policy

Al eliminar el PVC el PV puede quedarse huérfano. Por defecto los PV tienen la política `persistentVolumeReclaimPolicy` a `Retain` que espera que el PV se vuelva a reclamar manualmente por otro PVC.

Otras opciones son `Recycle` y `Delete`. Se describen en la documentación

## **3. Persistent Volume Claims**

El admin crea los Persistent Volumes y los usuarios los Persitent Volume Claims para utilizar los PV. Además están en namespaces diferentes.

Un PVC solo puede estar enlazado a un PV y viceversa (one-to-one), y pueden existir varios PV que encajen con la configuración del PVC. Para eso se utilizan labels.

Label en Persitent Volume:
```yaml
labels:
  name: my-pv
```

Selector de label en Persitent Volume Claim:
```yaml
selector:
  matchLabels:
    name: my-pv
```

OJO: 
* Si el PV tiene más capacidad que el PVC, el espacio restante no podrá se utilizado por otro PVC.
* Si el PVC no tiene PV disponible se quedará en estado "Pending" hasta que exista un PV que le encaje.

Ejemplo de Persitent Volume Claim:
```yaml
apiVersion: v1
kind: PersitentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

Al crearlo, encajará con el PV creado en el punto 1 y hará el binding por los criterios:
* Encaja el tamaño
* Encajan Access Modes

### **3.1. Comandos**

`kubectl create -f pvv-definition.yaml`

`kubectl get persitentvolumeclaim`

`kubectl delete persitentvolumeclaim myclaim`

### **3.2. Utilización de PVCs en Pods**

Once you create a PVC use it in a POD definition file by specifying the PVC Claim name under `persistentVolumeClaim` section in the volumes section like this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```

The same is true for ReplicaSets or Deployments. Add this to the pod template section of a Deployment on ReplicaSet.

Reference URL: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes



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


