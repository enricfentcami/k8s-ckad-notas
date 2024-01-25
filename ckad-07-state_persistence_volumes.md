# STATE PERSISTENCE - Volumes & Persistent Volumes

Storage within a Pod is volatile or transient. Volumes are used to persist data outside of the Pod.

## **1. Volumes & mount**

Mount a volume inside the Pod with storage on the host (only useful on single-node):

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

Kubernetes has many different types such as: NFS, ceph, AWS EBS, ...

https://kubernetes.io/docs/concepts/storage

AWS EBS example (Elastic Block Store):
```yaml
volumes:
- name: data-volume
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```

## **2. Persistent Volumes**

Pool of storage volumes for the entire cluster that is configured by an administrator.

Users can select storage from this pool using Persistent Volume Claims (PVC).

Persitent Volume example:
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
#  hostPath:              # Path of the node. Not recommended in production
#    path: /tmp/data
  awsElasticBlockStore:   # It is better to use an external service
    volumeID: <volume-id>
    fsType: ext4
```

### **2.1. Commands**

`kubectl create -f pv-definition.yaml`

`kubectl get persitentvolume`

Access modes:
* ReadOnlyMany
* ReadWriteOnce
* ReadWriteMany

### **2.2. Reclaim Policy**

https://kubernetes.io/docs/concepts/storage/persistent-volumes/#reclaim-policy

By eliminating PVC the PV can become orphaned. By default PVs have the policy `persistentVolumeReclaimPolicy` to `Retain` which expects the PV to be manually reclaimed by another PVC.

Other options are `Recycle` and `Delete`. They are described in the documentation.

## **3. Persistent Volume Claims**

The admin creates the Persistent Volumes and the users create the Persitent Volume Claims to use the PVs. They are also in different namespaces.

A PVC can only be linked to one PV and vice versa (one-to-one), and there can be several PVs that match the PVC configuration. For that, labels are used.

Label in Persistent Volume:
```yaml
labels:
  name: my-pv
```

Label selector in Persistent Volume Claim:
```yaml
selector:
  matchLabels:
    name: my-pv
```

NOTE:
* If the PV has more capacity than the PVC, the remaining space cannot be used by another PVC.
* If the PVC does not have a PV available, it will remain in the "Pending" state until there is a PV that fits it.
* 
Persitent Volume Claim example:
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

When created, it will match the PV created in point 1 and will bind according to the criteria:
* Fits size
* Fit Access Modes
* 
### **3.1. Comamnds**

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
