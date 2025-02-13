### **Creating a Storage Class in Kubernetes**  

A **Storage Class** in Kubernetes defines how storage is provisioned and managed. It helps developers get storage without worrying about backend details.  

---

### **Step 1: Understanding a Storage Class YAML File**  
The following is an example of a **Storage Class** using **AWS Elastic Block Store (EBS)**:  

```yaml
apiVersion: storage.k8s.io/v1  # 1. Specifies the API version
kind: StorageClass  # 2. Defines this object as a StorageClass
metadata:
  name: io1-gold-storage  # 3. The name of the storage class
  annotations:  # 4. Additional metadata
    storageclass.kubernetes.io/is-default-class: "false"
    description: "Provides RWO and RWOP Filesystem & Block volumes"
parameters:  # 5. Storage-specific settings
  type: io1
  iopsPerGB: "10"
provisioner: kubernetes.io/aws-ebs  # 6. Defines the storage backend
reclaimPolicy: Delete  # 7. Automatically deletes storage when PVC is removed
volumeBindingMode: Immediate  # 8. Creates storage immediately
allowVolumeExpansion: true  # 9. Allows increasing storage size
```

---

### **Step 2: Key Components Explained**
| **Field** | **Description** |
|-----------|---------------|
| `apiVersion` | Defines the API version (always `storage.k8s.io/v1`). |
| `kind` | Specifies that this object is a `StorageClass`. |
| `metadata.name` | Unique name for this storage class. |
| `annotations` | Extra details, e.g., whether it's the default class. |
| `parameters` | Defines specific settings (varies by storage type). |
| `provisioner` | Specifies the storage backend (e.g., AWS EBS, Azure Disk, Google PD). |
| `reclaimPolicy` | What happens when a PVC is deleted (`Retain` or `Delete`). |
| `volumeBindingMode` | Controls when storage is allocated (`Immediate` or `WaitForFirstConsumer`). |
| `allowVolumeExpansion` | If `true`, lets you increase the storage size later. |

---

### **Step 3: Creating the Storage Class**
Save the YAML file as **storage-class.yaml** and apply it to your Kubernetes cluster:  

```sh
kubectl apply -f storage-class.yaml
```

To verify it was created:  
```sh
kubectl get storageclass
```

---

### **Step 4: Using a Storage Class in a PVC**
After defining a **Storage Class**, you can request storage using a **Persistent Volume Claim (PVC)**:  

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-block-pvc
spec:
  accessModes:
    - ReadWriteOnce  # RWO: Only one node can write at a time
  volumeMode: Block  # Raw block storage
  storageClassName: io1-gold-storage  # Name of the storage class
  resources:
    requests:
      storage: 10Gi  # Request 10GB of storage
```

Create the PVC with:  
```sh
kubectl apply -f my-pvc.yaml
```

---

### **Step 5: Attaching PVC to a Deployment**
To use the PVC in a deployment:  

```sh
kubectl set volume deployment/<deployment-name> \
--add --name <my-volume-name> \
--claim-name my-block-pvc \
--mount-path /var/tmp
```

---

### **Final Thoughts**
✅ Storage Classes make it easy to manage storage in Kubernetes.  
✅ The `provisioner` defines the backend (AWS, Azure, etc.).  
✅ `reclaimPolicy` controls if storage is deleted or retained.  
✅ `volumeBindingMode` determines when storage is assigned.  
✅ PVCs request storage using a Storage Class.  


