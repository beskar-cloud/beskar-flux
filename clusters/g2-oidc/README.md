# g2-oidc cloud cluster / environment

## Flux CD bootsrapping commands

```sh
[freznicek@lenovo-t14 commandline 0]$ age-keygen -o ~/.config/sops/age/g2-oidc-key.txt
Public key: age1wv9468v9dcmzxksl4cmnny9pw679zh29cvc5u9fl42g6s0f4h9cq8yjzx5

[freznicek@lenovo-t14 ~ 0]$ kubectl --context=g2-oidc version
Client Version: version.Info{Major:"1", Minor:"24", GitVersion:"v1.24.9", GitCommit:"9710807c82740b9799453677c977758becf0acbb", GitTreeState:"clean", BuildDate:"2022-12-08T10:15:09Z", GoVersion:"go1.18.9", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v4.5.4
Server Version: version.Info{Major:"1", Minor:"24", GitVersion:"v1.24.9", GitCommit:"9710807c82740b9799453677c977758becf0acbb", GitTreeState:"clean", BuildDate:"2022-12-08T10:08:06Z", GoVersion:"go1.18.9", Compiler:"gc", Platform:"linux/amd64"}
[freznicek@lenovo-t14 ~ 0]$ kubectl --context=g2-oidc get no
NAME               STATUS   ROLES           AGE    VERSION
cln-14             Ready    <none>          132m   v1.24.9
controlplane-001   Ready    control-plane   134m   v1.24.9
controlplane-002   Ready    control-plane   134m   v1.24.9
controlplane-003   Ready    control-plane   134m   v1.24.9

[freznicek@lenovo-t14 ~ 0]$ flux --context=g2-oidc  check --pre
► checking prerequisites
✗ flux 0.38.3 <0.39.0 (new version is available, please upgrade)
✔ Kubernetes 1.24.9 >=1.20.6-0
✔ prerequisites checks passed


# g2/ceph-openstack-lma/-/settings/access_tokens token named: g2-oidc-fluxcd-token
[freznicek@lenovo-t14 ~ 0]$ flux --context=g2-oidc bootstrap gitlab --hostname gitlab.ics.muni.cz --owner=cloud/g2 --repository=ceph-openstack-lma --branch downstream --path clusters/g2-oidc --token-auth
Please enter your GitLab personal access token (PAT):
► connecting to https://gitlab.ics.muni.cz
► cloning branch "downstream" from Git repository "https://gitlab.ics.muni.cz/cloud/g2/ceph-openstack-lma.git"
✔ cloned repository
► generating component manifests
✔ generated component manifests
✔ committed sync manifests to "downstream" ("23aab14bb01afdd1e285967ab8f841f7bb251cc9")
► pushing component manifests to "https://gitlab.ics.muni.cz/cloud/g2/ceph-openstack-lma.git"
► installing components in "flux-system" namespace
✔ installed components
✔ reconciled components
► determining if source secret "flux-system/flux-system" exists
► generating source secret
► applying source secret "flux-system/flux-system"
✔ reconciled source secret
► generating sync manifests
✔ generated sync manifests
✔ committed sync manifests to "downstream" ("2ee4361eb781ff05dd0becfafaaf355f36b9efa6")
► pushing sync manifests to "https://gitlab.ics.muni.cz/cloud/g2/ceph-openstack-lma.git"
► applying sync manifests
✔ reconciled sync configuration
◎ waiting for Kustomization "flux-system/flux-system" to be reconciled
✔ Kustomization reconciled successfully
► confirming components are healthy
✔ helm-controller: deployment ready
✔ kustomize-controller: deployment ready
✔ notification-controller: deployment ready
✔ source-controller: deployment ready
✔ all components are healthy


[freznicek@lenovo-t14 ~ 0]$ flux --context=g2-oidc  check
► checking prerequisites
✗ flux 0.38.3 <0.39.0 (new version is available, please upgrade)
✔ Kubernetes 1.24.9 >=1.20.6-0
► checking controllers
✔ helm-controller: deployment ready
► ghcr.io/fluxcd/helm-controller:v0.28.1
✔ kustomize-controller: deployment ready
► ghcr.io/fluxcd/kustomize-controller:v0.32.0
✔ notification-controller: deployment ready
► ghcr.io/fluxcd/notification-controller:v0.30.2
✔ source-controller: deployment ready
► ghcr.io/fluxcd/source-controller:v0.33.0
► checking crds
✔ alerts.notification.toolkit.fluxcd.io/v1beta2
✔ buckets.source.toolkit.fluxcd.io/v1beta2
✔ gitrepositories.source.toolkit.fluxcd.io/v1beta2
✔ helmcharts.source.toolkit.fluxcd.io/v1beta2
✔ helmreleases.helm.toolkit.fluxcd.io/v2beta1
✔ helmrepositories.source.toolkit.fluxcd.io/v1beta2
✔ kustomizations.kustomize.toolkit.fluxcd.io/v1beta2
✔ ocirepositories.source.toolkit.fluxcd.io/v1beta2
✔ providers.notification.toolkit.fluxcd.io/v1beta2
✔ receivers.notification.toolkit.fluxcd.io/v1beta2
✔ all checks passed

[freznicek@lenovo-t14 ~ 0]$ cat ~/.config/sops/age/g2-oidc-key.txt | kubectl --context=g2-oidc create secret generic sops-age-key-g2-oidc --namespace=flux-system --from-file=age.agekey=/dev/stdin
secret/sops-age-key-g2-oidc created

```

