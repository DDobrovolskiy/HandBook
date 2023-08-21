#### DestinationRule
Пример манифеста

Манифест DestinationRule управляет обработкой трафика после маршрутизации, в частности это может быть: балансировка, проверка доступности хостов, SSL-терминирование.

В приведенном примере описывается DestinationRule, который реализует SSL-терминирование исходящего соединения к кафке. То есть на вход в Egress Gateway приходит незашифрованный трафик, на выходе из Pod Egress трафик заворачивается в mtls соединение и далее, за пределы пространства имен, трафик идет в зашифрованном виде.

В ключе tls манифеста, описываются поля:

mode: MUTUAL - соединение осуществляется по mtls.
caCertificate - цепочка сертификатов, включая корневые и промежуточные, обеспечивающая доверие в соединении. Частая ошибка при настройке соединения с кафкой или других mtls соединений: в данный файл администратор забывает поместить сертификаты доверия к серверному сертификату, к которому происходит подключение, то есть, в случае кафки в данный файл должны быть помещены корневой и промежуточный сертификаты серверного сертификата kafka. Если этого не сделать, то клиент при подключении не может верифицировать серверный сертификат конечной точки, к которой осуществляется подключение.
clientCertificate - клиентский сертификат содержащий DN, по которому осуществляется аутентификация/авторизация на противоположном конце mtls
privateKey - приватный ключ к клиентскому сертификату

#### Подготовительные работы
Запустим экземпляр приложения для отправки HTTP-запросов
``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: client
    marker: practice
  name: client
spec:
  selector:
    matchLabels:
      app: client
  template:
    metadata:
      annotations:
        sidecar.istio.io/inject: 'true'
      labels:
        app: client
        maistra.io/expose-route: "true"
    spec:
      containers:
        - image: >-
            nexus.local:5000/kib/http-client:1.0.0
          name: client
          command:
            - sh
            - '-c'
            - while true; do sleep 30; done
          env:
            - name: EASY_ADDRESS
              value: 4a1fa59d-c737-4102-a2df-cef1fe34afa5.apps.sbc-okd.pcbltools.ru
            - name: SIMPLE_ADDRESS
              value: 01a47b88-7dd8-4548-ada1-bab9599efc41.apps.sbc-okd.pcbltools.ru
            - name: MUTUAL_ADDRESS
              value: ddb8b515-40b1-4ead-aaf4-2e52704bc73e.apps.sbc-okd.pcbltools.ru
          imagePullPolicy: Always
          resources:
            limits:
              cpu: 100m
              memory: 128Mi
            requests:
              cpu: 100m
              memory: 128Mi
---
kind: Service
apiVersion: v1
metadata:
  name: client
  labels:
    app: client
    marker: practice
spec:
  ports:
    # Обратите внимание на название порта
    - name: tcp-8080
      port: 8080
  selector:
    app: client
```

``` shell
oc apply -f client.yml
``` 
Дождемся, пока под запустится
``` shell
oc get pods -l app=client
``` 
Проверим подключение к внешнему узлу из workload-контейнера пода клиента
``` shell
oc exec $(oc get pods -o name -l app=client | head -n 1) -- sh -c 'curl -v http://$EASY_ADDRESS'
```
В терминале и логе контейнера istio-proxy видим ошибку 502, т.к. соединение не настроено
``` shell
oc logs $(oc get pods -o name -l app=client | head -n 1) -c istio-proxy
[2022-10-26T22:22:30.254Z] "GET / HTTP/1.1" 502 - direct_response - "-" 0 0 0 - "-" "curl/7.85.0-DEV" "3cc763c2-8d13-9675-be67-f8f665fafaec" "495aa19d-72cd-4f6d-a488-14175d4087d8.apps.sbc-okd.pcbltools.ru" "-" - - 178.170.196.65:80 10.131.0.186:38118 - block_all
``` 

#### Простая реализация - HTTP соединение без Egress Gateway
Для объявления возможности подключения к внешнему ресурсу используется объект Istio Service Entry

Изучите шаблон ServiceEntry для настройки TCP-соединения с внешним узлом в файле, обратите внимание на комментарии
``` yaml
apiVersion: template.openshift.io/v1
kind: Template
metadata:
  name: test-server-template
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
      location: MESH_EXTERNAL
      # Перечень портов
      ports:
        - name: http-${PORT}
          number: ${{PORT}}
          # Перечень протоколов: HTTP|HTTPS|GRPC|HTTP2|MONGO|TCP|TLS. Для Kafka указываем TCP
          protocol: HTTP
      # Обращение к внешнему хосту будет осуществляться по доменному имени из блока hosts
      resolution: DNS
