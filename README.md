# flux_k8s
**PAY ATTENTION TO THE OTHER READMEs**

- Use choco install flux AND DO THE FOLLOWING IN GITBASH
- create github token and allow all permissions
- export GITHUB_TOKEN='GITHUBTOKEN'
- export GITHUB_USER='USERNAME'
- use echo $GITHUB_USER to confirm
- create a github repo and clone, from the codes below, the repo name should be "flux_k8s"
- start your cluster, make sure a cluster is running
- execute the codes below, in bash
```bash
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=flux_k8s \
  --path=clusters/manifests \
  --token-auth \
  --personal
```

8. use `kubectl get ns` to get namespaces
9. you should see **"flux-system"** as a new namespace

- use flux uninstall to uninstall and bootsrap again after staying off for sometime


- flux reconcile ks flux-system --with-source - to speed up without the interval