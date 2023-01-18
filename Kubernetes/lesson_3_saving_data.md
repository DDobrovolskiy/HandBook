##### SAVING DATA (HOST PATH) not best practics
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
          - name: data
            mountPath: /files
      volumes:
      - name: data
        hostPath:
          path: /data_pod              
```
##### SAVING DATA EmptyDir 
Для каждого Пода создается свой епти дир  
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
          - name: data
            mountPath: /files
      volumes:
      - name: data
        emptyDir: {}      
```
##### SAVING DATA PV / PVC
storageclass.yml:
``` yml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

pv.yml:
``` yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: fileshare
  labels:
    type: local
spec:
  storageClassName: local-storage
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 10Mi
  hostPath:
    path: /data/pv0001/
```

pvc.yml:
``` yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: fileshare
spec:
  storageClassName: local-storage
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 10Mi
```
configmap.yml:
``` yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fileshare
data:
  default.conf: |
    server {
    	listen 80 default_server;
    	
    	server_name _;
    	default_type text/plain;
    	
    	location / {
          return 200 'Show name $hostname\n';
      }
      
      location /files {
        alias /data;
        autoindex on;
        client_body_temp_path /tmp;
        dav_methods PUT DELETE MKCOL COPY MOVE;
        create_full_put_path on;
        dav_access user:rw group:rw all:r;
      }
    }
```
deployment_fileshare.yml:
``` yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fileshare
  namespace: default
spec:
  replicas: 2
  strategy:
    rollingUpdate:
      maxSurge: 50%
      maxUnavailable: 50%
    type: RollingUpdate
  selector:
    matchLabels:
      app: fileshare
  template:
    metadata:
      labels:
        app: fileshare
    spec:
      initContainers:
        - image: busybox
          name: mount-permissions-fix
          command: ["sh", "-c", "chmod 777 /data"]
          volumeMounts:
          - name: data
            mountPath: /data
      containers:
        - image: centosadmin/reloadable-nginx:1.12
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
          volumeMounts:
          - name: config
            mountPath: /etc/nginx/conf.d
          - name: data
            mountPath: /data
      volumes:
      - name: config
        configMap:
          name: fileshare
      - name: data
        persistentVolumeClaim:
          claimName: fileshare 
```
