# STATE PERSISTENCE - Volumes & Persistent Volumes

Storage within a Pod is volatile or transient. Volumes are used to persist data outside of the Pod.

On-disk files in a container are ephemeral, which presents some problems for non-trivial applications when running in containers. One problem occurs when a container crashes or is stopped. Container state is not saved so all of the files that were created or modified during the lifetime of the container are lost. During a crash, kubelet restarts the container with a clean state. Another problem occurs when multiple containers are running in a Pod and need to share files. It can be challenging to setup and access a shared filesystem across all of the containers. The Kubernetes volume abstraction solves both of these problems. Familiarity with Pods is suggested.

## **1. Volumes & mount**

Mount a volume inside the Pod with storage on the host (only useful on single-node, but not recommended):

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

NFS example (Elastic Block Store), removed:
```yaml
volumes:
- name: data-volume
  nfs:
    server: my-nfs-server.example.com
    path: /my-nfs-volume
    readOnly: true
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
  nfs:
    server: my-nfs-server.example.com
    path: /my-nfs-volume
```

### **2.1. Commands**

`kubectl create -f pv-definition.yaml`

`kubectl get persitentvolume`

Access modes:
* ReadWriteOnce: the volume can be mounted as read-write by a single node. ReadWriteOnce access mode still can allow multiple pods to access the volume when the pods are running on the same node. For single pod access, please see ReadWriteOncePod.
* ReadOnlyMany: the volume can be mounted as read-only by many nodes.
* ReadWriteMany: the volume can be mounted as read-write by many nodes.
* ReadWriteOncePod: the volume can be mounted as read-write by a single Pod. Use ReadWriteOncePod access mode if you want to ensure that only one pod across the whole cluster can read that PVC or write to it.

### **2.2. Reclaim Policy**

https://kubernetes.io/docs/concepts/storage/persistent-volumes/#reclaim-policy

By eliminating PVC the PV can become orphaned. By default PVs have the policy `persistentVolumeReclaimPolicy` to `Retain` which expects the PV to be manually reclaimed by another PVC.

Other options are `Recycle` and `Delete`. They are described in the documentation.

For Kubernetes 1.29, only nfs and hostPath volume types support recycling.

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

### **3.2. Using PVCs in Pods**

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

### **3.3. Example of hostPath persistent volume in Pods**

Ref: https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
```

## **4. Ephemeral volumes**

Some application need additional storage but don't care whether that data is stored persistently across restarts. For example, caching services are often limited by memory size and can move infrequently used data into storage that is slower than memory with little impact on overall performance.

Other applications expect some read-only input data to be present in files, like configuration data or secret keys.

Ephemeral volumes are designed for these use cases. Because volumes follow the Pod's lifetime and get created and deleted along with the Pod, Pods can be stopped and restarted without being limited to where some persistent volume is available.

Ephemeral volumes are specified inline in the Pod spec, which simplifies application deployment and management.

Ref: https://kubernetes.io/docs/concepts/storage/ephemeral-volumes/

