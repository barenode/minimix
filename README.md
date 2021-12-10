# minimix

## Prepare Kind cluster

```shell
kind create cluster --name minimix --config=./cluster-config.yaml
```

## Nginx

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

## ArgoCD

```shell
kubectl apply -f argocd/install.yaml -n argocd
kubectl apply -f argocd/argocd.ingress.yaml -n argocd
```

## Cleanup

```shell
kind delete cluster --name minimix 
```