parameters:
  - name: HOST
    required: true
  - name: PORT
    required: true
  - name: PREFIX
    required: true
```
Применим настройки ServiceEntry с параметрами из файла
``` env
HOST=4a1fa59d-c737-4102-a2df-cef1fe34afa5.apps.sbc-okd.pcbltools.ru
PORT=80
PREFIX=easy-test-server
```

``` shell
oc process -f easy.yml --param-file easy-params.env -o yaml > conf.yml
oc apply -f conf.yml
``` 
Повторим проверку, соединение успешно
``` shell
oc exec $(oc get pods -o name -l app=client | head -n 1) -- sh -c 'curl -v http://$EASY_ADDRESS'
``` 
В логах контейнера istio-proxy видим прямое обращение к внешнему узлу
``` shell
oc logs $(oc get pods -o name -l app=client | head -n 1) -c istio-proxy
```

#### Настройка Egress Gateway
Изучите шаблон Egress Gateway в файле. Обратите внимание, что в блоки Volumes и VolumeMounts добавлен Secret с клиентским сертификатом
``` yaml
apiVersion: template.openshift.io/v1
kind: Template
metadata:
  name: egressgateway-template
objects:
  - apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: ${EGRESS_NAME}
        app.kubernetes.io/component: gateways
        app.kubernetes.io/instance: ${CONTROL_PLANE}
        app.kubernetes.io/name: gateways
        app.kubernetes.io/part-of: istio
        chart: gateways
        heritage: Tiller
        istio: ${EGRESS_NAME}
        maistra-version: 2.1.0
        release: istio
        marker: practice
      name: ${EGRESS_NAME}
    spec:
      progressDeadlineSeconds: 1200
      revisionHistoryLimit: 0
      selector:
        matchLabels:
          app: ${EGRESS_NAME}
          istio: ${EGRESS_NAME}
      template:
        metadata:
          annotations:
            sidecar.istio.io/inject: "false"
          labels:
            app: ${EGRESS_NAME}
            chart: gateways
            heritage: Tiller
            istio: ${EGRESS_NAME}
            release: istio
        spec:
          containers:
            - resources:
                limits:
                  cpu: 200m
                  memory: 256Mi
                requests:
                  cpu: 100m
                  memory: 128Mi
              readinessProbe:
                httpGet:
                  path: /healthz/ready
                  port: 15021
                  scheme: HTTP
                initialDelaySeconds: 1
                timeoutSeconds: 3
                periodSeconds: 2
                successThreshold: 1
                failureThreshold: 30
              terminationMessagePath: /dev/termination-log
              name: istio-proxy
              env:
                - name: JWT_POLICY
                  value: first-party-jwt
                - name: PILOT_CERT_PROVIDER
                  value: istiod
                - name: CA_ADDR
                  value: 'istiod-basic.${CONTROL_PLANE}.svc:15012'
                - name: POD_NAME
                  valueFrom:
                    fieldRef:
                      apiVersion: v1
                      fieldPath: metadata.name
                - name: POD_NAMESPACE
                  valueFrom:
                    fieldRef:
                      apiVersion: v1
                      fieldPath: metadata.namespace
                - name: INSTANCE_IP
                  valueFrom:
                    fieldRef:
                      apiVersion: v1
                      fieldPath: status.podIP
                - name: SERVICE_ACCOUNT
                  valueFrom:
                    fieldRef:
                      apiVersion: v1
                      fieldPath: spec.serviceAccountName
                - name: HOST_IP
                  valueFrom:
                    fieldRef:
                      apiVersion: v1
                      fieldPath: status.hostIP
                - name: CANONICAL_SERVICE
                  valueFrom:
                    fieldRef:
                      apiVersion: v1
                      fieldPath: 'metadata.labels[''service.istio.io/canonical-name'']'
                - name: CANONICAL_REVISION
                  valueFrom:
                    fieldRef:
                      apiVersion: v1
                      fieldPath: 'metadata.labels[''service.istio.io/canonical-revision'']'
                - name: PROXY_CONFIG
                  value: >
                    {"discoveryAddress":"istiod-basic.${CONTROL_PLANE}.svc:15012","tracing":{"zipkin":{"address":"jaeger-collector.${CONTROL_PLANE}.svc:9411"}},"proxyMetadata":{"ISTIO_META_DNS_AUTO_ALLOCATE":"true","ISTIO_META_DNS_CAPTURE":"true","PROXY_XDS_VIA_AGENT":"true"}}
                - name: ISTIO_META_POD_PORTS
                  value: |-
                    [
                    ]
                - name: ISTIO_META_APP_CONTAINERS
                  value: server
                - name: ISTIO_META_CLUSTER_ID
                  value: Kubernetes
                - name: ISTIO_META_INTERCEPTION_MODE
                  value: REDIRECT
                - name: ISTIO_METAJSON_ANNOTATIONS
                  value: |
                    {"openshift.io/scc":"restricted","sidecar.istio.io/inject":"true"}
                - name: ISTIO_META_WORKLOAD_NAME
                  value: server
                - name: ISTIO_META_OWNER
                  value: >-
                    kubernetes://apis/apps/v1/namespaces/istio-practice-var1/deployments/server
                - name: ISTIO_META_MESH_ID
                  value: cluster.local
                - name: TRUST_DOMAIN
                  value: cluster.local
                - name: ISTIO_META_DNS_AUTO_ALLOCATE
                  value: 'true'
                - name: ISTIO_META_DNS_CAPTURE
                  value: 'true'
                - name: PROXY_XDS_VIA_AGENT
                  value: 'true'
              ports:
                - name: http-envoy-prom
                  containerPort: 15090
                  protocol: TCP
              imagePullPolicy: Always
              volumeMounts:
                - mountPath: /etc/istio/egress-certs
                  name: egress-certs
                - name: istiod-ca-cert
                  mountPath: /var/run/secrets/istio
                - name: istio-data
                  mountPath: /var/lib/istio/data
                - name: istio-envoy
                  mountPath: /etc/istio/proxy
                - name: istio-podinfo
                  mountPath: /etc/istio/pod
                - name: kube-api-access-wx7h4
                  readOnly: true
                  mountPath: /var/run/secrets/kubernetes.io/serviceaccount
              terminationMessagePolicy: File
              image: 'quay.io/maistra/proxyv2-ubi8:2.1.0'
              args:
                - proxy
                - router
                - '--domain'
                - $(POD_NAMESPACE).svc.cluster.local
                - '--proxyLogLevel=warning'
                - '--proxyComponentLogLevel=misc:error'
                - '--log_output_level=default:info'
                - '--serviceCluster'
                - ${EGRESS_NAME}
                - '--trust-domain=cluster.local'
          volumes:
            - name: istio-envoy
              emptyDir:
                medium: Memory
            - name: istio-data
              emptyDir: {}
            - name: istio-podinfo
              downwardAPI:
                items:
                  - path: labels
                    fieldRef:
                      apiVersion: v1
                      fieldPath: metadata.labels
                  - path: annotations
                    fieldRef:
                      apiVersion: v1
                      fieldPath: metadata.annotations
                  - path: cpu-limit
                    resourceFieldRef:
                      containerName: istio-proxy
                      resource: limits.cpu
                      divisor: 1m
                  - path: cpu-request
                    resourceFieldRef:
                      containerName: istio-proxy
                      resource: requests.cpu
                      divisor: 1m
                defaultMode: 420
            - name: istiod-ca-cert
              configMap:
                name: istio-ca-root-cert
                defaultMode: 420
            - name: kube-api-access-wx7h4
              projected:
                sources:
                  - serviceAccountToken:
                      expirationSeconds: 3607
                      path: token
                  - configMap:
                      name: kube-root-ca.crt
                      items:
                        - key: ca.crt
                          path: ca.crt
                  - downwardAPI:
                      items:
                        - path: namespace
                          fieldRef:
                            apiVersion: v1
                            fieldPath: metadata.namespace
                  - configMap:
                      name: openshift-service-ca.crt
                      items:
                        - key: service-ca.crt
                          path: service-ca.crt
            - name: egress-certs
              secret:
                defaultMode: 256
                secretName: certs
  - apiVersion: v1
    kind: ConfigMap
    metadata:
      name: egress-config
      labels:
        app.kubernetes.io/component: istio-discovery
        app.kubernetes.io/instance: ${CONTROL_PLANE}
        app.kubernetes.io/name: istio-discovery
        app.kubernetes.io/part-of: istio
        istio.io/rev: basic
        maistra-version: 2.0.2
        release: istio
        marker: practice
    data:
      mesh: |-
        accessLogEncoding: TEXT
        accessLogFile: /dev/stdout
        accessLogFormat: ""
        defaultConfig:
          concurrency: 2
          configPath: ./etc/istio/proxy
          controlPlaneAuthPolicy: NONE
          discoveryAddress: istiod-basic.${CONTROL_PLANE}.svc:15012
          drainDuration: 45s
          parentShutdownDuration: 1m0s
          proxyAdminPort: 15000
          proxyMetadata:
            DNS_AGENT: ""
          serviceCluster: istio-proxy
          tracing:
            tlsSettings:
              caCertificates: null
              clientCertificate: null
              mode: DISABLE
              privateKey: null
              sni: null
              subjectAltNames: []
            zipkin:
              address: jaeger-collector.${CONTROL_PLANE}.svc:9411
        disableMixerHttpReports: true
        disablePolicyChecks: true
        enableAutoMtls: true
        enableEnvoyAccessLogService: false
        enablePrometheusMerge: false
        enableTracing: true
        egressClass: istio
        egressControllerMode: STRICT
        egressService: istio-egressgateway
        localityLbSetting:
          enabled: true
        outboundTrafficPolicy:
          mode: ALLOW_ANY
        protocolDetectionTimeout: 5000ms
        reportBatchMaxEntries: 100
        reportBatchMaxTime: 1s
        rootNamespace: ${CONTROL_PLANE}
        sdsUdsPath: unix:/etc/istio/proxy/SDS
        trustDomain: cluster.local
        trustDomainAliases: null
      meshNetworks: 'networks: {}'
  - kind: Service
    apiVersion: v1
    metadata:
      name: ${EGRESS_NAME}
      labels:
        app: ${EGRESS_NAME}
        marker: practice
    spec:
      ports:
        - name: https-3000
          port: 3000
        - name: https-3001
          port: 3001
      selector:
        app: ${EGRESS_NAME}
        istio: ${EGRESS_NAME}
