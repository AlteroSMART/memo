# Ingress-Nginx

## Manual
### Install
```console
## Install Ingress-Nginx
$ kubectl apply -f https://raw.githubusercontent.com/AlteroSMART/memo/master/ingress-nginx/nginx-mandatory.yaml
# HTTPS configuration (TLS 1.2-1.3)
$ kubectl apply -f https://raw.githubusercontent.com/AlteroSMART/memo/master/ingress-nginx/nginx-configuration.yaml

# add HTTP(S) LoadBalancer
$ kubectl apply -f https://raw.githubusercontent.com/AlteroSMART/memo/master/ingress-nginx/nginx-lb-web.yaml
```
### Uninstall
```console
$ kubectl delete namespace ingress-nginx
```

## HELM-chart
### Install
[https://kubernetes.github.io/ingress-nginx/deploy/#using-helm](https://kubernetes.github.io/ingress-nginx/deploy/#using-helm)
```console
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
$ helm repo update
$ helm install nginx ingress-nginx/ingress-nginx --version 3.3.0 -n ingress-nginx
NAME: nginx
LAST DEPLOYED: Wed Sep 30 01:53:32 2020
NAMESPACE: ingress-nginx
STATUS: deployed
```
### Uninstall
```console
$ helm uninstall -n ingress-nginx nginx 
release "nginx" uninstalled
```

## Fix 'failed calling webhook "validate.nginx.ingress.kubernetes.io"'
```console
$ kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
## and reinstall Ingress-Nginx
```
