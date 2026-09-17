# Setup

How to run this project yourself on a local cluster.

## Prerequisites

- A local Kubernetes cluster: [kind](https://kind.sigs.k8s.io/) or Docker Desktop's
  built-in Kubernetes both work. Examples below assume `kind`.
- [`flux`](https://fluxcd.io/flux/installation/) CLI
  - macOS: `brew install fluxcd/tap/flux`
  - Windows: `choco install flux`
  - Linux: `curl -s https://fluxcd.io/install.sh | sudo bash`
- `kubectl`
- A GitHub personal access token with repo permissions, if you're bootstrapping
  against your own fork

Verify your cluster and the CLI are ready:

```sh
kind create cluster --name flux-demo
kubectl cluster-info
flux check --pre
```

## Bootstrap Flux

```sh
export GITHUB_TOKEN=<your-token>
export GITHUB_USER=<your-username>

flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=flux_k8s \
  --path=clusters/kind \
  --token-auth \
  --personal
```

This installs Flux's controllers into the `flux-system` namespace, and commits
`clusters/kind/flux-system/gotk-components.yaml` + `gotk-sync.yaml` back into this repo
(already present here from the last bootstrap — running it again is idempotent).

Confirm the two app `Kustomization`s are picked up and synced:

```sh
flux get kustomizations --watch
kubectl get ns dev uat
kubectl get pods -n dev
kubectl get pods -n uat
```

## Talk to the app

```sh
kubectl port-forward -n dev svc/app-service 8080:80
# then open http://localhost:8080
```

Swap `-n dev` for `-n uat` to hit the other environment.

## Everyday Flux commands

```sh
# force an immediate reconcile instead of waiting for the 1m interval
flux reconcile kustomization todolist-dev --with-source
flux reconcile kustomization todolist-uat --with-source

# check controller health / logs
flux check
flux logs --follow
flux logs --follow --kind=Kustomization --name=todolist-dev

# stop Flux from reverting manual edits to a Kustomization (debugging only)
flux suspend kustomization todolist-dev
flux resume kustomization todolist-dev

# tear down
flux uninstall --silent
kind delete cluster --name flux-demo
```

## Encrypting secrets for real use

`apps/todolist/base/secret.yaml` and `db-secret.yaml` are plaintext-base64 Kubernetes
Secrets, which is fine for a local demo but not for anything real — base64 is encoding,
not encryption. For an actual deployment, encrypt them in place with
[SOPS](https://github.com/mozilla/sops) before committing:

```sh
brew install gnupg sops
gpg --full-generate-key
gpg --list-secret-keys <your-email>

kubectl create secret generic sops-gpg \
  --namespace=flux-system \
  --from-file=sops.asc=<(gpg --export-secret-keys --armor <key-id>)

sops --encrypt --pgp=<key-id> --encrypted-regex '^(data|stringData)$' \
  --in-place apps/todolist/base/secret.yaml
```

Then add `decryption.provider: sops` / `decryption.secretRef` to the Flux
`Kustomization`s in `clusters/kind/apps-dev.yaml` and `apps-uat.yaml` so
kustomize-controller decrypts them at apply time.
