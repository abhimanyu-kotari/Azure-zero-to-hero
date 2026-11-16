# AKS vs Self-Managed Kubernetes: Architectural Comparison

A critical architectural decision for DevOps teams is choosing between a managed service like **Azure Kubernetes Service (AKS)** and a **Self-Managed Kubernetes** cluster (deployed via `kubeadm`, `kops`, or `Kubespray` on Azure VMs or bare metal).

---

## Detailed Comparison Matrix

| Aspect | Azure Kubernetes Service (AKS) | Self-Managed Kubernetes (kubeadm / VMs) |
| :--- | :--- | :--- |
| **Control Plane Management** | 100% managed and automated by Microsoft. Master nodes are invisible and abstracted. | Team must manually deploy, patch, monitor, and back up 3+ master nodes. |
| **Control Plane Cost** | **Free** (standard tier) or ~$73/month with 99.95% financially backed SLA. | You pay for all master VMs, OS licenses, disks, and bandwidth (easily $300-$1000+/mo). |
| **etcd High Availability & Backups**| Automated multi-zone etcd clustering and automated snapshots managed by Azure. | Manual etcd clustering, quorum management, TLS certificates, and disaster recovery snapshot scripts. |
| **Cluster Upgrades** | 1-click upgrade via Portal, CLI (`az aks upgrade`), or Terraform. Automated rolling node updates. | Complex manual orchestration: drain node, upgrade kubeadm, upgrade kubelet/kubectl, uncordon, repeat. High risk of control plane split-brain. |
| **Worker Node Scaling** | Native **Cluster Autoscaler** integrated with Azure VMSS. Fast, automatic scale-out and scale-in. | Must configure custom autoscaling scripts or standalone cluster-autoscaler binaries with cloud provider hooks. |
| **Storage Integration** | Native CSI drivers pre-installed (`disk.csi.azure.com`, `file.csi.azure.com`, `blob.csi.azure.com`). Dynamic volume provisioning out-of-the-box. | Must manually install, configure, test, and maintain cloud provider CSI drivers and storage classes. |
| **Identity & Access Management**| Seamless integration with Microsoft Entra ID (Azure AD), Azure RBAC, and Workload Identity. | Must configure custom OIDC providers, external authenticators, and manage Kubernetes kubeconfig certificates manually. |
| **Day-2 Operations & Monitoring** | 1-click Azure Monitor Container Insights, Managed Prometheus, and Managed Grafana. | Must manually deploy, configure, secure, and maintain Prometheus Operator, Alertmanager, Grafana, and persistent storage. |
| **Security & Compliance** | Certified HIPAA, ISO, PCI-DSS, SOC. Automated OS security patches applied to underlying node images. | Compliance and OS security patching are entirely your responsibility. |

---

## Real-World Scenarios: When to Choose What?

### Choose Azure Kubernetes Service (AKS) when:
1. **Focus on Applications, Not Infrastructure:** Your organization wants engineering teams to build and deploy business logic rather than spending 40% of their time babysitting control planes.
2. **Cost Optimization:** You want to avoid paying compute and disk costs for dedicated master nodes.
3. **Automated Lifecycle:** You need reliable, non-disruptive Kubernetes version upgrades and OS patch rollouts.
4. **Cloud-Native Integration:** Your application requires native connectivity with Azure Key Vault, ACR, Application Gateway Ingress Controller (AGIC), and Azure Virtual Networks.

### Choose Self-Managed Kubernetes when:
1. **Custom Control Plane Flags:** You require custom, esoteric alpha/beta API flags or low-level kernel configurations on master nodes that Azure restricts.
2. **Hybrid / Air-Gapped Environments:** You are deploying in isolated on-premises datacenters without public internet connectivity.
3. **Specific Custom OS Distros:** You are mandated by legacy corporate compliance to run a specific proprietary Linux distribution on all master and worker nodes that AKS does not support.
