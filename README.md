# Kubernates Notes

<br />

### Table of Contents
- [Introduction](./files/introduction.md)
- [Architecture](./files/architecture.md)
- [Objects](./files/objects.md)
- [Pod](./files/pod.md)
- [Deployment](./files/deployment.md)
- [Service](./files/services.md)
- [Environment Variables](./files/ev.yaml)
- [Config Maps](./files/configmaps.md)
- [Secrets](./files/secrets.md)
- [Demonset](./files/demonsets.md)
- [Job](./files/job.md)
- [Cron Job](./files/cronjob.md)
- [Volumes](#volumes)
- [Multi Container Pods](./files/multi-container-pods.md)
- [Kubernates Workflow](./files/workfkow.md)

<br />
<br />

### Volumes
- `Volume` in Kubernetes is a directory accessible to containers in a pod. It stores data and lives as long as the pod does (or longer, if persistent)
![Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1754435234/a173c720-b8dc-479f-8380-216c17a5c5b2.png)

#### Common Use Cases
- Persist logs or database data
- Share files between containers in a pod
- Mount cloud/distributed storage

####  Main Types of Volumes

| Type                          | Description                                                           |
| ----------------------------- | --------------------------------------------------------------------- |
| `emptyDir`                    | Temporary storage; erased when pod is deleted                         |
| `hostPath`                    | Mounts a directory from the host node                                 |
| `configMap`                   | Mounts config files into a pod                                        |
| `secret`                      | Mounts sensitive data like passwords/tokens                           |
| `persistentVolumeClaim (PVC)` | Connects to a PersistentVolume (PV); used for long-term storage       |
| `nfs`                         | Mounts a network file system                                          |
| `awsElasticBlockStore`        | AWS EBS volume                                                        |
| `gcePersistentDisk`           | Google Cloud persistent disk                                          |
| `azureDisk`                   | Azure managed disk                                                    |
| `csi`                         | Generic volume interface for external storage plugins                 |
| `projected`                   | Combines multiple sources (e.g., secrets, configMaps) into one volume |

#### PV and PVC
![Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1754436667/77e3e0fa-cada-45c0-8432-c2cfa7c88ddd.png)

- PV is an actual storage. It can be a storage device or remote storage and created by K8S admin
- PVC is the storage request from actual storage which is PV. If we have to use storage then we have to claim that storage using PVC

#### Access Modes
- Access modes define how a volume can be mounted by the pods
    - ReadWriteOnce (RWO)
    - ReadOnlyMany (ROX)
    - ReadWriteMany (RWX)
- PV and PVC access modes should be the same of both of them
```yaml
accessModes:
  - ReadWriteOnce
```

#### Static Vs Dynamic Provisioning
- In `Static Provisioning`, it is a manual process. An `administrator` pre-creates PV object and configures them with actual storage. Users then create `PVC` objects to bind to those PVs
- In that case,
    - Admin creates a `PV` manually
    - User creates a `PVC`
    - Kubernetes matches the `PVC` to an available `PV`

<br />

![Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1754438139/9257de51-f752-43e7-aeae-1583254c4038.png)
- In `Dynamic Provisioning`, automatic provisioning of storage volumes when a PVC is created. Requires a StorageClass which tells Kubernetes how to provision the storage. Kubernetes creates the underlying volume on demand
- In that case,
    - User creates a PVC with a `StorageClass`
    - Kubernetes uses the `StorageClass` to dynamically create a new PV
    - `PVC` is bound to the newly created PV automatically

#### Reclaim Policy
- It defines what happens to PV when its associated PVC is deleted?
- 
| Policy  | Keeps Data?   | Deletes Storage? | Manual Cleanup Needed? |
| ------- | ------------- | ---------------- | ---------------------- |
| Retain  | ✅ Yes         | ❌ No             | ✅ Yes                  |
| Delete  | ❌ No          | ✅ Yes            | ❌ No                   |
| Recycle | 🚫 Deprecated | 🚫 Deprecated    | 🚫 Deprecated          |