## Flux CD in-place upgrade commands

```sh
freznicek@lenovo-t14 02-rook-ceph-external 0]$ flux check
► checking prerequisites
✗ flux 0.41.2 <2.0.0-rc.1 (new version is available, please upgrade)
✔ Kubernetes 1.24.9 >=1.20.6-0
► checking controllers
✔ helm-controller: deployment ready
► ghcr.io/fluxcd/helm-controller:v0.28.1
✔ kustomize-controller: deployment ready
► ghcr.io/fluxcd/kustomize-controller:v0.32.0
✔ notification-controller: deployment ready
► ghcr.io/fluxcd/notification-controller:v0.30.2
✔ source-controller: deployment ready
► ghcr.io/fluxcd/source-controller:v0.33.0
► checking crds
...
✔ all checks passed
[freznicek@lenovo-t14 ~ 2]$ flux --context=g2-oidc bootstrap gitlab --hostname gitlab.ics.muni.cz --owner=cloud/g2 --repository=ceph-openstack-lma --branch downstream --path clusters/g2-oidc --token-auth
Please enter your GitLab personal access token (PAT):
► connecting to https://gitlab.ics.muni.cz
► cloning branch "downstream" from Git repository "https://gitlab.ics.muni.cz/cloud/g2/ceph-openstack-lma.git"
✔ cloned repository
► generating component manifests
✔ generated component manifests
✔ committed sync manifests to "downstream" ("1ed21030dbbb0f1303cc6d29f2bcec6adc7f3619")
► pushing component manifests to "https://gitlab.ics.muni.cz/cloud/g2/ceph-openstack-lma.git"
✔ reconciled components
► determining if source secret "flux-system/flux-system" exists
► generating source secret
► applying source secret "flux-system/flux-system"
✔ reconciled source secret
► generating sync manifests
✔ generated sync manifests
✔ sync manifests are up to date
► applying sync manifests
✔ reconciled sync configuration
◎ waiting for Kustomization "flux-system/flux-system" to be reconciled
✔ Kustomization reconciled successfully
► confirming components are healthy
✔ helm-controller: deployment ready
✔ kustomize-controller: deployment ready
✔ notification-controller: deployment ready
✔ source-controller: deployment ready
✔ all components are healthy
[freznicek@lenovo-t14 ~ 0]$ flux check
► checking prerequisites
✗ flux 0.41.2 <2.0.0-rc.1 (new version is available, please upgrade)
✔ Kubernetes 1.24.9 >=1.20.6-0
► checking controllers
✔ helm-controller: deployment ready
► ghcr.io/fluxcd/helm-controller:v0.31.2
✔ kustomize-controller: deployment ready
► ghcr.io/fluxcd/kustomize-controller:v0.35.1
✔ notification-controller: deployment ready
► ghcr.io/fluxcd/notification-controller:v0.33.0
✔ source-controller: deployment ready
► ghcr.io/fluxcd/source-controller:v0.36.1
► checking crds
...
✔ all checks passed
```
