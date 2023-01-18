##### Configurations
Файл:
deployment.yml
``` yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 1
  strategy:
    rollingUpdate:
      maxSurge: 50%
      maxUnavailable: 50%
    type: RollingUpdate
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - image: quay.io/testing-farm/nginx:1.12
          name: nginx
          env:
          - name: TEST 
            value: foo
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 10m
              memory: 100Mi
            limits:
              cpu: 100m
              memory: 100Mi
```
Манифест configmap.yml
``` yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-configmap-env
data:
  dbhost: postgresql
  DEBUG: "false"
```
Применить:
- kubectl apply -f configmap.yml 
- kubectl get configmaps

Редактируем deployment.yml:
``` yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 1
  strategy:
    rollingUpdate:
      maxSurge: 50%
      maxUnavailable: 50%
    type: RollingUpdate
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - image: quay.io/testing-farm/nginx:1.12
          name: nginx
          env:
          - name: TEST 
            value: foo
          envFrom:
          - configMapRef:
              name: my-configmap-env
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 10m
              memory: 100Mi
            limits:
              cpu: 100m
              memory: 100Mi
```

Посмотреть что внутри:
- kubectl describe pod my-deployment-57764d7b57-55jkd  
- kubectl get configmaps my-configmap-env -o yaml  
Зайти на внутрь контейнера с консолью
>kubectl exec -it my-deployment-57764d7b57-55jkd -- bash  
>kubectl exec -it my-deployment-57764d7b57-55jkd -- env

##### Secrets
1 Создаем секрет
``` bash
kubectl create secret generic my-name-secret --from-literal=test1=asdf --from-literal=dbpassword=postgres
kubectl get secret
kubectl get secret my-name-secret -o yaml  

echo cG9zdGdyZXM= | base64 -d
```
2 Применим в деплоймент
deployment.yml:
``` yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 1
  strategy:
    rollingUpdate:
      maxSurge: 50%
      maxUnavailable: 50%
    type: RollingUpdate
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - image: quay.io/testing-farm/nginx:1.12
          name: nginx
          env:
          - name: TEST 
            value: foo
          - name: TEST_1
            valueFrom:
              secretKeyRef:
                name: my-name-secret
                key: dbpassword
          envFrom:
          - configMapRef:
              name: my-configmap-env
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 10m
              memory: 100Mi
            limits:
              cpu: 100m
              memory: 100Mi
```
``` bash
kubectl apply -f ...
```

4 Применяем манифест с секретом 
secret.yml:
``` yml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
stringData:
  test: updated
```
``` bash
kubectl apply -f ...
```
Новый тип  
Манифест configmap.yml
``` yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-configmap
data:
  default.conf: |
    server {
    	listen 80 default_server;
    	
    	server_name _;
    	default_type text/plain;
    	
    	location / {
          return 200 'Show name\n';
      }
    }
```
``` bash
kubectl get configmap my-configmap -o yaml
```
deployment.yml:
``` yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 1
  strategy:
    rollingUpdate:
      maxSurge: 50%
      maxUnavailable: 50%
    type: RollingUpdate
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - image: quay.io/testing-farm/nginx:1.12
          name: nginx
          env:
          - name: TEST 
            value: foo
          - name: TEST_1
            valueFrom:
              secretKeyRef:
                name: my-name-secret
                key: dbpassword
          envFrom:
          - configMapRef:
              name: my-configmap-env
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 10m
              memory: 100Mi
            limits:
              cpu: 100m
              memory: 100Mi
          volumeMounts:
          - name: config
            mountPath: etc/nginx/conf.d/
      volumes:
      - name: config
        configMap:
          name: my-configmap
```
``` bash
kubectl exec -it my-deployment-557d7d7775-z5wgh -- bash
cd etc/nginx/conf.d/
cat default.conf
```
Проброска портов
``` bash
kubectl port-forward my-deployment-557d7d7775-z5wgh 8080:80 & curl 127.0.0.1:8080
```
##### DownwardAPI
deployment.yml:
``` yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 1
  strategy:
    rollingUpdate:
      maxSurge: 50%
      maxUnavailable: 50%
    type: RollingUpdate
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - image: quay.io/testing-farm/nginx:1.12
          name: nginx
          env:
          - name: TEST 
            value: foo
          - name: TEST_1
            valueFrom:
              secretKeyRef:
                name: my-name-secret
                key: dbpassword
          - name: __NODE_NAME #new ->
            valueFrom:
              fieldRef:
                fieldPath: spec.nodeName
          - name: __POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: __POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace
          - name: __POD_IP
            valueFrom:
              fieldRef:
                fieldPath: status.podIP
          - name: __NODE_IP
            valueFrom:
              fieldRef:
                fieldPath: status.hostIP
          - name: __POD_SERVICE_ACCOUNT
            valueFrom:
              fieldRef:
                fieldPath: spec.serviceAccountName #new <-
          envFrom:
          - configMapRef:
              name: my-configmap-env
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 10m
              memory: 100Mi
            limits:
              cpu: 100m
              memory: 100Mi
          volumeMounts:
          - name: config
            mountPath: /etc/nginx/conf.d/
          - name: podinfo
            mountPath: /etc/podinfo
      volumes:
      - name: config
        configMap:
          name: my-configmap
      - name: podinfo  #new ->
        downwardAPI:
          items:
            - path: "labels"
              fieldRef:
                fieldPath: metadata.labels
            - path: "annotations"
              fieldRef:
                fieldPath: metadata.annotations   #new <-
              
```
Расширенная информация о подах
``` bash
kubectl get pods -o wide
```

