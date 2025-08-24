# SETUP PROMETHEUS NODE EXPORTER ON KUBERNETES

# Introduction

Prometheus is a widely-used monitoring system that collects and processes metrics from various sources. The Node Exporter is Prometheus exporter that collects hardware and operating system metrics from a system. By deploying Node Exporter on Kubernetes, I can monitor the nodes in my Kubernetes cluster and gain insights into their performance.

# Objectives

1. Understand the purpose of Prometheus Node Exporter.
2. Deploy Node Exporter as a DaemonSet in a Kubernetes cluster.
3. Configure Prometheus to scrape metrics from Node Exporter.
4. Visualize metrics using Prometheus UI.
5. Explore metrics available through Node Exporter.

# Prerequisites

1. **Kubernetes Cluster**: A working Kuberntes cluster (e.g., Minikube, Kind, or a managed kubernetes service like EKS or AKS or GKE).
2. **Kubernetes CLI**: kubectl Installed and configured.
3. **Prometheus Setup**: Basic Prometheus installation running in the Kubernetes cluster.
4. **Tools**: A text editor to modify YAML files.


# Project Tasks

## Task 1 - Understand How Node Exporter Works

1. Node Exporter is a lightweight application that runs on a node and exposes metrics about the node's hardware and operating system.

2. Key metrics include:

- CPU and memory usage
- Disk I/O
- Metwork statistics
- Filesystem usage
3. Node Exporter runs as a containerized application in Kubernetes to collect metrics from each node.

## Task 2 - Deploy Node Exporter as a DaemonSet

1. I create new directory which is monitoring.

``` bash
mkdir monitoring 
cd monitoring
```

2. Then create a new file inside the directory monitoring.


``` bash
# vi node-exporter-daemonset.yaml


```bash
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      containers:
        - name: node-exporter
          image: prom/node-exporter:latest
          ports:
            - containerPort: 9100
              name: metrics
          securityContext:
            runAsNonRoot: true
            allowPrivilegeEscalation: false
          resources:
            limits:
              memory: "100Mi"
              cpu: "100m"
            requests:
              memory: "50Mi"
              cpu: "50m"
```

3. Then I create a new namaspace which I name monitoring

``` bash
kubectl create ns monitoring
kubectl get ns  # to verify the namespace I create
```

![](./Images/1.%20Namespace.png)


3. Apply the YAML file using kubectl.

``` bash
kubectl apply -f node-exporter-daemonset.yaml
```

4. Verify the deployment.

``` bash
kubectl get daemonset -n monitoring
```

![](./Images/2.%20DaemonSet.png)


5. Then I create another file for service node-exporter-service.yaml

``` bash
apiVersion: v1
kind: Service
metadata:
  name: node-exporter
  namespace: monitoring
  labels:
    app: node-exporter
spec:
  selector:
    app: node-exporter
  ports:
    - name: metrics
      port: 9100
      targetPort: 9100
  type: ClusterIP
```

6. Then apply the service file

``` bash
kubectl apply -f node-exporter-service.yaml
```

![](./Images/3.%20Service.png)


# ask 3 - Configure Prometheus to Scrape Metric from Node Exporter

1. Edit the Prometheus configuration to add a scrape job for Node Exporter.

``` bash
scrape_configs:
  - job_name: 'node-exporter'
    kubernetes_sd_configs:
      - role: endpoints
    relabel_configs:
      - source_labels: [__meta_kubernetes_service_label_app]
        action: keep
        regex: node-exporter
```

2. Apply the updated Prometheus configuration.
3. Restart the Prometheus deployment to load the new configuration.

![](./Images/4.%20Minikube%20Service.png)


# Task 4 - Verify Metrics in Prometheus

1. Access the Prometheus UI (e.g., by port-forwarding) This is done using putty ssh tunnel.
2. In the Prometheus UI, run a query to view Node Exporter metrics.
- Example: `node_cpu_seconds_total`
3. Ensure metrics are being collected for all cluster nodes.


![](./Images/5.%20Browser.png)


# Task 5 - Explore Metrics Provided by Node Exporter

1. List and understand key metrics.
- `node_memory_MemAvailable_bytes`: Available memory on the node.
- `node_filesystem_avail_bytes`: Free space on filesystem.
- `node_network_receive_bytes_total`: Total network bytes received.
2. Use Prometheus expression to analyze data e.g.,:
- rate(node_network_receive_bytes_total[5m])
3. Optionally, set up alerts for critical metrics in Prometheus.

# Conclusion

By completing this project, I have set up Prometheus Node Exporter on Kubernetes, enabling comprehensive monitoring of node-level metrics.

I also integrate Node Exporter with Prometheus, learned to query metrics, and explored the data it provides. This setup can now be extended with dashboard(e.g., Grafana) or alerts for advanced monitoring needs.