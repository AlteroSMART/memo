# Monitoring
Install BEST monitoring-stack "Prometheus + Grafana + Alter Manager" by Prometheus Operator (HELM-chart). 

Visit [github.com/coreos/prometheus-operator](https://github.com/coreos/prometheus-operator) for instructions on how
to create & configure Alertmanager and Prometheus instances using the Operator.

## Install

### Configure chart-values file
Download  `prometheus-operator-values.yaml` and change `grafana.adminPassword` value !
```console
$ wget https://raw.githubusercontent.com/AlteroSMART/memo/master/monitoring/prometheus-operator-values.yaml \
  -O config-monitoring.local.yaml
$ nano config-monitoring.local.yaml
```

### Install by HELM
```console
$ kubectl create namespace monitoring

# PV
$ kubectl apply -f pv-mon-prometheus.yml

$ helm repo add stable https://kubernetes-charts.storage.googleapis.com/
$ helm install mon stable/prometheus-operator -n monitoring \
  --values config-monitoring.local.yaml  --version 8.13.14
```

### Configure Ingress-Nginx
```console
$ kubectl apply -f ingress-monitoring.yaml
```

### Add service-monitors
```console
$ kubectl apply -f servicemonitor-clickhouse.yaml
$ kubectl apply -f servicemonitor-rabbitmq.yaml
$ kubectl apply -f servicemonitor-redis.yaml
```

## Uninstall
```console
$ helm delete mon -n monitoring

# Delete CRD (with settings)
$ kubectl delete crd prometheuses.monitoring.coreos.com ;
$ kubectl delete crd prometheusrules.monitoring.coreos.com ;
$ kubectl delete crd servicemonitors.monitoring.coreos.com ;
$ kubectl delete crd podmonitors.monitoring.coreos.com ;
$ kubectl delete crd alertmanagers.monitoring.coreos.com ;
$ kubectl delete crd thanosrulers.monitoring.coreos.com ;

# Delete PVC/PV
$ kubectl delete pvc -n monitoring \
  prometheus-mon-prometheus-operator-prometheus-db-prometheus-mon-prometheus-operator-prometheus-0
$ kubectl delete -f pv-mon-prometheus.yml

# (optional) Purge Prometheus data
$ sudo rm -rf /srv/mon-prometheus/*

$ kubectl delete namespace monitoring
```
