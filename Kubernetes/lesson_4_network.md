https://github.com/Slurmio/school-dev-k8s  

``` yml
---
# file: practice/1.kube-basics-lecture/4.resources-and-probes/deployment-with-stuff.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
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
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /
            port: 80
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /
            port: 80
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
          initialDelaySeconds: 10
        startupProbe:
          httpGet:
            path: /
            port: 80
          failureThreshold: 30
          periodSeconds: 10
        resources:
          requests:
            cpu: 10m
            memory: 100Mi
          limits:
            cpu: 100m
            memory: 100Mi
...
```

###### services

clusterip.yml
``` yml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: my-app
  type: ClusterIP
```

nodeport.yml
``` yml
apiVersion: v1
kind: Service
metadata:
  name: my-service-np
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: my-app
  type: NodePort
```

loadbalancer.yml
``` yml
apiVersion: v1
kind: Service
metadata:
  name: my-service-lb
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: my-app
  type: LoadBalancer
```

external_name.yml
``` yml
apiVersion: v1
kind: Service
metadata:
  name: my-replicaset
spec:
  type: ExternalName
  externalName: example.com
```

externalIps.yml
``` yml
apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  selector:
    app: my-app
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
  externalIps:
  - 80.11.12.10
```

headless.yml
``` yml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ClusterIP: none
  ports:
  - name: http
    port: 80
    tergetPort: 80
    protocol: TCP
```

deployment.yml
``` yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
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
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /
            port: 80
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /
            port: 80
          periodSeconds: 10
          successThreshold: 1
          timeoutSeconds: 1
          initialDelaySeconds: 10
        resources:
          requests:
            cpu: 10m
            memory: 100Mi
          limits:
            cpu: 10m
            memory: 100Mi
        volumeMounts:
        - name: config
          mountPath: /etc/nginx/conf.d/
      volumes:
      - name: config
        configMap:
          name: my-configmap
```

configmap.yml
``` yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-configmap
data:
  default.conf: |
    server {
        listen       80 default_server;
        server_name  _;
        default_type text/plain;
        location / {
            return 200 '$hostname\n';
        }
    }
```

2) Запустим тестовое приложение, с которого мы будем обращаться к основному:

```bash
kubectl run test --image=centosadmin/utils:0.3 -it bash

exit
```

3) Создаем Service типа ClusterIP:

```bash
kubectl apply -f clusterip.yaml
```

4) Убедимся, что Service работает. Узнаем его IP, зайдем внутрь нашего тестового Pod'а и обратимся к основному приложению, используя имя сервиса и IP:

```bash
kubectl get svc
kubectl exec test -it bash

curl <ip-адрес сервиса>
curl my-service

bash-5.0# curl my-service
my-deployment-c6b56c8cf-8bcv9
bash-5.0# curl my-service
my-deployment-c6b56c8cf-5stq9

exit
```

###### ingress

host-ingress.yml
``` yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress-nginx
  annotations:
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  rules:
  - host: foo.mydomain.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: my-service
            port:
              number: 80
```
```bash
minikube addons enable ingress

cd etc/
sudo nano hosts    
```

https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/
