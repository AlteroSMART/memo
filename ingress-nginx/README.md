# Ingress-Nginx

## Install
```bash
kubectl apply -f nginx-mandatory.yaml

## HTTPS configuration
kubectl apply -f nginx-configuration.yaml

## HTTP(S) LoadBalancer
kubectl apply -f nginx-lb-web.yaml

```

## Uninstall
```bash
kubectl delete namespace ingress-nginx
``
