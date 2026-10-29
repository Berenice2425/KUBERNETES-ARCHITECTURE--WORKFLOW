# Kubernetes Architecture & Workloads

Provisioned an Azure Kubernetes Service (AKS) cluster, deployed a containerized workload exposed via an Azure Standard LoadBalancer Service, configured multi-stage application health probes (Startup, Liveness, and Readiness), set up automated horizontal pod autoscaling based on CPU utilization, and implemented multi-tenant team isolation using namespaces and strict compute resource quotas.

## 🛠️ Cluster Architecture & Commands Covered

### 1. Cloud Provisioning & Authentication
Before interacting with the cluster, authenticate your local terminal context against your Azure subscription:

```powershell
# Authenticate and target your tenant if hitting multi-factor checks
az login --tenant <your-tenant-id>

# Download cluster credentials (kubeconfig) to your local machine
az aks get-credentials --resource-group rg-freightflow --name aks-freightflow
```

### 2. Core Operational Infrastructure
Execute the following commands to provision, deploy, and scale your workloads. 

*Note: Multi-line commands are formatted using PowerShell backticks (`` ` ``). If running in a Linux/Bash shell, swap the backticks to backslashes (`\`).*

```powershell
# Provision a 2-node managed AKS cluster using production-ready CNI networking
az aks create `
  -g rg-freightflow `
  -n aks-freightflow `
  --node-count 2 `
  --node-vm-size Standard_D2s_v5 `
  --enable-managed-identity `
  --generate-ssh-keys `
  --network-plugin azure

# Deploy the core freightflow application controllers
kubectl apply -f freightflow-deploy.yaml

# Expose the API to the public internet via an external Azure Load Balancer
kubectl apply -f freightflow-svc.yaml

# Update workloads with resilient startup, liveness, and readiness probes
kubectl apply -f freightflow-probes.yaml

# Configure the Horizontal Pod Autoscaler (HPA) targeting 50% average CPU usage
kubectl autoscale deployment freightflow-api --min=3 --max=10 --cpu=50%

# Establish governance boundaries and resource constraints for your teams
kubectl apply -f quota-team-api.yaml
```

## 🔍 Validation & Diagnostic Baselines

To verify that your cluster and workloads are working properly, use these diagnostic verification commands:

```powershell
# Inspect physical cluster infrastructure topology
kubectl get nodes -o wide
kubectl cluster-info

# Track deployment rollout and verify pod lifecycle state transitions
kubectl get pods -l app=freightflow-api -w

# Monitor real-time Horizontal Pod Autoscaler (HPA) metric targets
kubectl get hpa freightflow-api -w

# Fetch the public web entrypoint IP address mapped by Azure
kubectl get service freightflow-api-svc
```
