###### daemonset
https://github.com/Slurmio/school-dev-k8s/tree/main/practice/10.deployment-alternative  
``` yml 
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: node-exporter
  name: node-exporter
spec:
  updateStrategy:
    rollingUpdate:
      maxUnavailable: 1
    type: RollingUpdate
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      containers:
      - name: node-exporter
        image: k8s.gcr.io/pause:3.3
        imagePullPolicy: IfNotPresent
        resources:
          limits:
            cpu: 10m
            memory: 64Mi
          requests:
            cpu: 10m
            memory: 64Mi
      nodeSelector:
        kubernetes.io/os: linux
      securityContext:
        runAsNonRoot: true
        runAsUser: 65534
      tolerations:
      - effect: NoSchedule
        key: node-role.kubernetes.io/ingress
```

###### statefulset
https://github.com/Slurmio/school-dev-k8s/tree/main/practice/10.deployment-alternative/2.statefulset/rabbitmq-statefulset  