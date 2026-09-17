# Flux CD Commands for DevOps Engineers

# This file contains a comprehensive list of Flux CD commands for various use cases.
# These commands are essential for managing GitOps workflows with Flux.

# Note: Replace placeholders (e.g., <your-github-username>, <source-name>, etc.) with your actual values.

# Installation
# ============

aws eks update-kubeconfig --name production-eks --region us-east-1

  # create Ecr secret for flux
aws ecr get-login-password --region us-east-2 | kubectl create secret docker-registry ecr-secret --docker-server=001526952227.dkr.ecr.us-east-2.amazonaws.com --docker-username=AWS --docker-password-stdin

aws ecr get-login-password --region us-east-1| kubectl create secret docker-registry ecr-secret \
  --docker-server=12345678910.dkr.ecr.us-east-2 .amazonaws.com \
  --docker-username=AWS \
  --docker-password=$(aws ecr get-login-password --region us-east-1) 

kubectl create secret docker-registry ecr-secret \
  --docker-server=12345678910.dkr.ecr.us-east-1.amazonaws.com \
  --docker-username=AWS \
  --docker-password=$(aws ecr get-login-password --region us-east-1) \
  --dry-run=client -o yaml > jenkins/ecr-secret.yaml


# Install Flux CLI on macOS using Homebrew
```sh
brew install fluxcd/tap/flux
```
 Install on windows
```sh
choco install flux
```
Install Flux CLI on Linux
```sh
curl -s https://fluxcd.io/install.sh | sudo bash
```
Verify the installation
```sh
flux --version
```
kubectl config use-context kind-production

# Bootstrapping
# =============
before bootstraping make sure to export your Username and token
```sh
export GITHUB_TOKEN=<your-token>
export GITHUB_USER=<your-username>
```

Bootstrap Flux using a GitHub repository
```sh
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=<your-repository-name> \
  --path=clusters/my-cluster \
  --token-auth
  --personal #remove this part if its an organization account
```
Install Flux with the image automation components:
```sh
flux bootstrap github \
  --components-extra=image-reflector-controller,image-automation-controller \
  --owner=$GITHUB_USER \
  --repository=flux-image-updates \
  --branch=main \
  --path=clusters/my-cluster \
  --read-write-key \
  --personal # remove for organization
```

# Bootstrap Flux using a GitLab repository
flux bootstrap gitlab \
  --owner=<your-gitlab-username> \
  --repository=<your-repository-name> \
  --path=clusters/my-cluster \
  --token-auth 
  --personal   #remove this part if its an organization account

# Bootstrap Flux using a generic Git repository
flux bootstrap git \
  --url=https://your-git-repo.com/owner/repository \
  --username=<your-username> \
  --password=<your-password> \
  --path=clusters/my-cluster


# Sources
# =======
age1t5adadzz40qssqlcque6jgws2yducsxwugp7adj5vue3tkkgvdaq4czaxn
# Manage Git repositories
# -----------------------

# Create a GitRepository source
flux create source git dev-team \
    --namespace=apps \
    --url=https://github.com/<org>/<dev-team> \
    --branch=main \
   # --secret-ref=gh-auth \ optional if public 
    --export > ./tenants/base/dev-team/sync.yaml

# creating a secrete to be added to a git repo, it should be in thesame namespace as the git repository.
kubectl create secret generic flux-system \
  --namespace=flux-system \  
  --from-literal=username=$GITHUB_USER \
  --from-literal=password=$GITHUB_TOKEN
  kubectl create secret generic gh-auth \
  --namespace=apps \
  --from-literal=username=$GITHUB_USER \
  --from-literal=password=$GITHUB_TOKEN
 #--export >> topath / if you wish to save it to a file

#sample git repository
apiVersion: source.toolkit.fluxcd.io/v1beta1
kind: GitRepository
metadata:
  name: apps
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/$GITHUB_USER/private-helm-charts
  secretRef:
    name: gh-auth
  ref:
    branch: main
# List Git sources
flux get sources git

flux get sources git -n namespace

flux reconcile source git flux-system
# Delete a Git source
flux delete source git <source-name>

# Manage Helm repositories
# ------------------------

# Add a Helm repository
flux create source helm <repo-name> \
  --url=https://charts.bitnami.com/bitnami \
  --interval=1h