parameters:
  - name: PROJECT_NAME
    required: true
  - name: CONTROL_PLANE
    required: true
  - name: EGRESS_NAME
    required: true
```
Для запуска своего Egress Gateway нам потребуются:
* Имя проекта  
* Имя Control Plane Istio, к которой подключен проект
* Название сервиса Egress. Название должно содержать КЭ АС, это требование сопровождения Istio
* Перечень портов, используемых для организации трафика через Egress Gateway. Все порты должны быть объявлены в Service Egress Gateway
* Secret, содержащий клиентский сертификат и цепочку доверенных сертификатов
Получим имя проекта
``` shell
oc project -q
``` 
Получим имя Control Plane
``` shell
oc describe project $(oc project -q) | grep member-of | head -n 1 | cut -d '=' -f2
``` 
Заполните имена проекта и Control Plane в файле
``` env
PROJECT_NAME=sbercode-8edc35c8-9995-4743-b441-7d9daf4bbaa7-work
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

#### Настройка Simple TLS
Настроим Simple TLS для исходящих запросов. Требуется указать только цепочки доверенных УЦ. Т.к. используем самоподписанный сертификат, то используем его вместо цепочек

Изучите шаблон для настройки Simple TLS на Egress Gateway в файле, обратите внимание на комментарии
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
      location: MESH_EXTERNAL
      ports:
        # В ServiceEntry указываем 2 порта, т.к. от приложения идет HTTP вызов, а от Egress - HTTPS
        - name: https-${HTTPS_PORT}
          number: ${{HTTPS_PORT}}
          protocol: HTTPS
        - name: http-${HTTP_PORT}
          number: ${{HTTP_PORT}}
          protocol: HTTP
      resolution: DNS
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
            name: http-${{EGRESS_PORT}}
            number: ${{EGRESS_PORT}}
            protocol: HTTP
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
      # Этот пункт важен. Для TC отдельный конфиг
      http:
        # Приложение -> Egress
        - match:
            - gateways:
                - mesh
              # HTTP-порт для перенаправления на Egress
              port: ${{HTTP_PORT}}
          route:
            - destination:
                host: ${EGRESS_NAME}
                # Порт Egress Gateway
                port:
                  number: ${{EGRESS_PORT}}
        # Egress -> внешний узел
        - match:
            - gateways:
                - ${PREFIX}-gw
              # Порт Egress Gateway
              port: ${{EGRESS_PORT}}
          route:
            - destination:
                host: ${HOST}
                # HTTPS-порт для перенаправления из Egress
                port:
                  number: ${{HTTPS_PORT}}
  - apiVersion: networking.istio.io/v1alpha3
    kind: DestinationRule
    metadata:
      name: ${PREFIX}-dr
      labels:
        marker: practice
    spec:
      host: ${HOST}
      exportTo:
        - .
      trafficPolicy:
        loadBalancer:
          simple: ROUND_ROBIN
        portLevelSettings:
          - port:
              number: 443
            tls:
              # Цепочки доверенных УЦ
              caCertificates: /etc/istio/egress-certs/crt.pem
              # Для режима SIMPLE клиентский сертификат не требуется
              mode: SIMPLE
              # Имя узла используется для проверки SAN сертификата сервера
              sni: ${HOST}
