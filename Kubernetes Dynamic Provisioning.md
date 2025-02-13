### **Kubernetes Dynamic Provisioning**  

Kubernetes provides two ways to create Persistent Volumes (PVs):  
1. **Static Provisioning** – The administrator manually creates PVs before they are used.  
2. **Dynamic Provisioning** – Kubernetes **automatically creates PVs** when needed, using a **StorageClass**.  

---

### **How Dynamic Provisioning Works**  
When you create a **Persistent Volume Claim (PVC)**, you specify:  
✅ The **amount of storage** (e.g., 15Gi)  
✅ The **access mode** (e.g., ReadWriteOnce)  
✅ The **storage class** (e.g., `nfs-storage`)  

🔹 Kubernetes looks for an existing PV that matches the request.  
🔹 If no matching PV exists, **a provisioner in the storage class automatically creates one**.  
🔹 If no PV can be created, the PVC remains **unbound** until a suitable PV is available.  

### **Example of Storage Classes**  
You can check available storage classes using:

```sh
oc get storageclass
```

Example output:  
```
NAME                    PROVISIONER
nfs-storage (default)   k8s-sigs.io/nfs-subdir-external-provisioner
lvms-vg1               topolvm.io
```

- The **default storage class** (`nfs-storage`) is used if no storage class is specified.  
- If you want a **specific** storage class, you must mention it explicitly.  

---

### **How to Create a PVC with Dynamic Provisioning**  
Use the following command to create a dynamically provisioned PV and attach it to an application:

```sh
oc set volumes deployment/example-application \
  --add --name example-pv-storage \
  --type pvc --claim-class nfs-storage \
  --claim-mode rwo --claim-size 15Gi \
  --mount-path /var/lib/example-app \
  --claim-name example-pv-claim
```

### **Breaking Down the Command**  
| **Option** | **Description** |
|------------|---------------|
| `--claim-class nfs-storage` | Uses the `nfs-storage` storage class. |
| `--claim-mode rwo` | Sets **ReadWriteOnce** access mode (one pod can write at a time). |
| `--claim-size 15Gi` | Requests **15GB** of storage. |
| `--mount-path /var/lib/example-app` | Mounts the storage inside the container. |
| `--claim-name example-pv-claim` | Specifies the name of the PVC. |

⚠️ **Tip**: Always specify the **storage class** to avoid unexpected behavior, since the default storage class may change.

---

### **Persistent Volume (PV) Lifecycle**  
A **PV** follows different stages in its lifecycle:

| **State**  | **Description** |
|------------|---------------|
| **Available** | The PV is created but not yet used. |
| **Bound** | The PV is assigned to a PVC and cannot be used by another PVC. |
| **In Use** | A pod is actively using the PVC, preventing deletion. |
| **Released** | The PVC is deleted, and the PV is free for reuse. |
| **Reclaimed** | The PV is either **deleted** or requires **manual cleanup** (depending on policy). |

---

### **Reclaim Policy**  
Each **PV has a reclaim policy** that defines what happens after a PVC is deleted:

| **Policy** | **What Happens?** |
|-----------|----------------|
| **Retain** | The storage remains but must be manually cleaned up. |
| **Delete** | The storage and PV are automatically deleted. |

---

### **How to Delete a PVC**  
To delete a **Persistent Volume Claim**, use:

```sh
oc delete pvc/example-pvc-storage
```

If the **storage class** is set to **Delete**, Kubernetes will also remove the associated PV.

---

### **Key Takeaways**  
🔹 **Dynamic Provisioning** allows Kubernetes to **create storage on demand**.  
🔹 **Storage Classes** define how storage is allocated.  
🔹 **PVs follow a lifecycle**, and their **reclaim policy** determines what happens after deletion.  
🔹 Always **specify the storage class** to avoid surprises!  
