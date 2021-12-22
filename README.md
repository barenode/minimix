# minimix


<table>
    <tr>
        <th>name</th>
        <th>type</th>
        <th>version</th>
    </tr>
    <tr>
        <td>nginx</td>
        <td>ingress</td>
        <td>1.1.0</td>
    </tr>
</table>

## Prepare Kind cluster

```shell
kind create cluster --name minimix --config=./cluster-config.yaml
```

## Nginx

[nging ingress operator original source](https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml)

Nging ingress with empty TCP config map (proxyfied ports) configured

```shell
kubectl apply -f nginx/kind/nginx-deploy-kind.yaml
```

## ArgoCD

[original source](https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)

```shell
kubectl create ns argocd
kubectl apply -f argocd/install.yaml -n argocd
kubectl apply -f argocd/argocd.ingress.yaml -n argocd
```

## Cert Manager

```shell
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.3.1/cert-manager.yaml
```

## RabbitMQ

[cluster operator original source](https://github.com/rabbitmq/cluster-operator/releases/latest/download/cluster-operator.yml)

[topology operator original source](https://github.com/rabbitmq/messaging-topology-operator/releases/latest/download/messaging-topology-operator-with-certmanager.yaml)

```shell
kubectl apply -f rabbitmq/cluster-operator.yml
kubectl apply -f rabbitmq/messaging-topology-operator-with-certmanager.yaml
kubectl apply -f rabbitmq/rabbitmq-cluster.yaml
kubectl apply -f rabbitmq/rabbitmq-ingress.yaml
```

username/password (base64 encoded)

```shell
kubectl get secret rabbitmq-default-user -o jsonpath='{.data.username}'  -n rabbitmq
kubectl get secret rabbitmq-default-user -o jsonpath='{.data.password}'  -n rabbitmq
```

output (base64 decoded):

default_user_VK5bQ_W9cwMVcunrOsT
6XnSgFNuvAsK0MxcZYR9jxrD2F0u0Oiw


### Expose RabbitMQ port 5672

[guide official](https://kubernetes.github.io/ingress-nginx/user-guide/exposing-tcp-udp-services/)
[guide SO](https://stackoverflow.com/questions/61430311/exposing-multiple-tcp-udp-services-using-a-single-loadbalancer-on-k8s)

patch nginx ingress

```shell
kubectl patch deployment ingress-nginx-controller -n ingress-nginx --patch-file "rabbitmq/nginx-ingress-controller-patch.yaml"
kubectl patch service ingress-nginx-controller -n ingress-nginx --patch-file "rabbitmq/nginx-ingress-svc-controller-patch.yaml"
kubectl patch configmap tcp-services -n ingress-nginx --patch "{\"data\":{\"5672\":\"rabbitmq/rabbitmq:5672\"}}"
```

## Inventory

```shell
kubectl create ns inventory
kubectl apply -f inventory/inventory-argocd.yaml
kubectl apply -f inventory/inventory-client-argocd.yaml
```

## Cleanup

```shell
kind delete cluster --name minimix
```
