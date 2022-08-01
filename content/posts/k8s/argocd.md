+++ 
date = 2022-07-31T10:00:00+02:00
title = "Install ArgoCD"
slug = "" 
tags = "k8s, k3s, GitOps"
categories = []
thumbnail = "/img/argocd-logo.png"
description = "Set up a ArgoCD GitOps flow"
+++

## Quick install


```sh
mkdir -p ~/git/github.com/argoproj
cd ~/git/github.com/argoproj
kubectl create namespace argocd
git clone https://github.com/argoproj/argo-cd.git
kubectl apply --namespace argocd -f ~/git/github.com/argoproj/argo-cd/manifests/install.yaml
```

## Access ArgoCD

To access ArgoCD you port forward as below. The installation process automatically creates an ```admin``` user. You'll see the generated password below.

```sh
echo "argocd password: $(kubectl -n argocd get secret argocd-initial-admin-secret  -ojson | jq -r .data.password| base64 -d)"
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
