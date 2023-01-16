##### REPLICAL SET
Файл replicaset.yml
``` yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
spec:
  replicas: 2
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
          ports:
            - containerPort: 80
```
Создание:
- kubectl create -f replicaset.yml
- kubectl apply -f replicaset.yml

Разница между create (только один раз) и apply (можно несколько раз)  
Показать поды определенного лейбла 
- kubectl get pod -l app=my-app
Изменить количество реплик
- kubectl scale --replicas 3 replicaset my-replicaset 
Очистить 
- kubectl delete replicaset --all

##### DEPLOYMENT
Файл deployment.yml
``` yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 2
  strategy:
    rollingUpdate:
      maxSurge: 50%
      maxUnavailable: 50%
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
          ports:
            - containerPort: 80
```
Включить:
- kubectl apply -f deployment.yml 
Удалить:
- kubectl delete deployments.apps --all
Откатить:
- kubectl rollout undo deployment my-deployment

##### Resource

Файл deployment.yml
``` yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 2
  strategy:
    rollingUpdate:
      maxSurge: 50%
      maxUnavailable: 50%
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
