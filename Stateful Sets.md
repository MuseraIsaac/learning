### **Managing Non-Shared Storage with Stateful Sets**  

If you’re new to **Kubernetes**, understanding how to manage storage for applications can be confusing. Some applications (like databases) need **persistent storage**, meaning they need to **remember data even if they restart**. This is where **StatefulSets** come in.  

---

## **1. Why Do We Need StatefulSets?**  
Some applications, like **MySQL, PostgreSQL, or Cassandra**, rely on **consistent storage**. If a pod crashes and restarts, it must retain its **original data** and **network identity**.   

📌 **Example Problem:**  
Imagine you’re running a **database cluster**. You want each database instance to keep its data even if the pod is recreated. If we use a **Deployment (stateless)**, Kubernetes may start a new pod with **no memory of past data**.  

✅ **Solution:** Use **StatefulSets**, which ensure each pod has:  
- A **stable name** (e.g., `dbserver-0`, `dbserver-1`).  
- A **dedicated storage volume** that persists even after deletion.  
- Ordered, predictable startup and scaling.  

---

## **2. Understanding Non-Shared Storage**  
There are **two main types of storage** in Kubernetes:  

### **A. Shared Storage (Network Storage - NAS)**  
Used when **multiple pods need to access the same data** (e.g., a file server). Common protocols:  
- **NFS (Network File System):** Allows multiple pods to access shared files.  
- **SMB (Server Message Block):** Used for Windows file sharing.  

### **B. Non-Shared Storage (Block Storage - SAN & iSCSI)**  
Used when **each pod needs its own private storage** (e.g., databases).  
- **SAN (Storage Area Network):** Provides fast, raw disk storage.  
- **iSCSI (Internet Small Computer Systems Interface):** Used for direct disk access over a network.  

💡 **StatefulSets use block storage (SAN/iSCSI) to give each pod a separate, persistent disk.**  

---

## **3. How StatefulSets Work in Kubernetes**  

📌 **Key Features of StatefulSets:**  
- **Each pod gets a unique name and persistent volume** (e.g., `dbserver-0`, `dbserver-1`).  
- **Pods restart in order** (if `dbserver-0` fails, Kubernetes replaces it with a pod of the same name).  
- **Scaling is sequential** (it creates `dbserver-2` only after `dbserver-1` is running).  

📌 **Key Difference From Deployments:**  
- **Deployments** use a **shared volume** across all replicas.  
- **StatefulSets** give **each pod its own separate volume** (no data sharing).  

---

## **4. Example: Deploying a StatefulSet with MySQL**  

Here’s a **Kubernetes YAML file** for deploying a **3-replica MySQL cluster** using a **StatefulSet**:  

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-cluster
spec:
  serviceName: "mysql-service"
  replicas: 3  # Three database instances
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:latest
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "my-secret-password"
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: "standard"
      resources:
        requests:
          storage: 10Gi
```

### **Explanation of the YAML:**
✅ **`replicas: 3`** → Creates three MySQL pods (`mysql-cluster-0`, `mysql-cluster-1`, `mysql-cluster-2`).  
✅ **`volumeClaimTemplates`** → Ensures **each pod gets a separate 10GB storage volume**.  
✅ **`MYSQL_ROOT_PASSWORD`** → Sets the database password.  
✅ **`storageClassName: "standard"`** → Uses block storage for persistent data.  

---

## **5. Deploying and Managing the StatefulSet**  

### **A. Deploy the StatefulSet**
Run the following command to deploy the StatefulSet:  
```sh
kubectl apply -f mysql-statefulset.yml
```

### **B. Verify StatefulSet Pods**
Check if the MySQL pods are running:  
```sh
kubectl get pods
```
🔹 Expected Output:  
```
NAME                READY   STATUS    RESTARTS   AGE
mysql-cluster-0     1/1     Running   0          10s
mysql-cluster-1     1/1     Running   0          8s
mysql-cluster-2     1/1     Running   0          6s
```

### **C. Verify Persistent Volumes**
```sh
kubectl get pvc
```
🔹 Expected Output:  
```
NAME                    STATUS   VOLUME              CAPACITY   ACCESS MODES   STORAGECLASS
mysql-storage-0         Bound    pvc-xxxx           10Gi       RWO            standard
mysql-storage-1         Bound    pvc-yyyy           10Gi       RWO            standard
mysql-storage-2         Bound    pvc-zzzz           10Gi       RWO            standard
```
🔹 Notice that **each pod has its own persistent volume**.

### **D. Scale the StatefulSet**
To **increase the number of MySQL pods**, run:  
```sh
kubectl scale statefulset/mysql-cluster --replicas=5
```
🔹 This adds `mysql-cluster-3` and `mysql-cluster-4` while keeping the previous pods.  

### **E. Delete the StatefulSet**
```sh
kubectl delete statefulset mysql-cluster
```
🚨 **Important:** This deletes the pods **but keeps the persistent volumes**. If you want to delete the storage too, run:  
```sh
kubectl delete pvc --all
```

---

## **6. Summary**  

| Feature          | StatefulSet (For Databases) | Deployment (For APIs, Web Apps) |
|-----------------|------------------|------------------|
| **Data Storage** | Each pod has its own persistent volume | Pods share a single volume |
| **Pod Identity** | Each pod has a fixed name (`pod-0`, `pod-1`) | Pods are randomly named (`pod-xyz`) |
| **Scaling** | Creates pods **one by one** | Creates pods in **parallel** |
| **Restart Behavior** | Restores the same pod with its data | New pod is created without old data |
| **Use Cases** | Databases, Kafka, Zookeeper | Web apps, APIs, microservices |

---

## **7. When Should You Use StatefulSets?**  
✅ Use a **StatefulSet** when:  
✔️ Running databases (**MySQL, PostgreSQL, MongoDB, Cassandra**)  
✔️ Running message queues (**Kafka, RabbitMQ**)  
✔️ Running storage systems (**Ceph, GlusterFS**)  
✔️ Need persistent, unique storage per pod  


