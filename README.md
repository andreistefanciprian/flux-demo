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

- GitHub App for Flux authentication ([setup guide](https://fluxcd.io/blog/2025/04/flux-operator-github-app-bootstrap/#github-app-docs))
- GKE cluster provisioned via [terraform](https://github.com/andreistefanciprian/terraform-kubernetes-gke-cluster)
- `kubectl` configured to access your cluster
- `helmfile` installed locally

### Installation Steps

#### 1. Create GitHub App Secret

Follow the [Flux Operator GitHub App docs](https://fluxcd.io/blog/2025/04/flux-operator-github-app-bootstrap/#github-app-docs) to create a GitHub App, then create the Kubernetes secret:

```bash
flux create secret githubapp flux-system \
  --app-id=<app_id> \
  --app-installation-id=<app_install_id> \
  --app-private-key=<private_key.pem>
```

#### 2. Update Configuration

Update the `GCP_PROJECT` variable in:
- `clusters/home/flux-system/cluster-vars.yaml`
- `clusters/home/flux-system/values-flux-instance.yaml`

> **Note:** Variables in the ConfigMap are propagated across all manifests in the `./infra` folder.

#### 3. Deploy Flux Operator

```bash
# Preview changes
helmfile -f clusters/home/flux-system/helmfile.yaml diff -l name=flux-operator

# Deploy operator
helmfile -f clusters/home/flux-system/helmfile.yaml apply -l name=flux-operator
```

#### 4. Deploy Flux Instance

```bash
# Preview changes
helmfile -f clusters/home/flux-system/helmfile.yaml diff -l name=flux-instance

# Deploy instance
helmfile -f clusters/home/flux-system/helmfile.yaml apply -l name=flux-instance
```

#### 5. Access Flux Operator UI

```bash
# Forward port to access the operator UI
kubectl port-forward svc/flux-operator -n flux-system 9080:9080

# Open in browser: http://localhost:9080
```


## 🔍 Monitoring & Debugging

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

# Force reconciliation of a specific app
flux reconcile kustomization <app-name> --with-source

# Flux controller logs
kubectl -n flux-system logs -l app=helm-controller -f

# Delete application
kubectl delete kustomization <app-name> -n flux-system

# If your GitHub token expires, update the Flux secret
# Generate base64 encoded token
echo -n 'ghp_yourNewTokenHere' | base64

# Edit the secret and replace data.password with the new base64 value
kubectl edit secret flux-system -n flux-system
```