parameters:
  - name: HOST
    required: true
  - name: HTTP_PORT
    required: true
  - name: HTTPS_PORT
    required: true
  - name: PREFIX
    required: true
  - name: EGRESS_NAME
    required: true
  - name: EGRESS_PORT
    required: true
```

``` env
HOST=01a47b88-7dd8-4548-ada1-bab9599efc41.apps.sbc-okd.pcbltools.ru
HTTPS_PORT=443
HTTP_PORT=80
PREFIX=simple-test-server
EGRESS_NAME=ci00000000-test-egress
EGRESS_PORT=3000
```

 ``` shell
oc process -f simple.yml --param-file simple-params.env -o yaml > conf.yml
oc apply -f conf.yml
``` 
Проверим подключение. Обратите внимание, что приложение по-прежнему обращается к порту 80, а уже Egress Gateway - к порту 443
``` shell
oc exec $(oc get pods -o name -l app=client | head -n 1) -- sh -c 'curl -v http://$SIMPLE_ADDRESS'
``` 
В логах istio-proxy, что запрос к внешнему узлу был перенаправлен на порт 3000 Egress Gateway
``` shell
oc logs $(oc get pods -o name -l app=client | head -n 1) -c istio-proxy
[2022-10-23T18:28:33.789Z] "GET / HTTP/1.1" 200 - via_upstream - "-" 0 44 28 27 "-" "curl/7.64.0" "56140728-4722-972f-afaa-299e00837e1a" "349bc685-0338-48e8-bb74-414c2fd9f301.apps.sbc-okd.pcbltools.ru" "10.128.2.35:3000" outbound|3000||ci00000000-test-egress.sbercode-a06d6040-d4da-456c-ae55-2725df9438f0-work.svc.cluster.local 10.128.2.33:50174 240.240.0.2:80 10.128.2.33:56948 - -
``` 
В логах Egress Gateway видим обращение к порту 443 внешнего узла
``` shell
oc logs $(oc get pods -o name | grep egress | head -n 1)
[2022-10-23T18:28:33.790Z] "GET / HTTP/1.1" 200 - via_upstream - "-" 0 44 21 20 "10.128.2.33" "curl/7.64.0" "56140728-4722-972f-afaa-299e00837e1a" "349bc685-0338-48e8-bb74-414c2fd9f301.apps.sbc-okd.pcbltools.ru" "178.170.196.65:443" outbound|443||349bc685-0338-48e8-bb74-414c2fd9f301.apps.sbc-okd.pcbltools.ru 10.128.2.35:49182 10.128.2.35:3000 10.128.2.33:50174 - -
``` 
#### Настройка Mutual TLS
Настроим Mutual TLS для исходящих запросов. Для Mutual TLS кроме цепочек УЦ необходимо указать открытую и закрытую части сертификата
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
      location: MESH_EXTERNAL
      ports:
        # В ServiceEntry указываем 2 порта, т.к. от приложения идет HTTP вызов, а от Egress - HTTPS
        - name: https-${HTTPS_PORT}
          number: ${{HTTPS_PORT}}
          protocol: HTTPS
        - name: http-${HTTP_PORT}
          number: ${{HTTP_PORT}}
          protocol: HTTP
      resolution: DNS
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
            name: http-${{EGRESS_PORT}}
            number: ${{EGRESS_PORT}}
            protocol: HTTP
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
      # Этот пункт важен, для TCP отдельный конфиг
      http:
        # Приложение -> Egress
        - match:
            - gateways:
                - mesh
              # HTTP-порт для перенаправления на Egress
              port: ${{HTTP_PORT}}
          route:
            - destination:
                host: ${EGRESS_NAME}
                # Порт Egress Gateway
                port:
                  number: ${{EGRESS_PORT}}
        # Egress -> внешний узел
        - match:
            - gateways:
                - ${PREFIX}-gw
              # Порт Egress Gateway
              port: ${{EGRESS_PORT}}
          route:
            - destination:
                host: ${HOST}
                # HTTPS-порт для перенаправления из Egress
                port:
                  number: ${{HTTPS_PORT}}
  # DestinationRule для настроек TLS
  - apiVersion: networking.istio.io/v1alpha3
    kind: DestinationRule
    metadata:
      name: ${PREFIX}-dr
      labels:
        marker: practice
    spec:
      host: ${HOST}
      exportTo:
        - .
      trafficPolicy:
        loadBalancer:
          simple: ROUND_ROBIN
        portLevelSettings:
          - port:
              number: 443
            tls:
              # Цепочки доверенных УЦ
              caCertificates: /etc/istio/egress-certs/ca.pem
              # Для MUTUAL указываем клиентский сертификат
              clientCertificate: /etc/istio/egress-certs/crt.pem
              privateKey: /etc/istio/egress-certs/key.pem
              mode: MUTUAL
              # Имя узла используется для проверки SAN сертификата сервера
              sni: ${HOST}
parameters:
  - name: HOST
    required: true
  - name: HTTP_PORT
    required: true
  - name: HTTPS_PORT
    required: true
  - name: PREFIX
    required: true
  - name: EGRESS_NAME
    required: true
  - name: EGRESS_PORT
    required: true
```

