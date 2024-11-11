### check
* The nginx_dep.yml is a deployment yml, it's manifest is directly from the flux-system dir, which was created during the bootstrap
* the gotk-sync.yml, make sure to change the interval to 1min0s
* **UNCOMMENT ALL *-sync.yml** FILES FOR MANIFESTS TO RUN WITH FLUX

## Available manifests
1. **testing-sync.yml**
- the testing-sync.yml is working with this repo https://github.com/Asiwomex/testing.git 
- this means that it is getting the deployment and service files from the **testing** repo on github

2. **uat-syn.yml**
- this is running deployments from the uat dir in the clusters dir

3. **dev-sync.yml**
- this is running deployments from the dev dir in the clusters dir