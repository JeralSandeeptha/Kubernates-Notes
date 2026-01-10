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
- [Config Maps](#config-maps)
- [Secrets](#secrets)
- [Demonset](#demonset)
- [Job](#job)
- [Cron Job](#cron-job)
- [Volumes](#volumes)
- [Kubernates Workflow](#kubernates-workflow)
- [Multi Container Pods](#multi-container-pods)

<br />
<br />

### Config Maps
- A ConfigMap is a Kubernetes object used to store non-sensitive key-value pairs. It allows you to decouple configuration from application code
- Use ConfigMaps to store configuration data like environment variables, command-line arguments, port numbers, URLs, and feature flags
- `If we use env inside of pods and deployments we can't reuse them. But with the ConfigMaps we can reuse them`
- We create kubernates object with all the envs

#### Why Use ConfigMaps?
- Keep your app configuration separate from container images
- Easily change app behavior without rebuilding the image
- Make config reusable and manageable across multiple pods
- Configurable at runtime via environment variables, command-line args, or mounted files

#### Basic Example
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_MODE: production
  APP_PORT: "8080"
  LOG_LEVEL: debug
```

#### Ways to Use a ConfigMap
- As Environment Variables
```yaml
envFrom:
- configMapRef:
    name: app-config

env:
- name: MODE
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: APP_MODE
```

- As Mounted Files (Volumes)
```yaml
volumes:
- name: config-volume
  configMap:
    name: app-config

volumeMounts:
- name: config-volume
  mountPath: /etc/config
```

#### Examples
- Create a Configmap
```cmd
kubectl create configmap my-config --from-literal=APP_MODE=dev --from-literal=PORT=3000

kubectl create configmap my-config --from-file=app.properties
```

- Get Configmaps
```cmd
kubectl get configmaps

kubectl get configmap app-config -o yaml
```

- Describe Configmaps
```cmd
kubectl describe configmap app-config
```

- Changes won’t reflect in existing Pods automatically
- You must restart or recreate the Pods to pick up new values
- Don't store passwords or secrets — use Secrets instead
- ConfigMaps are not encrypted and are stored in plaintext

#### Example
- Create a ConfigMap
```cmd
kubectl create cm sample-cm --from-literal=firstName=jeral --from-literal=lastName=sandeeptha
```

- Create a manifestfile of ConfigMap
```cmd
kubectl create cm sample-cm --from-literal=firstName=jeral --from-literal=lastName=sandeeptha --dry-run=client -o yaml > configmap.yaml
```

- Get ConfigMaps
```cmd
kubectl get cm
```

- Describe ConfigMaps
```cmd
kubectl describe cm
```

<br />
<br />

### Secrets

#### What is a Secret in Kubernetes?
- A `Secret` is a Kubernetes object used to store sensitive data in your cluster. 
- These include:
    - `Database passwords`
    - `API tokens`
    - `SSH keys`
    - `TLS certificates`
- `Secrets` help keep this information securely managed and accessible, without embedding it directly in your application code or container images.

#### Why Use Secrets (Instead of ConfigMaps or Hardcoding)?
- Secrets offer:
    - Base64 encoding for obfuscation
    - Optional encryption at rest (etcd encryption)
    - Controlled access via RBAC (Role-Based Access Control)
    - Mounted with file permission restrictions
    - Integration with Kubernetes ServiceAccounts and imagePullSecrets
- Unlike `ConfigMaps`, `Secrets` are not meant to be human-readable and are intended to store confidential data.

#### Types of Kubernetes Secrets
| Type                                    | Description                                              |
| --------------------------------------- | -------------------------------------------------------- |
| **Opaque**                              | Default; used for arbitrary user-defined key-value pairs |
| **kubernetes.io/basic-auth**            | For storing username/password credentials                |
| **kubernetes.io/ssh-auth**              | Stores SSH credentials                                   |
| **kubernetes.io/tls**                   | TLS cert/key pair (used with Ingress or HTTPS services)  |
| **kubernetes.io/service-account-token** | Automatically created and managed by Kubernetes for pods |

#### YAML Manifest
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4=         # base64 of "admin"
  password: UzNjcjN0IQ==     # base64 of "S3cr3t!"
```

#### Using Secrets in Pods
```yaml
env:
- name: DB_USER
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: username
- name: DB_PASS
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: password
```

#### Examples
- Create a Secrets
```bash
kubectl create secret generic config-secret \
  --from-file=./config.txt
```
 
- Apply from a file
```bash
kubectl create secret generic config-secret \
  --from-file=./config.txt
```

<br />
<br />

### Demonset
- A `DaemonSet` ensures that a copy of a specific Pod runs on every node in a Kubernetes cluster (or on selected nodes using label selectors).

- Runs one Pod per node.
- Automatically runs new Pods when:
    - A node is added.
    - A node is removed → its Pod is deleted.
- Can target specific nodes using:
    - Node labels
    - Tolerations and taints
    - Node selectors or affinity rules

- It’s used to automate infrastructure-level tasks like:
    - Logging agents
    - Monitoring agents
    - Node-level storage
    - Network plugins
    - Security tools

| Use Case                        | Example Tool             |
| ------------------------------- | ------------------------ |
| Log collection                  | Fluentd, Logstash        |
| Metrics monitoring              | Prometheus Node Exporter |
| System security scanning        | Falco, Aqua Security     |
| Container runtime monitoring    | cAdvisor                 |
| Node local storage provisioning | Local volume provisioner |

<br />
<br />

### Job
- A `Job` in Kubernetes is used to run a one-time task to completion.
- It ensures that a specified number of Pods successfully terminate, and then the Job is considered complete.

| Feature             | Description                          |
| ------------------- | ------------------------------------ |
| One-shot tasks      | Run once, then exit on success       |
| Pod retry mechanism | Can retry failed Pods (with limits)  |
| Parallelism support | Can run multiple Pods simultaneously |
| Completion tracking | Stops when all Pods complete         |

<br />
<br />

### Cron Job
- A `CronJob` allows you to run Jobs on a time-based schedule (just like UNIX cron)
- It creates a Job at scheduled intervals, and each Job runs independently

| Feature             | Description                             |
| ------------------- | --------------------------------------- |
| Time-based trigger  | Uses cron format (e.g., `*/5 * * * *`)  |
| Job template        | Uses the same spec as a normal Job      |
| Independent runs    | Each execution is its own Job           |
| Missed run handling | Can skip or catch up on missed runs     |
| History tracking    | Keeps logs of completed and failed jobs |

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

<br />
<br />

### Kubernates Workflow

- Get Cluster Status
```bash
minikube status
```

- Get clusters
```bash
kubectl config get-clusters
```

- Get current cluster
```bash
kubectl config current-context
```

- Get cluster information
```bash
kubectl cluster-info
```

<br />

- Get Nodes
```
kubectl get nodes

kubectl get nodes -o wide
```

- Describe node
```bash
kubectl describe node minikube
```

- Switch to another Node
```bash
kubectl 
```

- Log into a specific Node
```bash
minikube ssh -n=<node_name> 
```

<br />

- Get Kubernates default resources
```bash
kubectl get all
```

- Delete Kubernates default resources
```bash
kubectl delete all --all
```

<br />

- Create a Pod yaml file
```bash
kubectl run nginx-pod --image=nginx:latest --dry-run=client -o yaml > nginx-pod.yaml
```

- Get a Pods
```bash
kubectl get pods
```

- Get inside of the Pod
```bash
kubectl exec -it nginx-pod -- bash
```

<br />
<br />

### Multi Container Pods

![Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1753529854/ce36d536-d392-4d7c-a98a-0ae915209318.png)

- In Kubernetes, a Pod is the smallest deployable unit and can contain one or more containers. While most Pods run a single container, multi-container Pods allow you to group multiple tightly-coupled containers that:
    - Share the same network namespace (IP, port space)
    - Share storage volumes
    - Are scheduled together on the same node
    - Can communicate using localhost

#### When to Use Multi-Container Pods?

- Init Container Pattern
    - Main container depends on this container
- Sidecar / Helper Pattern
    - An auxiliary container that enhances or adds functionality to the main container
    - Eg: a logging or metrics collector that reads logs from a shared volume.

<br />
<br />
