# Service

<br />

![Service Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1752943857/f056d0da-b594-4f38-b052-a555ea0dcd13.png)
![Service External Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1752944168/6317e539-88b4-4dcc-ada3-04aef1568430.png)

<br />

#### What is a Kubernetes Service?
- A Service in Kubernetes is an abstraction that defines a logical set of Pods and a policy by which to access them.
- Pods have dynamic IPs and are ephemeral (can die/restart anytime).
- A Service provides a stable IP and DNS name to reliably access Pods.
- It enables communication between different parts of your app or with the outside world.

#### Why Services?
- Pods can come and go. Their IP addresses change.
- You don’t want clients to keep track of individual Pods.
- Services act as a load balancer and stable endpoint for a group of Pods.

#### How Services Work
- Services select Pods by label selectors.
- They route traffic to the matching Pods using round-robin load balancing.
- Kubernetes assigns a Cluster IP for internal access.
- Services are backed by iptables/IPVS rules.

## Types of Services

![Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1767850187/WhatsApp_Image_2026-01-08_at_10.51.36_lwgx3m.jpg)

### ClusterIP (Default)
- Accessible only within the cluster.
- Ideal for internal communication between services.
- Example use: A backend service used by a frontend inside the same cluster.
```yaml
type: ClusterIP
```

### NodePort
![Service Works Image](https://res.cloudinary.com/djgwvmcdl/image/upload/v1752944279/62808821-5622-4abc-817e-4ff40fe95c99.png)
- Exposes the service on each Node's IP at a static port (30000–32767).
- You can access it as: http://<NodeIP>:<NodePort>
- Example use: Quick access for debugging, basic external traffic entry.
```yaml
type: NodePort
```

### LoadBalancer
- Provisions an external load balancer (via cloud provider).
- Exposes the service externally using the cloud provider’s load balancer.
- When we create a load balancer it will go to the cloud provider and create a loadbalancer in the cloud service.
- Example use: Production-level public access to an app (e.g., AWS ELB, GCP LB).
```yaml
type: LoadBalancer
```

### ExternalName
- Maps a service to an external DNS name.
- Used to redirect in-cluster traffic to external services.
```yaml
type: ExternalName
externalName: api.external.com
```

| Concept             | Description                                    |
| ------------------- | ---------------------------------------------- |
| **Selector**        | Matches Pods based on labels                   |
| **Port**            | Port exposed by the Service (clients use this) |
| **targetPort**      | Port on the Pod container                      |
| **Cluster IP**      | Internal IP assigned to the Service            |
| **NodePort**        | Port on each Node to access service externally |
| **LoadBalancer IP** | External IP allocated by cloud provider        |

<br />

### Examples
- Expose a deployment
```cmd
kubectl expose deploy nginx-deployment --port=80
```

- Create Service yaml file
```cmd
kubectl get service --dry-run=client -o yaml > service.yaml
```

- Describe a Service
```cmd
kubectl describe service <service_name>
```

- How to use Minikube service
```cmd
minikube service <name_service>
```

- Get endpoints
```cmd
kubectl get endpoints
```

- Delete service
```cmd
kubectl delete service <service_name>
```

<br />
<br />