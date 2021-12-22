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

### Kind

```shell
kubectl apply -f nginx/kind/nginx-deploy-kind.yaml
```

### Bare metal

Nginx ingress pod is bidnded directly to host network, [as described here](https://kubernetes.github.io/ingress-nginx/deploy/baremetal/).
So nginx ingress service is efectivelly bypassed and is not needed anymore.

```shell
kubectl apply -f nginx/baremetal/nginx-deploy-baremetal.yaml
```

## Monitoring

### Install kube-prometheus monitoring stack

See [kube-prometheus](https://github.com/prometheus-operator/kube-prometheus)

```shell
kubectl apply -f monitoring/kube-prometheus-release-0.9/manifests/setup
kubectl apply -f monitoring/kube-prometheus-release-0.9/manifests/
```

### Ingress

Add dns mapping for host (for windows located at c:/windows/system32/drivers/etc/hosts)

```
127.0.0.1 grafana.minimix.com
127.0.0.1 prometheus.minimix.com
```

Create ingress
```shell
kubectl apply -f monitoring/ingress.yaml
```
Access grafana console at http://grafana.minimix.com (admin:admin). Access prometheus console at http://prometheus.minimix.com

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

### Monitoring

[prometheus rules](https://www.rabbitmq.com/kubernetes/operator/operator-monitoring.html#config-perm)

```shell
kubectl apply -f rabbitmq/prometheus-roles.yaml
kubectl apply --filename https://raw.githubusercontent.com/rabbitmq/cluster-operator/main/observability/prometheus/monitors/rabbitmq-servicemonitor.yml
kubectl apply --filename https://raw.githubusercontent.com/rabbitmq/cluster-operator/main/observability/prometheus/monitors/rabbitmq-cluster-operator-podmonitor.yml
```
[grafana guide](https://www.rabbitmq.com/prometheus.html#grafana-configuration)
[official dasboards](https://grafana.com/orgs/rabbitmq)



## Inventory

```shell
kubectl create ns inventory
kubectl apply -f inventory/inventory-argocd.yaml
kubectl apply -f inventory/inventory-client-argocd.yaml
```

## Monitoring

https://www.tutorialworks.com/spring-boot-prometheus-micrometer/

## Cleanup

```shell
kind delete cluster --name minimix
```