# List Helm repositories
flux get sources helm

# Delete a Helm repository
flux delete source helm <repo-name>

# Kustomizations
# ==============

# Create a Kustomization
flux create kustomization dev-team \
    --namespace=apps \
    --service-account=dev-team \
    --source=GitRepository/dev-team \
    --path="./" \
    --export >> ./tenants/base/dev-team/sync.yaml

# sample kustomization resource
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: flux-system
  namespace: flux-system
spec:
  interval: 10m
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./tenants/staging
  prune: true
  wait: true
  timeout: 5m



# List Kustomizations
flux get kustomizations
flux get kustomizations --watch
flux get kustomizations --watch -n namespace

# Reconcile a Kustomization
flux reconcile kustomization --with-source

# Kustomize tool


#suspending updates 
# Suspending updates to a kustomization allows you to directly edit objects applied from a kustomization, 
# without your changes being reverted by the state in Git.
flux suspend kustomization <name>

#To resume updates run the command 
flux suspend kustomization <name>

# Delete a Kustomization
flux delete kustomization <name>

Kus

## Helm Releases
## =============
# creating a helm repository
```sh
flux create source helm
```
# Create a source for an HTTPS public Helm repository
  flux create source helm podinfo \
    --url=https://stefanprodan.github.io/podinfo \
    --interval=10m

  # Create a source for an HTTPS Helm repository using basic authentication
  flux create source helm podinfo \
    --url=https://stefanprodan.github.io/podinfo \
    --username=username \
    --password=password

  # Create a source for an HTTPS Helm repository using TLS authentication
  flux create source helm podinfo \
    --url=https://stefanprodan.github.io/podinfo \
    --cert-file=./cert.crt \
    --key-file=./key.crt \
    --ca-file=./ca.crt

  # Create a source for an OCI Helm repository
  flux create source helm podinfo \
    --url=oci://ghcr.io/stefanprodan/charts/podinfo \
    --username=username \
    --password=password

 #  sample helm repository
```sh
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
spec:
  interval: 24h
  url: https://kubernetes.github.io/ingress-nginx
```


  # Create a source for an OCI Helm repository using an existing secret with basic auth or dockerconfig credentials
  flux create source helm podinfo \
    --url=oci://ghcr.io/stefanprodan/charts/podinfo \
    --secret-ref=docker-config


## Create a HelmRelease
flux create helmrelease <name> \
  --source=HelmRepository/<repo-name> \
  --chart=<chart-name> \
  --target-namespace=default \
  --interval=5m \
  --export >> topath/

3 Example 
```sh
apiVersion: helm.toolkit.fluxcd.io/v2beta2
kind: HelmRelease
metadata:
  name: nginx
  namespace: nginx
spec:  
  chart:
    spec:
      chart: nginx-ingress
      version: 1.2.x
      sourceRef:
        kind: HelmRepository
        name: nginx
      interval: 1h
  interval: 1h  
  releaseName: nginx
```
## List HelmReleases
flux get helmreleases

# Reconcile a HelmRelease
flux reconcile helmrelease <name>

# Delete a HelmRelease
flux delete helmrelease <name>

# Reconciliation
# ==============

# Reconcile all resources in the flux-system namespace
flux reconcile source git flux-system

# Reconcile a specific Kustomization
flux reconcile kustomization <name>

# Reconcile a specific HelmRelease
flux reconcile helmrelease <name>

# Monitoring and Debugging
# ========================

# Check Flux components health
flux check

# Get logs for all Flux components
flux logs 

# Get logs for a specific Flux controller
flux logs --daemon=source-controller

# Export resources to yaml
flux export source git <source-name>
flux export kustomization <name>
flux export helmrelease <name>

# Uninstallation
# ==============

# Uninstall Flux and remove its components
flux uninstall --silent



#   ONBORDING NEW TENANTS IN A MULTI CLUSTER.

The Flux CLI offers commands to generate the Kubernetes manifests needed to define tenants.

Assuming a platform admin wants to create a tenant named `dev-team` with access to the `apps` namespace.

Create the tenant base directory:

```sh
mkdir -p ./tenants/base/dev-team
```

Generate the namespace, service account and role binding for the dev-team:

```sh
flux create tenant dev-team --with-namespace=apps \
    --export > ./tenants/base/dev-team/rbac.yaml
```

