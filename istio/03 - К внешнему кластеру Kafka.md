03 - К внешнему кластеру Kafka
#### Подготовительные работы
Для проверки соединения развернем образ с клиентом Kafka
``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    name: kafka-client
    marker: practice
  name: kafka-client
spec:
  selector:
    matchLabels:
      name: kafka-client
  template:
    metadata:
      annotations:
        sidecar.istio.io/inject: 'true'
      labels:
        name: kafka-client
    spec:
      containers:
        - image: >-
            nexus.local:5000/bitnami/kafka:3.2.1-debian-11-r16
          command:
            - sh
            - '-c'
            - while true; do sleep 30; done
          name: kafka-client
          env:
            - name: KAFKA_ADDRESS
              value: 'kafka-4a20753d-deb2-41cb-b3eb-ed206cb1a6ea.apps.sbc-okd.pcbltools.ru:9092'
          resources:
            limits:
              cpu: 100m
              memory: 128Mi
            requests:
              cpu: 100m
              memory: 128Mi
```

``` shell
oc apply -f kafka-client.yml
``` 
Дождемся, пока под с клиентом запустится
``` shell
oc get pods -l name=kafka-client
``` 
Проверим подключение к брокеру Kafka, запустим команду для получения версий API брокера из workload контейнера пода клиента

Проверка может выполняться в течение одной минуты, это не является ошибкой
``` shell
oc exec $(oc get pods -o name -l name=kafka-client | head -n 1) -- bash -c 'kafka-broker-api-versions.sh --bootstrap-server $KAFKA_ADDRESS'
``` 
В логах istio-proxy видим ошибку с кодом UH
``` shell
oc logs $(oc get pods -o name -l name=kafka-client | head -n 1) -c istio-proxy
``` 
Изучите формат логов Envoy Proxy и описание кодов ошибок на странице
``` shell
UH: No healthy upstream hosts in upstream cluster in addition to 503 response code.

2022-10-25T20:37:07.830Z] "- - -" 0 UH - - "-" 0 0 0 - "-" "-" "-" "-" "-" - - 178.170.196.65:9092 10.128.3.43:56440 - -
```

#### Настройка прямого соединения с помощью ServiceEntry - простой вариант

Для объявления возможности подключения к внешнему ресурсу используется объект Istio Service Entry

Изучите шаблон ServiceEntry для настройки TCP-соединения с внешним узлом в файле, обратите внимание на комментарии
``` yaml
apiVersion: template.openshift.io/v1
kind: Template
metadata:
  name: out-tcp-template
objects:
  - apiVersion: networking.istio.io/v1alpha3
    kind: ServiceEntry
    metadata:
      name: ${PREFIX}-se
      labels:
        marker: practice
    spec:
      # Обязательно указывать во всех конфигах Istio. Ограничивает область применения конфига нашим проектом
      # Если не указывать exportTo возможно влияние конфига на другие проекты и сам Service Mesh
      exportTo:
        - .
      # Имя внешнего хоста для подключения
      hosts:
        - ${HOST}
      # IP адрес для STATIC
      addresses:
        - ${IP}
      endpoints:
        - address: ${IP}
      location: MESH_EXTERNAL
      # Перечень портов
      ports:
        - name: tcp-${PORT}
          number: ${{PORT}}
          # Перечень протоколов: HTTP|HTTPS|GRPC|HTTP2|MONGO|TCP|TLS. Для Kafka указываем TCP
          protocol: TCP
      resolution: STATIC
parameters:
  - name: HOST
    required: true
  - name: IP
    required: true
  - name: PORT
    required: true
  - name: PREFIX
    required: true
  - name: EGRESS_NAME
    required: false
``` 
Применим настройки ServiceEntry с параметрами из файла

``` env
HOST=kafka-4a20753d-deb2-41cb-b3eb-ed206cb1a6ea.apps.sbc-okd.pcbltools.ru
PORT=9092
PREFIX=kafka
EGRESS_NAME=ci00000000-test-egress
IP=172.30.213.192
```

``` shell
oc process -f easy.yml --param-file connection.env -o yaml > conf.yml
oc apply -f conf.yml
``` 
Проверим подключение к брокеру Kafka, соединение успешно
``` shell
oc exec $(oc get pods -o name -l name=kafka-client | head -n 1) -- bash -c 'kafka-broker-api-versions.sh --bootstrap-server $KAFKA_ADDRESS'
``` 
В логах контейнера istio-proxy видим прямое обращение к брокеру Kafka
``` shell
oc logs $(oc get pods -o name -l name=kafka-client | head -n 1) -c istio-proxy
[2022-10-18T20:24:03.639Z] "- - -" 0 - - - "-" 85 585 1100 - "-" "-" "-" "-" "172.30.247.31:9092" outbound|9092||kafka.apps.sbc-okd.pcbltools.ru 10.128.2.33:54464 172.30.247.31:9092 10.128.2.33:54460 - -
``` 
Удаляем созданные конфиги
``` shell
oc delete serviceentry -l marker=practice
```

#### Настройка Egress Gateway
В соответствии с требованями УЭК весь исходящий из проекта АС трафик должен проходить через Egress Gateway в проекте АС. Для запуска своего Egress Gateway нам потребуются:

Имя проекта
Имя Control Plane Istio, к которой подключен проект
Название сервиса Egress. Название должно содержать КЭ АС, это требование сопровождения Istio
Перечень портов, используемых для организации трафика через Egress Gateway. Все порты должны быть объявлены в Service Egress Gateway
Получим имя проекта
``` shell
oc project -q
``` 
Получим имя Control Plane
``` shell
oc describe project $(oc project -q) | grep member-of | head -n 1 | cut -d '=' -f2
``` 
Заполните полученные параметры в файле
``` env
PROJECT_NAME=sbercode-19a6d4ea-95da-4417-ad41-20d438527117-work
CONTROL_PLANE=istio-system
EGRESS_NAME=ci00000000-test-egress
```
Создадим Deployment Egress Gateway
``` shell
oc process -f egress-template.yml --param-file egress-params.env -o yaml > conf.yml
oc apply -f conf.yml
``` 
В логах пода Egress Gateway необходимо дождаться сообщения
Envoy Proxy is ready
``` shell
oc logs $(oc get pods -o name | grep egress | head -n 1)
```

#### Настройка соединения через Egress Gateway - правильный вариант
Изучите шаблон для настройки TCP-соединения с внешним узлом через Egress Gateway в файле, обратите внимание на комментарии
``` yaml
apiVersion: template.openshift.io/v1
kind: Template
metadata:
  name: egressgateway-template
