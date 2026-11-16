# Cloud Monitoring & Observability Overview

## What is Monitoring and Observability?

In modern cloud and DevOps environments, simply knowing whether a server is "up" or "down" is no longer enough. Observability is the practice of understanding the internal state of complex distributed systems by analyzing three primary telemetry pillars:

1. **Metrics:** Numerical values representing system behavior measured over intervals of time (CPU utilization, memory usage, request counts, network throughput).
2. **Logs:** Immutable, timestamped text records of discrete events (HTTP 500 error logs, container crashes, system syslog).
3. **Traces:** End-to-end journey maps of requests traversing microservices (distributed tracing showing latency bottlenecks across APIs).

---

## The 6 Levels of Monitoring for DevOps Engineers

Abhishek Veeramalla highlights 6 critical architectural levels that every DevOps engineer must monitor to build a resilient cloud infrastructure:

```
+-----------------------------------------------------------------------------+
| Level 6: Application Performance Monitoring (APM)                            |
|          Application Insights, HTTP request rate, error rates, dependencies  |
+-----------------------------------------------------------------------------+
| Level 5: Data Plane / Workload Monitoring (Kubernetes / Containers)         |
|          Pod crashes (CrashLoopBackOff, OOMKilled), replica counts, restarts|
+-----------------------------------------------------------------------------+
| Level 4: Control Plane Monitoring                                           |
|          kube-apiserver response times, etcd latency, scheduler queue depth |
+-----------------------------------------------------------------------------+
| Level 3: Node / VMSS Infrastructure Monitoring                              |
|          Worker node CPU, RAM, disk I/O, ephemeral disk exhaustion          |
+-----------------------------------------------------------------------------+
| Level 2: Azure Native Cloud Resource Monitoring                             |
|          Storage account IOPS, Azure SQL DTUs, Key Vault request rates      |
+-----------------------------------------------------------------------------+
| Level 1: Network Layer Monitoring                                           |
|          Azure Network Watcher, NSG flow logs, dropped packets, DNS delays  |
+-----------------------------------------------------------------------------+
```

---

### Level 1: Network Monitoring
- **Tools:** Azure Network Watcher, NSG Flow Logs, Traffic Analytics.
- **What to track:** Dropped packets, blocked IP traffic by NSG rules, latency across VNet peering connections, VPN gateway bandwidth.

### Level 2: Azure Native Cloud Resources
- **Tools:** Azure Monitor Metrics.
- **What to track:** Azure Storage account throughput and ingress/egress throttling, Key Vault throttling, Azure SQL database utilization.

### Level 3: Compute & Node Infrastructure
- **Tools:** Azure Monitor Agent (AMA), Telegraf, Node Exporter.
- **What to track:** CPU saturation, memory exhaustion, disk queue depth, network interface drops on Azure VMs and VMSS nodes.

### Level 4: Kubernetes Control Plane
- **Tools:** Azure Monitor Managed Service for Prometheus, Container Insights.
- **What to track:** API server request duration and rate, etcd write latency, controller manager sync delays.

### Level 5: Kubernetes Data Plane / Workloads
- **Tools:** kube-state-metrics, Prometheus, Grafana.
- **What to track:** Pod phase (`Pending`, `Failed`), container restarts, memory limits vs usage (OOM risks), deployment rollout status.

### Level 6: Application Performance Monitoring (APM)
- **Tools:** Azure Application Insights, OpenTelemetry.
- **What to track:** HTTP request duration, failed requests (4xx / 5xx), SQL query execution time, exceptions, unhandled promises.
