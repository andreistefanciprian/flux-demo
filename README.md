# GitOps Kubernetes Deployments with FluxCD

This repository contains Kubernetes manifests managed by [FluxCD](https://fluxcd.io/) for automated GitOps deployments to a [private GKE cluster](https://github.com/andreistefanciprian/terraform-kubernetes-gke-cluster).

## 📦 Deployed Components

- **[FluxCD](https://fluxcd.io/flux/)** - GitOps toolkit for Kubernetes
- **[Istio](https://github.com/istio/istio/tree/master/manifests/charts)** - Service mesh
- **[Kube-Prometheus-Stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)** - Monitoring and alerting
- **[Cert-Manager](https://cert-manager.io/docs/installation/helm/)** - Automatic TLS certificate management
- **[Gateway API](https://gateway-api.sigs.k8s.io/)** - Kubernetes Gateway API CRDs
- **[Secrets Store CSI Driver](https://secrets-store-csi-driver.sigs.k8s.io/introduction)** - Secret management integration
- **[Go Demo App](https://github.com/andreistefanciprian/go-demo-app)** - Demo application
- **[Pod Restarter](https://github.com/andreistefanciprian/pod-restarter-go)** - Automated pod restart utility

## 🚀 Initial Setup

### Prerequisites

- GitHub personal access token with repo permissions
- GKE cluster provisioned via [terraform](https://github.com/andreistefanciprian/terraform-kubernetes-gke-cluster)
- `kubectl` configured to access your cluster

### Installation Steps

1. **Set environment variables:**
   ```bash
   export GITHUB_TOKEN=<your_github_token>
   export GITHUB_USER=andreistefanciprian
   ```

2. **Configure GCP project:**
   
   Update the `GCP_PROJECT` variable in:
   - `clusters/home/flux-system/cluster-vars.yaml`
   - `clusters/home/flux-system/_patches/gar-workload-identity.yaml`
   
   > **Note:** Variables in the ConfigMap are propagated across all manifests in the `./infra` folder.

3. **Install FluxCD:**
   ```bash
   brew install fluxcd/tap/flux
   
   flux bootstrap github \
     --components-extra=image-reflector-controller,image-automation-controller \
     --owner=$GITHUB_USER \
     --repository=flux-demo \
     --branch=main \
     --path=./clusters/home \
     --personal \
     --token-auth \
     --reconcile=true
   ```

## 🔍 Monitoring & Debugging

### Check Flux Resources

```bash
# View all Flux resources
kubectl get kustomizations -A
kubectl get helmrepositories -A
kubectl get helmreleases -A
kubectl get gitrepositories -A
kubectl get imagerepositories -A
kubectl get imageupdateautomations -A

# Check Helm releases
helm list -A
helm get manifest <release-name>
```

### Reconcile Resources

```bash
# Force reconciliation of a specific app
flux reconcile kustomization <app-name> --with-source

# Example:
flux reconcile kustomization secrets-store-csi-driver --with-source
```

### View Logs

```bash
# Flux controller logs
kubectl -n flux-system logs -l app=helm-controller -f

# Application logs (example: descheduler)
kubectl -n descheduler logs -l app.kubernetes.io/instance=descheduler -f
```

### Delete an Application

```bash
kubectl delete kustomization <app-name> -n flux-system

# Example:
kubectl delete kustomization secrets-store-csi-driver -n flux-system
```

## 🔐 Token Management

If your GitHub token expires, update the Flux secret:

```bash
# Generate base64 encoded token
echo -n 'ghp_yourNewTokenHere' | base64

# Edit the secret and replace data.password with the new base64 value
kubectl edit secret flux-system -n flux-system
```