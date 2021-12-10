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

[original source](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)

```shell
kubectl create ns argocd
kubectl apply -f argocd/install.yaml -n argocd
kubectl apply -f argocd/argocd.ingress.yaml -n argocd
```

## RabbitMQ

[original source](https://github.com/rabbitmq/cluster-operator/releases/latest/download/cluster-operator.yml)




## Cleanup

```shell
kind delete cluster --name minimix 
```