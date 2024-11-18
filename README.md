# Kubernetes_FluxCD
**PAY ATTENTION TO THE OTHER READMEs**

- Use choco install flux AND DO THE FOLLOWING IN GITBASH
- create github token and allow all permissions
- export GITHUB_TOKEN='GITHUBTOKEN'
- export GITHUB_USER='USERNAME'
- use echo $GITHUB_USER to confirm
- create a github repo and clone, from the codes below, the repo name should be "flux_k8s"
- start your cluster, make sure a cluster is running
- execute the codes **(Bootsrap)** below, in bash. this should be done in the main working dir, like flux_k8s here

```bash
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=Kubernetes_FluxCD \
  --path=clusters/manifests \
  --token-auth \
  --personal
```

## **Below is the explanation to the Bootsrap codes**

This command is used to **initialize Flux CD in a GitOps setup** by bootstrapping it with a GitHub repository. It automates the setup process by creating necessary Flux resources in the specified repository, so Flux can begin managing Kubernetes resources from a Git repository.

### Command Components

1. **`flux bootstrap github`**:
   - This tells Flux to bootstrap (initialize) a GitOps environment using a **GitHub** repository. `bootstrap` is the command, and `github` specifies the provider (GitHub in this case).

2. **`--owner=$GITHUB_USER`**:
   - Specifies the **GitHub account (owner)** of the repository. `$GITHUB_USER` is a variable that should hold your GitHub username or organization name.
   - For example, if `$GITHUB_USER` is set to "asiwomex," Flux will use that account.

3. **`--repository=flux_k8s`**:
   - This is the **name of the GitHub repository** where Flux will be set up. In this case, Flux will use the repository named `flux_k8s`.

4. **`--path=clusters/manifests`**:
   - Specifies the **directory path within the repository** where Flux will look for Kubernetes manifests (YAML files).
   - Flux will use this path to manage and synchronize the resources within `clusters/manifests` to the Kubernetes cluster.
   - This path could be a folder where you organize YAML files for different clusters or environments (e.g., `prod`, `staging`).

5. **`--token-auth`**:
   - Enables **token-based authentication** for the GitHub API, necessary for private repositories or when Flux requires access beyond what public permissions allow.

6. **`--personal`**:
   - Specifies that the repository is a **personal repository** (not an organization-owned repository).
   - This flag tells Flux that it’s dealing with a personal GitHub account.

### What Happens When You Run This Command
When you execute this command:
- **Flux connects to GitHub**: It uses the provided GitHub credentials (via token) to authenticate.
- **Creates required manifests**: Flux creates Kubernetes YAML files and other resources needed to install Flux components into your cluster. These include Custom Resource Definitions (CRDs) and controllers for managing GitOps resources.
- **Commits Flux configuration to the repository**: It pushes Flux’s own configuration files to the specified `path` within the repository. These files instruct Flux on where to look for Kubernetes manifests to apply to your cluster.
- **Syncs Kubernetes with GitHub repository**: Once bootstrapped, Flux continuously syncs your Kubernetes cluster with the repository, applying any changes made to manifests within `clusters/manifests`.

### Purpose
The command sets up Flux to manage your Kubernetes cluster by reading and applying configurations directly from the specified GitHub repository and path. Any changes to these files in GitHub will automatically sync with your cluster, making it easy to manage and deploy configurations consistently.


8. use `kubectl get ns` to get namespaces
9. you should see **"flux-system"** as a new namespace

- use flux uninstall to uninstall and bootsrap again after staying off for sometime

- flux reconcile ks flux-system --with-source - to speed up without the interval
