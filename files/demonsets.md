# Demonset
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