objects:
  # ServiceEntry аналогичен предыдущему варианту
  - apiVersion: networking.istio.io/v1alpha3
    kind: ServiceEntry
    metadata:
      name: ${PREFIX}-se
      labels:
        marker: practice
    spec:
      exportTo:
        - .
      hosts:
        - ${HOST}
      addresses:
        - ${IP}
      endpoints:
        - address: ${IP}
      location: MESH_EXTERNAL
      ports:
        - name: tcp-${PORT}
          number: ${{PORT}}
          protocol: TCP
      resolution: STATIC
  # Gateway для обработки трафика, входящего в Egress Gateway
  - apiVersion: networking.istio.io/v1alpha3
    kind: Gateway
    metadata:
      name: ${PREFIX}-gw
      labels:
        marker: practice
    spec:
      selector:
        istio: ${EGRESS_NAME}
      servers:
        - hosts:
            - ${HOST}
          port:
            name: tcp-3000
            number: 3000
            protocol: TCP
  # VirtualService, реализующий:
  # - перенаправление обращений к внешнему узлу на Egress Gateway
  # - перенаправление трафика из Egress Gateway на внешний узел
  - apiVersion: networking.istio.io/v1alpha3
    kind: VirtualService
    metadata:
      name: ${PREFIX}-vs
      labels:
        marker: practice
    spec:
      exportTo:
        - .
      gateways:
        - ${PREFIX}-gw
        - mesh
      hosts:
        - ${HOST}
      # Этот пункт важен. Для HTTP отдельный конфиг
      tcp:
        # Приложение -> Egress
        - match:
            - gateways:
                - mesh
              # Порт внешнего узла
              port: ${{PORT}}
          route:
            - destination:
                host: ${EGRESS_NAME}
                # Порт Egress Gateway
                port:
                  number: 3000
        # Egress -> внешний узел
        - match:
            - gateways:
                - ${PREFIX}-gw
              # Порт Egress Gateway
              port: 3000
          route:
            - destination:
                host: ${HOST}
                # Порт внешнего узла
                port:
                  number: ${{PORT}}
parameters:
  - name: HOST
    required: true
  - name: IP
    required: true
  - name: PORT
    required: true
  - name: PREFIX
    required: true
  - name: EGRESS_NAME
    required: true

``` 
Документация по объектам Istio, использующимся для настройки маршрутизации:

Gateway
Virtual Service
Настроим маршрутизацию исходящего трафика через Egress Gateway с параметрами из файла
``` env
HOST=kafka-4a20753d-deb2-41cb-b3eb-ed206cb1a6ea.apps.sbc-okd.pcbltools.ru
PORT=9092
PREFIX=kafka
EGRESS_NAME=ci00000000-test-egress
IP=172.30.213.192
```

``` shell
oc process -f correct.yml --param-file connection.env -o yaml > conf.yml
oc apply -f conf.yml
``` 
Повторим проверку подключения к Kafka, соединение успешно
``` shell
oc exec $(oc get pods -o name -l name=kafka-client | head -n 1) -- bash -c 'kafka-broker-api-versions.sh --bootstrap-server $KAFKA_ADDRESS'
``` 
В логах istio-proxy, что запрос к внешнему узлу был перенаправлен на порт 3000 Egress Gateway
``` shell
oc logs $(oc get pods -o name -l name=kafka-client | head -n 1) -c istio-proxy
[2022-10-25T21:01:00.331Z] "- - -" 0 - - - "-" 85 585 50690 - "-" "-" "-" "-" "10.131.0.79:3000" outbound|3000||ci00000000-test-egress.sbercode-f4869733-8f72-4aae-a1be-eb097f174235-work.svc.cluster.local 10.128.3.43:44730 172.30.26.92:9092 10.128.3.43:58754 - -
``` 
В логах Egress Gateway видим обращение к внешнему узлу
``` shell
oc logs $(oc get pods -o name | grep egress | head -n 1)
``` 
[2022-10-25T21:01:00.334Z] "- - -" 0 - - - "-" 85 585 50688 - "-" "-" "-" "-" "172.30.26.92:9092" outbound|9092||kafka.apps.sbc-okd.pcbltools.ru 10.131.0.79:51230 10.131.0.79:3000 10.128.3.43:44730 - -
