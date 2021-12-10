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

```shell
kubectl apply -f rabbitmq/cluster-operator.yml
kubectl apply -f rabbitmq/rabbitmq-cluster.yaml
kubectl apply -f rabbitmq/rabbitmq-ingress.yaml
```

username/password (base64 encoded)
```shell
kubectl get secret rabbitmq-default-user -o jsonpath='{.data.username}'  -n rabbitmq
kubectl get secret rabbitmq-default-user -o jsonpath='{.data.password}'  -n rabbitmq
```
output (base64 decoded):

default_user_D3Iwgkfj1qy8Fdseip9
UM944fqb265dM4GtGFPPCI9rZZPIUosD

## Cleanup

```shell
kind delete cluster --name minimix 
```