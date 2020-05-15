# About
Инструкция по развертыванию логирования в небольшом кластере K3S/K8S на основе EFK-стека.    
Развертывается ElasticSearch (одно-серверный вариант) + Fluentd в пространстве имен `kube-logs`.  
Данные ElasticSearch хранятся в папке `/srv/kube-logs-es` на Ноде с меткой `kube-logs-storage=true`.

## K8S logging stack
Микросервисы в Kubernetes должны отправлять все сообщения ТОЛЬКО в stdout/stderr (согласно, [12factor.net/ru/](https://12factor.net/ru/)),
а ELK/EFK сами должны их собирать и складывать в Хранилище Логов.

## Установка

### Ставим метку на Ноде c Персистентным Хранилищем
```bash
## Получить список Нод
kubectl get nodes
NAME   STATUS   ROLES    AGE   VERSION
iec    Ready    master   73d   v1.18.2+k3s1
iec2   Ready    <none>   5d    v1.18.2+k3s1

## Добавить метку
kubectl label nodes iec kube-logs-storage=true
## Убрать метку
kubectl label nodes iec kube-logs-storage-
```

### Разворачиваем ElasticSearch
```bash
## (опционально) Удалить ES-dir (от предыдущих данных)
iec# rm -rf /srv/kube-logs-es

## Deploy ElasticSearch
kubectl apply -f  https://github.com/AlteroSMART/memo/raw/master/kube-logs/10_elasticsearch.yaml
```
После развертывания ES, нужно дождаться готовности (status = **green**):
```bash
kubectl exec -it -n kube-logs elasticsearch-0 -- curl http://elasticsearch:9200/_cat/health
1570445371 10:49:31 k8s-logs green 1 1 1 1 0 0 0 0 - 100.0%
```
Инициируем лог с помощью CURL-запроса
```bash
# Prepare ES for Logging
INIT_LOG="$(curl -sfL https://github.com/AlteroSMART/memo/raw/master/kube-logs/11_es_curl.json)"
kubectl exec -it -n kube-logs elasticsearch-0 -- \
  curl -H "Content-Type: application/json" -d "$INIT_LOG" -X PUT http://elasticsearch:9200/_template/au-logs
```
Если всё прошло хорошо, вы увидете ответ `{"acknowledged":true}`. 

### Разворачиваем Fluentd
```bash
## Deploy Fluentd
kubectl apply -f  https://github.com/AlteroSMART/memo/raw/master/kube-logs/20_fluentd-es-configmap.yaml
kubectl apply -f  https://github.com/AlteroSMART/memo/raw/master/kube-logs/21_fluentd-es-ds.yaml
```
### (optional) Разворачиваем Kibana
```bash
## (optional) Deploy Kibana
kubectl apply -f  https://github.com/AlteroSMART/memo/raw/master/kube-logs/30_kibana.yaml
```
После развертывания Kibana, можем пробросить порт для работы с Кибаной с локальной машины
```bash
kubectl get pods -n kube-logs | grep kibana
kibana-85bbb684bf-6qbxp   1/1     Running   0          4m10s

kubectl port-forward -n kube-logs kibana-85bbb684bf-6qbxp 8000:5601
Forwarding from 127.0.0.1:8000 -> 5601
Forwarding from [::1]:8000 -> 5601
...
```
Чтобы завершить проброс-порта на локальную Машину - в консоли нажмите **CTRL+C**

#### Настраиваем Kibana (когда появятся любые логи)
* Включаем форвардинг-порта на локальную машину (см. Выше `kubectl port-forward ...` ) 
* Переходим в веб-интерфейс Kibana по адресу [http://localhost:8000/kibana/](http://localhost:8000/kibana/)
* Жмем на кнопку **Discover**
* В поле вводим шаблон для Индекса: `logstash-*` и нажимаем **"Next step"**
* Выбираем в фильтре поле **@timestamp** и нажимаем **"Create index pattern"**

Для проверки логов - опять жмем на кнопку **Discover** 
