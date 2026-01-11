# Secrets

## What is a Secret in Kubernetes?
- A `Secret` is a Kubernetes object used to store sensitive data in your cluster. 
- These include:
    - `Database passwords`
    - `API tokens`
    - `SSH keys`
    - `TLS certificates`
- `Secrets` help keep this information securely managed and accessible, without embedding it directly in your application code or container images.

## Why Use Secrets (Instead of ConfigMaps or Hardcoding)?
- Secrets offer:
    - Base64 encoding for obfuscation
    - Optional encryption at rest (etcd encryption)
    - Controlled access via RBAC (Role-Based Access Control)
    - Mounted with file permission restrictions
    - Integration with Kubernetes ServiceAccounts and imagePullSecrets
- Unlike `ConfigMaps`, `Secrets` are not meant to be human-readable and are intended to store confidential data.

## Types of Kubernetes Secrets
| Type                                    | Description                                              |
| --------------------------------------- | -------------------------------------------------------- |
| **Opaque**                              | Default; used for arbitrary user-defined key-value pairs |
| **kubernetes.io/basic-auth**            | For storing username/password credentials                |
| **kubernetes.io/ssh-auth**              | Stores SSH credentials                                   |
| **kubernetes.io/tls**                   | TLS cert/key pair (used with Ingress or HTTPS services)  |
| **kubernetes.io/service-account-token** | Automatically created and managed by Kubernetes for pods |

## YAML Manifest
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

## Using Secrets in Pods
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

## Examples
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