Create the sync manifests for the tenant Git repository:

```sh
flux create source git dev-team \
    --namespace=apps \
    --url=https://github.com/<org>/<dev-team> \
    --branch=main \
    --export > ./tenants/base/dev-team/sync.yaml

flux create kustomization dev-team \
    --namespace=apps \
    --service-account=dev-team \
    --source=GitRepository/dev-team \
    --path="./" \
    --export >> ./tenants/base/dev-team/sync.yaml
```

Create the base `kustomization.yaml` file:

```sh
cd ./tenants/base/dev-team/ && kustomize create --autodetect --namespace apps 
```

Create the staging overlay and set the path to the staging dir inside the tenant repository:

```sh
cat << EOF | tee ./tenants/staging/dev-team-patch.yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: dev-team
  namespace: apps
spec:
  path: ./staging
EOF

cat << EOF | tee ./tenants/staging/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: apps
resources:
  - ../base/dev-team
patches:
  - path: dev-team-patch.yaml
EOF
```

With the above configuration, the Flux instance running on the staging cluster will clone the
dev-team's repository, and it will reconcile the `./staging` directory from the tenant's repo
using the `dev-team` service account. Since that service account is restricted to the `apps` namespace,
the dev-team repository must contain Kubernetes objects scoped to the `apps` namespace only.

# Enforce tenant isolation

To enforce tenant isolation, cluster admins must configure Flux to reconcile 
the `Kustomization` and `HelmRelease` kinds by impersonating a service account
from the namespace where these objects are created. 

Flux has built-in [multi-tenancy lockdown] features which enables tenant isolation 
at Control Plane level without the need of external admission controllers (e.g. Kyverno). The
recommended patch:

- Enforce controllers to block cross namespace references.
  Meaning that a tenant can’t use another tenant’s sources or subscribe to their events.
- Deny accesses to Kustomize remote bases, thus ensuring all resources refer to local files. 
  Meaning that only approved Flux Sources can affect the cluster-state.
- Sets a default service account via `--default-service-account` to `kustomize-controller` and `helm-controller`.
  Meaning that, if a tenant does not specify a service account in a Flux `Kustomization` or 
  `HelmRelease`, it would automatically default to said account. 

> **NOTE:** It is recommended that the default service account has no privileges.
> And each named service account used observes the least privilege model.

This repository applies this patch automatically via
[kustomization.yaml](clusters/production/flux-system/kustomization.yaml) in both clusters.

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - gotk-components.yaml
  - gotk-sync.yaml
patches:
  - patch: |
      - op: add
        path: /spec/template/spec/containers/0/args/-
        value: --no-cross-namespace-refs=true
    target:
      kind: Deployment
      name: "(kustomize-controller|helm-controller|notification-controller|image-reflector-controller|image-automation-controller)"
  - patch: |
      - op: add
        path: /spec/template/spec/containers/0/args/-
        value: --no-remote-bases=true
    target:
      kind: Deployment
      name: "kustomize-controller"
  - patch: |
      - op: add
        path: /spec/template/spec/containers/0/args/-
        value: --default-service-account=default
    target:
      kind: Deployment
      name: "(kustomize-controller|helm-controller)"
  - patch: |
      - op: add
        path: /spec/serviceAccountName
        value: kustomize-controller
    target:
      kind: Kustomization
      name: "flux-system"
```

### Side Effects
## Onboard tenants with private repositories

You can configure Flux to connect to a tenant repository
using SSH or token-based authentication. The tenant credentials will be stored 
in the platform admin repository as a Kubernetes secret. 

### Encrypt Kubernetes secrets in Git

In order to store credentials safely in a Git repository, you can use Mozilla's
SOPS CLI to encrypt Kubernetes secrets with OpenPGP, Age or KMS.

Install [gnupg](https://www.gnupg.org/) and [sops](https://github.com/mozilla/sops):

```sh
brew install gnupg sops
```

Generate a GPG key for Flux without specifying a passphrase and retrieve the GPG key ID:

```console
$ gpg --full-generate-key
Email address: fluxcdbot@users.noreply.github.com

$ gpg --list-secret-keys fluxcdbot@users.noreply.github.com
sec   rsa3072 2020-09-06 [SC]
      1F3D1CED2F865F5E59CA564553241F147E7C5FA4
