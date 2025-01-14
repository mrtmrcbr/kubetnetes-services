# GitOps Demo with argocd and helmchart

This repository include base kubernetes application deployment.

This project follows basic gitops principles, 

- In argocd directory, there are helm values and configs for basic ArgoCD setup. In values file I enabled notification controller and then i added slack-token secret and config map for customize the slack messaged. Also there is a apps directory for demo-app application. It is automate demo-app deployments after failed or succeded events it send slack notification and it scan docker-hub image registry. If CI pipeline pushs a new image to hub.docker.io argocd create new deployment from newly pushed image. In this part I havent implemented git-write-back functionality.
- In demo-app-chart directory there is a helmChart for demo-app, If someone updated this helmChart ArgoCD application detects the change and it applies to kubernetes cluster which is configured inside of ArgoCD application.
- In demo-app directory there is a basic kubernetes deployment and service resources. There is no ArgoCD automation for this resources.

## Setup

```sh

kubectl create namespace argocd

helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

helm install argocd -n argocd -f argocd-values.yaml argo/argo-cd
helm install updater -n argocd argo/argocd-image-updater

kubectl apply -f argocd/argocd-secrets.yaml
kubectl apply -f argocd/argocd-cm.yaml
kubectl apply -f argocd/apps/myapp.yaml

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
kubectl port-forward service/argocd-server -n argocd 8080:443
````

### Improvements

- slack-token is not sealed and i removed my own token. If we add some sealing mechanism slack-token will be more secured.
- this argocd setup works well but is not accessible very easyly. In this setup we have to create port forwarding. If we want to deploy this ArgoCD to cloud provided kubernetes cluster it may good add a loadbalancer.

