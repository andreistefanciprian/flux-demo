
# Gitops kubernetes deployments with fluxcd

The k8s resources associated with this repository can be deployed in a [private GKE cluster](https://github.com/andreistefanciprian/terraform-kubernetes-gke-cluster).

K8s resources to be deployed with flux:
* fluxcd
* istio
* prometheus
* [go-demo-app](https://github.com/andreistefanciprian/go-demo-app)
* [pod-restarter](https://github.com/andreistefanciprian/pod-restarter-go)
* [descheduler](https://github.com/kubernetes-sigs/descheduler)

## Deploy flux and other k8s resources to GKE cluster

```
export GITHUB_TOKEN=<GITHUB_TOKEN_WITH_REPO_PERMISSIONS>
export GITHUB_USER=andreistefanciprian

# install flux into cluster
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

## Debug commands

```
# check flux k8s objects status
kubectl get kustomizations -A
kubectl get helmrepositories -A
kubectl get helmreleases -A
kubectl get gitrepositories -A
kubectl get imagerepositories -A
kubectl get imageupdateautomations -A

kubectl -n flux-system logs -l app=helm-controller -f
kubectl -n descheduler logs -l app.kubernetes.io/instance=descheduler -f
```