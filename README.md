# Gitops 3 in 1 with ArgoCD & Crossplane

Manage continious delivery for applicatons, third party services and infrastructure with ArgoCD and Crossplane.

## Project structure

- `Main` directory : The directory that contains all our manifests used in the different steps.  
- `Demo` directory : This directory is used to simulate the configuration repository. The CI is simulated to update this directory that being synched. 

## Setup

### Create a local cluster

$ create new kind cluster named `argocd-3-in-1`
```kind create cluster --name argocd-3-in-1```

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

- Port-forward Argo-CD user interface

```kubectl port-forward svc/argocd-server -n argocd 8080:443```

- Get admin secret 

```kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo```

### Install crossplane

```bash
kubectl create namespace crossplane-system

helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update

helm install crossplane --namespace crossplane-system crossplane-stable/crossplane
```

#### Create GCP provider secret 

* Generate a `crossplane-gcp-provider-key` using `gcloud` cli. 
* Export credentials : `export BASE64ENCODED_GCP_PROVIDER_CREDS=$(base64 crossplane-gcp-provider-key.json | tr -d "\n")`

* Create a the gcp provider secret :

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: gcp-account-creds
  namespace: crossplane-system
type: Opaque
data:
  credentials: ${BASE64ENCODED_GCP_PROVIDER_CREDS}
```

* Activate billing on project

- For futher information : https://crossplane.io/docs/v1.8/cloud-providers/gcp/gcp-provider.html 

## Step 1 - Example application - Nginx server

Install a nginx server :

- CI : `bash main/ci/ci-example.sh`

- Create nginx-app argo application : `kubectl apply -f demo/argo/nginx-app.yaml`

- Commit and push

- Check out the application : `kubectl port-forward svc/ngin 8081:80 -n example`
  
- Change nginx version to `1.22.0` in demo application manifest

## Applications - Bookinfo project

- CI : `bash main/ci/ci-bookinfo.sh`

- Create app of apps argo application : `kubectl apply -f demo/argo/apps.yaml`

- Commit and push

- Checkout the application :

- `kubectl port-forward svc/productpage 9080:9080 -n bookinfo`

- <http://localhost:9080/productpage>

- Delete deployment `review-v1`

## Third party services

### Prometheus

- CI : `bash main/ci/ci-bookinfo.sh`

- Create argo application for third party services : `kubectl apply -f demo/argo/third-party.yaml`

- Commit and push

## Infrastructure

### Create a SQLInstance database GCP resource

- CI : `bash main/ci/ci-infra.sh`

- Create argo application for third party services : `kubectl apply -f demo/argo/infra.yaml`

- Commit and push

### Tools

- Kind
- kubectl
- crossplane-cli
- gloud-cli
