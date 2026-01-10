# Config Maps
- A ConfigMap is a Kubernetes object used to store non-sensitive key-value pairs. It allows you to decouple configuration from application code
- Use ConfigMaps to store configuration data like environment variables, command-line arguments, port numbers, URLs, and feature flags
- `If we use env inside of pods and deployments we can't reuse them. But with the ConfigMaps we can reuse them`
- We create kubernates object with all the envs

## Why Use ConfigMaps?
- Keep your app configuration separate from container images
- Easily change app behavior without rebuilding the image
- Make config reusable and manageable across multiple pods
- Configurable at runtime via environment variables, command-line args, or mounted files

## Basic Example
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

## Ways to Use a ConfigMap
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

## Examples
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

## Example
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