```

Create a Kubernetes secret in the `flux-system` namespace with the GPG private key:

```sh
gpg --export-secret-keys \
--armor 1F3D1CED2F865F5E59CA564553241F147E7C5FA4 |
kubectl create secret generic sops-gpg \
--namespace=flux-system \
--from-file=sops.asc=/dev/stdin
```

You should store the GPG private key in a safe place for disaster recovery,
in case you need to rebuild the cluster from scratch.
The GPG public key can be shared with the platform team, so anyone with 
write access to the platform repository can encrypt secrets.

### Git over SSH

Generate a Kubernetes secret with the SSH and known host keys:

```sh
flux -n apps create secret git dev-team-auth \
    --url=ssh://git@github.com/<org>/<dev-team> \
    --export > ./tenants/base/dev-team/auth.yaml
```

Print the SSH public key and add it as a read-only deploy key to the dev-team repository:

```sh
yq eval 'data."identity.pub"' git-auth.yaml | base64 --decode
```

### Git over HTTP/S

Generate a Kubernetes secret with basic auth credentials:

```sh
flux -n apps create secret git dev-team-auth \
    --url=https://github.com/<org>/<dev-team> \
    --username=$GITHUB_USERNAME \
    --password=$GITHUB_TOKEN \
    --export > ./tenants/base/dev-team/auth.yaml
```

The GitHub token must have read-only access to the dev-team repository.

### Configure Git authentication

Encrypt the `dev-team-auth` secret's data field with sops:

```sh
sops --encrypt \
    --pgp=1F3D1CED2F865F5E59CA564553241F147E7C5FA4 \
    --encrypted-regex '^(data|stringData)$' \
    --in-place ./tenants/base/dev-team/auth.yaml
```

Create the sync manifests for the tenant Git repository referencing the `git-auth` secret:

```sh
flux create source git dev-team \
    --namespace=apps \
    --url=https://github.com/<org>/<dev-team> \
    --branch=main \
    --secret-ref=dev-team-auth \
    --export > ./tenants/base/dev-team/sync.yaml

flux create kustomization dev-team \
    --namespace=apps \
    --service-account=dev-team \
    --source=GitRepository/dev-team \
    --path="./" \
    --export >> ./tenants/base/dev-team/sync.yaml
```

Create the base kustomization.yaml file:

```sh
cd ./tenants/base/dev-team/ && kustomize create --autodetect
```

Configure Flux to decrypt secrets using the `sops-gpg` key:

```yaml
flux create kustomization tenants \
  --depends-on=kyverno-policies \
  --source=flux-system \
  --path="./tenants/staging" \
  --prune=true \
  --interval=5m \
  --validation=client \
  --decryption-provider=sops \
  --decryption-secret=sops-gpg \
  --export > ./clusters/staging/tenants.yaml
```

With the above configuration, the Flux instance running on the staging cluster will:

* create the tenant namespace, service account and role binding
* decrypt the tenant Git credentials using the GPG private key
* create the tenant Git credentials Kubernetes secret in the tenant namespace
* clone the tenant repository using the supplied credentials
* apply the `./staging` directory from the tenant's repo using the tenant's service account

## Testing

Any change to the Kubernetes manifests or to the repository structure should be validated in CI before
a pull request is merged into the main branch and synced on the cluster.

This repository contains the following GitHub CI workflows:

* the [test](./.github/workflows/test.yaml) workflow validates the Kubernetes manifests
  and Kustomize overlays with [kubeconform](https://github.com/yannh/kubeconform)
* the [e2e](./.github/workflows/e2e.yaml) workflow starts a Kubernetes cluster in CI
  and tests the staging setup by running Flux in Kubernetes Kind


# Flux image Automation 
# Install Flux with the image automation components:

```sh
flux bootstrap github \
  --components-extra=image-reflector-controller,image-automation-controller \
  --owner=$GITHUB_USER \
  --repository=flux-image-updates \
  --branch=main \
  --path=clusters/my-cluster \
  --read-write-key \
  --personal
```

# Create an ImageRepository to tell Flux which container registry to scan for new tags:

```sh
flux create image repository podinfo \
--image=ghcr.io/stefanprodan/podinfo \
--interval=5m \
--export > ./clusters/my-cluster/podinfo-registry.yaml
```
# sample image repository
```sh
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImageRepository
metadata:
  name: podinfo
  namespace: flux-system
