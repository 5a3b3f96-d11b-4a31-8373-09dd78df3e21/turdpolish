# ArgoCD
## Install
I install ArgoCD using a bootstrap setup. My de-facto bootstrap setup renders the Argo helm chart that I have set up in my charts folder using `helm template argocd --namespace argocd -f ./ARGO_VALUES_FILE --include-crds` and that runs as a GitHub Action on every PR so there's *always* a fresh Argo bootstrap that's the latest (not the *most* critical, but it helps avoid troubleshooting "why can't I upgrade from a version 3 years old to one I just got on this new Cluster?")
DO NOT install with Helm, because helm will own everything and it will brick your shit. Use the helm template then apply the YAML it produces to the cluster so nothing owns them and Argo can take it over.

## Files
### [argoproj.yaml](./argoproj.yaml)
This file is where I define the ArgoCD project. ArgoCD applications belong to projects. Argo ships with a default project (literally called default), but I don't recommend staying with it. Managing via project is a good idea.

### [argocd.yaml](./argocd.yaml)
This file is where I define the ArgoCD Application. You deploy this once ArgoCD is bootstrapped and set up. It will allow Argo to manage Argo (explained below).

## Example
This is a sample Argo app to deploy ArgoCD. It may seem counterproductive, but ArgoCD *managing ArgoCD* is actually a valid and awesome pattern. Your Argo will automatically update itself in line with the helm chart/the repo. This primarily relies on the way deployment rollouts happen where a rollout keeps pods alive until the newest set go live, allowing old Argo to deploy new one until it's healthy then new Argo being healthy kills old Argo.

TLDR; Bootstrap Argo which then deploys Argo. Do it every time.
