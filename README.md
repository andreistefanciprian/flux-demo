
# Gitops kubernetes deployments with fluxcd

The k8s resources associated with this repository can be deployed in a [private GKE cluster](https://github.com/andreistefanciprian/terraform-kubernetes-gke-cluster).

K8s resources to be deployed with flux:
* fluxcd
* istio
* prometheus
* [cert-manager](https://cert-manager.io/docs/installation/helm/) (used for automatic management and issuance of TLS certificates)
* [go-demo-app](https://github.com/andreistefanciprian/go-demo-app)
* [pod-restarter](https://github.com/andreistefanciprian/pod-restarter-go)
* [descheduler](https://github.com/kubernetes-sigs/descheduler)
* [secrets-store-csi-driver](https://secrets-store-csi-driver.sigs.k8s.io/introduction)

## Deploy flux and other k8s resources to GKE cluster

```
export GITHUB_TOKEN=<GITHUB_TOKEN_WITH_REPO_PERMISSIONS>
export GITHUB_USER=andreistefanciprian

# Note: update GCP_PROJECT var in clusters/home/flux-system/cluster-vars.yaml
# vars in this config map get propagated in the app flux manifests.

# Note: update GCP_PROJECT var in clusters/home/flux-system/_patches/gar-workload-identity.yaml and clusters/home/watchlist-slack-bot.yaml

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

# trigger cronjob for GAR secret
kubectl create job --from=cronjob/gcr-credentials-sync -n flux-system gcr-credentials-sync-init
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

helm list -A
helm get manifest <releaseName>

# delete one of the apps
kubectl delete kustomization secrets-store-csi-driver -n flux-system

# reconcile app
flux reconcile kustomization secrets-store-csi-driver --with-source

# logs
kubectl -n flux-system logs -l app=helm-controller -f
kubectl -n descheduler logs -l app.kubernetes.io/instance=descheduler -f

# if the git token expires the flux-system/flux-system secret needs to be updated
# replace data.password with the new token base64 value below 
echo -n ghp_newGeneratedTokenHere | base64
kubectl edit secret flux-system -n flux-system
```