spec:
  image: ghcr.io/stefanprodan/podinfo
  interval: 5m
```

## For private images, you can create a Kubernetes secret in the same namespace as the ImageRepository with kubectl create secret docker-registry. Then you can configure Flux to use the credentials by referencing the Kubernetes secret in the ImageRepository:

```sh
kind: ImageRepository
spec:
  secretRef:
    name: regcred
```

## Create an ImagePolicy to tell Flux which semver range to use when filtering tags:
```sh
flux create image policy podinfo \
--image-ref=podinfo \
--select-semver=5.0.x \
--export > ./clusters/my-cluster/podinfo-policy.yaml
```
## The above command generates the following manifest:

```sh
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImagePolicy
metadata:
  name: podinfo
  namespace: flux-system
spec:
  imageRepositoryRef:
    name: podinfo
  policy:
    semver:
      range: 5.0.x
```
## After commiting and reconciling the kustomization, 
# Wait for Flux to fetch the image tag list from GitHub container registry:
```sh
flux get image repository podinfo
```

## Find which image tag matches the policy semver range with
```sh
flux get image policy podinfo
```
## Edit the podinfo-deployment.yaml and add a marker to tell Flux which policy to use when updating the container image:

```sh
spec:
  containers:
  - name: podinfod
    image: ghcr.io/stefanprodan/podinfo:5.0.0 # {"$imagepolicy": "flux-system:podinfo"}
```
## Create an ImageUpdateAutomation to tell Flux which Git repository to write image updates to:
```sh
flux create image update flux-system \
--interval=30m \
--git-repo-ref=flux-system \
--git-repo-path="./clusters/my-cluster" \
--checkout-branch=main \
--push-branch=main \
--author-name=fluxcdbot \
--author-email=fluxcdbot@users.noreply.github.com \
--commit-template="{{range .Changed.Changes}}{{print .OldValue}} -> {{println .NewValue}}{{end}}" \
--export > ./clusters/my-cluster/flux-system-automation.yaml
```
## The above command generates the following manifest:
```sh
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImageUpdateAutomation
metadata:
  name: flux-system
  namespace: flux-system
spec:
  interval: 30m
  sourceRef:
    kind: GitRepository
    name: flux-system
  git:
    checkout:
      ref:
        branch: main
    commit:
      author:
        email: fluxcdbot@users.noreply.github.com
        name: fluxcdbot
      messageTemplate: '{{range .Changed.Changes}}{{print .OldValue}} -> {{println .NewValue}}{{end}}'
    push:
      branch: main
  update:
    path: ./clusters/my-cluster
    strategy: Setters
```
flux reconcile kustomization flux-system --with-source

git pull && cat clusters/my-cluster/podinfo-deployment.yaml | grep "image:"

# Configure image update for custom resources 

# Flux Kustomization example:

```sh
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: podinfo
  namespace: default
spec:
  images:
    - name: ghcr.io/stefanprodan/podinfo
      newName: ghcr.io/stefanprodan/podinfo # {"$imagepolicy": "flux-system:podinfo:name"}
      newTag: 5.0.0 # {"$imagepolicy": "flux-system:podinfo:tag"}
```

# Kustomize config (kustomization.yaml) example:

```sh
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
images:
- name: ghcr.io/stefanprodan/podinfo
  newName: ghcr.io/stefanprodan/podinfo # {"$imagepolicy": "flux-system:podinfo:name"}
  newTag: 5.0.0 # {"$imagepolicy": "flux-system:podinfo:tag"}
```

# helm release 
```sh
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: podinfo
  namespace: default
spec:
  values:
    image:
      repository: ghcr.io/stefanprodan/podinfo # {"$imagepolicy": "flux-system:podinfo:name"}
      tag: 5.0.0  # {"$imagepolicy": "flux-system:podinfo:tag"}
```

# Push updates to a different branch
# with .spec.git.push.branch you can configure Flux to push the image updates to different branch than the one used for checkout. If the specified branch doesn’t exist, Flux will create it for you.

```sh
kind: ImageUpdateAutomation
metadata:
  name: flux-system
spec:  
  git:
    checkout:
      ref:
        branch: main
    push:
      branch: flux-image-updates
```

