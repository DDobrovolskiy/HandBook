#### Описание задачи
Необходимо настроить безопасное Mutual TLS соединение вашего прикладного контейнера, который находится под управлением Istio Service Mesh, c внешним сервисом.

В результате решения задачи, необходимо убедиться, что запросы от контейнера успешно достигают внешнего ресурса (в ответе приходит http статус 200)

Способ решения задачи
Существует, как минимум, 2 способа решения задачи:

с использованием Egress Gateway (шлюз для исходящих соедиений)
без использования Egress Gateway
Первый способ используется в случае, если согласно политикам безопасности соединение с внешними сервисами обязательно контролируется при помощи Egress Gateway.

Второй способ может исползоваться в случае, если требования маршутизации трафика на внешние сервисы через единый шлюз нет.

В этом сценарии предлагается рассмотреть второй способ без использования Egress Gateway.

1. Создать корневой сертификат и приватный ключ
``` shell
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -subj '/O=example Inc./CN=example.com' -keyout example.com.key -out example.com.crt
```
2. Создать сертификат для внешнего сервиса
``` shell
openssl req -out my-nginx.mesh-external.svc.cluster.local.csr -newkey rsa:2048 -nodes -keyout my-nginx.mesh-external.svc.cluster.local.key -subj "/CN=my-nginx.mesh-external.svc.cluster.local/O=some organization"
openssl x509 -req -sha256 -days 365 -CA example.com.crt -CAkey example.com.key -set_serial 0 -in my-nginx.mesh-external.svc.cluster.local.csr -out my-nginx.mesh-external.svc.cluster.local.crt
```
3. Создать сертификат для клиента
``` shell
openssl req -out client.example.com.csr -newkey rsa:2048 -nodes -keyout client.example.com.key -subj "/CN=client.example.com/O=client organization"
openssl x509 -req -sha256 -days 365 -CA example.com.crt -CAkey example.com.key -set_serial 1 -in client.example.com.csr -out client.example.com.crt
```

``` shell
kubectl create -n mesh-external \
  secret generic nginx-ca-certs \
  --from-file=example.com.crt
``` 