``` env
HOST=ddb8b515-40b1-4ead-aaf4-2e52704bc73e.apps.sbc-okd.pcbltools.ru
HTTPS_PORT=443
HTTP_PORT=80
PREFIX=mutual-test-server
EGRESS_NAME=ci00000000-test-egress
EGRESS_PORT=3001
```
Изучите шаблон для настройки Mutual TLS на Egress Gateway в файле, обратите внимание на комментарии

  ``` shell
oc process -f mutual.yml --param-file mutual-params.env -o yaml > conf.yml
oc apply -f conf.yml
 ``` 
Проверим подключение
 ``` shell
oc exec $(oc get pods -o name -l app=client | head -n 1) -- sh -c 'curl -v http://$MUTUAL_ADDRESS'
 ``` 
В логах istio-proxy, что запрос к внешнему узлу был перенаправлен на порт 3001 Egress Gateway
 ``` shell
oc logs $(oc get pods -o name -l app=client | head -n 1) -c istio-proxy
[2022-10-23T18:29:53.969Z] "GET / HTTP/1.1" 200 - via_upstream - "-" 0 44 20 20 "-" "curl/7.64.0" "14c51ea4-a2e7-94fa-9e8e-45f5a9a7ef52" "b9e93687-47e6-4ff6-a64c-7fa635e08ef7.apps.sbc-okd.pcbltools.ru" "10.128.2.35:3001" outbound|3001||ci00000000-test-egress.sbercode-a06d6040-d4da-456c-ae55-2725df9438f0-work.svc.cluster.local 10.128.2.33:57396 240.240.0.3:80 10.128.2.33:50008 - -
 ``` 
В логах Egress Gateway видим обращение к порту 443 внешнего узла
 ``` shell
oc logs $(oc get pods -o name | grep egress | head -n 1)
[2022-10-23T18:29:53.969Z] "GET / HTTP/1.1" 200 - via_upstream - "-" 0 44 13 13 "10.128.2.33" "curl/7.64.0" "14c51ea4-a2e7-94fa-9e8e-45f5a9a7ef52" "b9e93687-47e6-4ff6-a64c-7fa635e08ef7.apps.sbc-okd.pcbltools.ru" "178.170.196.65:443" outbound|443||b9e93687-47e6-4ff6-a64c-7fa635e08ef7.apps.sbc-okd.pcbltools.ru 10.128.2.35:56616 10.128.2.35:3001 10.128.2.33:57396 - -
 ``` 


