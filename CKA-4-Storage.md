# 4. Storage 10% 

Domain	| Weight
------- | -------------
**4. Storage** | 10%  
[4.1. Understand storage classes, persistent volumes](CKA-4-Storage.md#41-understand-storage-classes-persistent-volumes) |  
[4.2. Understand volume mode, access modes and reclaim policies for volumes](CKA-4-Storage.md#42-understand-volume-mode-access-modes-and-reclaim-policies-for-volumes) |  
[4.3. Understand persistent volume claims primitive](CKA-4-Storage.md#43-understand-persistent-volume-claims-primitive) |  
[4.4 Know how to configure applications with persistent storage](CKA-4-Storage.md#44-know-how-to-configure-applications-with-persistent-storage) |   



## 4.1. Understand storage classes, persistent volumes  

### StorageClass object  
https://kubernetes.io/docs/concepts/storage/storage-classes/  

Defining your class of Storage
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: my-localdisk
provisioner: kubernetes.io/no-provisioner # <--- no | AWS EBS | AzureFile | CephFS | FS | Local …
allowVolumeExpansion: true                # <---- a PVC can only expand if this is set to true 
```
<br/>  
  
### Volumes  
https://kubernetes.io/docs/concepts/storage/volumes/  
https://kubernetes.io/docs/concepts/storage/persistent-volumes/  

Remember that container File Sytems is ephemeral !  
Volumes allow Persistant Storage for containers within a single Pod. 
Its lifecycle is coupled to a Pod.
It enables safe container restarts and sharing data between containers within a Pod.

Volume type:
* emptyDir = dynamically created, useful to share data between containers (ephemeral)
* hostPath = directory on local node
* NFS
* Cloud Storage
* ConfigMaps & Secrets
* Simple Directory on K8s node
<br/>

```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: host-volume-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ['sh', '-c', 'while true; do echo "Success!" >> /output/success.txt; sleep 5; done']
    volumeMounts:
    - name: my-volume
      mountPath: /output
  volumes:
  - name: my-volume
    hostPath:
      path: /home/cloud_user/
      type: Directory

---
apiVersion: v1
kind: Pod
metadata:
  name: shared-volume-pod
spec:
  containers:
  - name: busybox1
    image: busybox
    command: ['sh', '-c', 'while true; do echo "Success!" >> /output/success.txt; sleep 5; done']
    volumeMounts: 
    - name: my-volume
      mountPath: /output
  - name: busybox2
    image: busybox
    command: ['sh', '-c', 'while true; cat /input/success.txt; sleep 5; done']
    volumeMounts: 
    - name: my-volume
      mountPath: /input
  volumes: 
  - name: my-volume
    emptyDir: {}
```
<br/>

### Persistent Volumes  
https://kubernetes.io/docs/concepts/storage/persistent-volumes/  
* K8s Object (not simply referred to in a Pod Object)
* More advanced form of Volume. 
* Allow to treat Storage as an abstract resource and consume it using Pods.
* Its lifecycle is independent of a single Pod, so it enables safe Pod restarts and sharing data between Pods.

PersistentVolume (PV) object  
https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistent-volumes

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  storageClassName: my-localdisk
  persistentVolumeReclaimPolicy: Recycle 
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /var/output
```  
<br/>

### Types of Persistent Volumes

Supported PV plugins | Description
-------------------- | -------------
awsElasticBlockStore | AWS Elastic Block Store (EBS)
azureDisk | Azure Disk
azureFile | Azure File
cephfs | CephFS volume
cinder | Cinder (OpenStack block storage) (deprecated)
csi | Container Storage Interface (CSI)
fc | Fibre Channel (FC) storage
flexVolume | FlexVolume
flocker | Flocker storage
gcePersistentDisk | GCE Persistent Disk
glusterfs | Glusterfs volume
hostPath | HostPath volume (for single node testing only; WILL NOT WORK in a multi-node cluster; consider using local volume instead)
iscsi | iSCSI (SCSI over IP) storage
local | local storage devices mounted on nodes.
nfs | Network File System (NFS) storage
photonPersistentDisk | Photon controller persistent disk. (This volume type no longer works since the removal of the corresponding cloud provider.)
portworxVolume | Portworx volume
quobyte | Quobyte volume
rbd - Rados Block Device (RBD) volume
scaleIO | ScaleIO volume (deprecated)
storageos | StorageOS volume
vsphereVolume | vSphere VMDK volume

<br/><br/>


## 4.2. Understand volume mode, access modes and reclaim policies for volumes  

### volume mode  
There are two ways PVs may be provisioned: statically or dynamically.

<br/>

### Access mode   
https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes

The access modes are:
* ReadWriteOnce -- the volume can be mounted as read-write by a single node
* ReadOnlyMany -- the volume can be mounted read-only by many nodes
* ReadWriteMany -- the volume can be mounted as read-write by many nodes

```yaml
apiVersion: v1
kind: PersistentVolume
spec:
  accessModes:
    - ReadWriteOnce
  ...
```

In the CLI, the access modes are abbreviated to:

RWO - ReadWriteOnce
ROX - ReadOnlyMany
RWX - ReadWriteMany

<br/>

### Reclaim Policies   
https://kubernetes.io/docs/concepts/storage/persistent-volumes/#reclaiming

Reclaim Policies:  
* Recycle = automatically delete all data in underlying storage resource  
* Retain = keep all data (manual cleanup required by admin)  
* Delete = (only for cloud storage resources) automatically delete all data  
  
```yaml
apiVersion: v1
kind: PersistentVolume
spec:
  ...
  persistentVolumeReclaimPolicy: Recycle | Retain | Delete
  ...
```
  
<br/><br/>  
  
## 4.3. Understand persistent volume claims primitive  
https://kubernetes.io/docs/concepts/storage/persistent-volumes/#reserving-a-persistentvolume  

`PersistentVolumeClaim` object is a user’s request with attributes. It is bound to a `PersistentVolume` based on a matching `StorageClass`.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  storageClassName: my-localdisk
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
```

“Expand” a PVC = edit `spec.resource.request.storage` attribute of existing PVC. (must support `allowVolumeExpansion=true` in the `StorageClass` definition)

<br/><br/>

## 4.4 Know how to configure applications with persistent storage   
https://kubernetes.io/docs/concepts/storage/persistent-volumes/

Using PVC in a Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pv-pod
spec:
  containers:
    - name: busybox
      image: busybox
      command: ['sh', '-c', 'echo "Success!" > /output/success.txt']
      volumeMounts:
      - name: pv-vol
        mountPath: "/output"
  volumes:
    - name: pv-vol
      persistentVolumeClaim:
        claimName: my-pvc
```
