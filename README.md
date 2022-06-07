# ArgoCD three in one
Manage continious delivery for applicatons, third party services and infrastructure with ArgoCD.

## Setup

### Create a local cluster

create new kind cluster named `argocd-3-in-1`
```bash
kind create cluster --name argocd-3-in-1
```

### Install ArgoCD

- install helm chart

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

- port-forward

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

- Get admin secret 

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

## Example application - Nginx server

Install a nginx server :

- `cp -r main/applications/nginx demo/applications`

- `cp main/argo/nginx-app.yaml demo/argo/nginx-app.yaml`  

## Applications - Mircolearning project

## Third party services

## Infrastructure