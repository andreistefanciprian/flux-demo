
#### Description

Explore gitops kubernetes deployments with fluxcd.

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

# debug commands
kubectl get kustomizations -A
kubectl get helmreleases -A
kubectl get gitrepositories -A
kubectl get imagerepositories -A
kubectl get imageupdateautomations -A
kubectl -n descheduler logs -l app.kubernetes.io/instance=descheduler -f
```