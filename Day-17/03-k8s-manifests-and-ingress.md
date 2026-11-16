# Kubernetes Manifests and Ingress Controller on AKS

To run our 3-tier e-commerce application on Azure Kubernetes Service, we write declarative Kubernetes YAML manifests for Deployments, Services, and Ingress routing.

---

## What is an Ingress Controller and How Does it Work?

- A standard Kubernetes `Service` of type `ClusterIP` is only accessible inside the cluster.
- A `Service` of type `LoadBalancer` provisions an Azure Public Load Balancer and public IP for *each* service. In a microservices architecture with 10+ services, provisioning 10 public load balancers is expensive and difficult to maintain.
- An **Ingress Controller** acts as an intelligent Layer 7 reverse proxy (typically NGINX or Envoy). A single Azure Public Load Balancer routes all external traffic into the Ingress Controller, which inspects the HTTP Host header and URL path to route requests to the correct backend `ClusterIP` service:

```
[ External User Request: / ] ---------> [ Ingress Controller ] -------> [ web:8080 ]
[ External User Request: /api/cart ] -> [ Ingress Controller ] -------> [ cart:8080 ]
[ External User Request: /api/user ] -> [ Ingress Controller ] -------> [ user:8080 ]
```

---

## Step 1: Install NGINX Ingress Controller on AKS

Run Helm to deploy the NGINX Ingress Controller into the `ingress-basic` namespace:

```bash
# Add the ingress-nginx repository
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# Deploy NGINX Ingress Controller with Azure Load Balancer integration
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --create-namespace \
  --namespace ingress-basic \
  --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"=/healthz

# Wait for Azure to assign an External Public IP address
kubectl get service ingress-nginx-controller --namespace ingress-basic -w
```

---

## Step 2: Deploy Application Manifests

Apply the Kubernetes manifests in the `manifests/` directory:

```bash
# 1. Create the application namespace
kubectl apply -f manifests/01-namespace.yaml

# 2. Deploy database tier (MongoDB & Redis)
kubectl apply -f manifests/02-databases.yaml

# 3. Deploy microservices tier (8 services)
kubectl apply -f manifests/03-microservices.yaml

# 4. Deploy Ingress routing rules
kubectl apply -f manifests/04-ingress.yaml
```

---

## Step 3: Verify the Deployment

```bash
# Check all pods are Running
kubectl get pods -n ecommerce -o wide

# Check services
kubectl get svc -n ecommerce

# Check ingress resource and assigned IP
kubectl get ingress -n ecommerce
```

Open a web browser and navigate to the external Ingress IP address (e.g. `http://<INGRESS_EXTERNAL_IP>/`). The e-commerce storefront should load smoothly, allowing users to browse products, add items to the cart, and simulate checkout.
