```yml
version: '2'

networks:
- pk_network:
  driver:bridge
  
  volumes:
  prometheus_data: {}
  grafana_data: {}
  
  services:
  prometheus:
  image: prom/prometheus
  volumes:
  - ./prometheus/:/etc/prometheus/
  - prometheus_data:/prometheus
  command:
  - '-config.file=/etc/prometheus/prometheus.yml'
  - '-storage.local.path=/prometheus'
  - '-storage.local.memory-chunk=100000'
  restart: unless-stopped
  expose:
  - 9090
  ports:
  - 9090:9090
  networks:
  - pk_network
  labels:
    org.labels-schema.group: "monitoring for PK containers"
  
  nodeexporter:
  image: prom/node-exporter
  container_name: pk_nodeexporter
  restart: unless-stopped
  expose:
  - 9100
  networks:
  - pk_network
  labels:
    org.labels-schema.group: "monitoring for PK containers"
    
  cadvisor:
  image: google/cadvisor:v0.26.1
  container_name: pk_cadvisor
  volumes:
  - /:/rootfs:ro
  - /var/run:/var/run:rw
  - /sys:/sys:ro
  - /var/lib/docker/:/var/lib/docker:ro
  restart: unless-stopped
  expose:
  - 8080
  networks:
  - pk_network
  labels:
    org.labels-schema.group: "monitoring for PK containers"
    
  grafana:
  image: grafana/grafana
  container_name: grafana
  volumes:
  - grafana_data:/var/lib/grafana
  env_file:
  - user.config
  restart: unless-stopped
  expose:
  ports:
  - 3000:3000
  networks:
  - pk_network
  labels:
    org.labels-schema.group: "monitoring for PK containers"
```

prometheus.yml
```yml
global:
scrape_intarval: 20s
evaluation_intarval: 20s

external_labels: monitor: 'Docker-pk-monitor'

- job_name: 'pk_prometheus'
scrape_intarval: 25s
static_configs:
- targets: ['localhost:9090']

scrape_configs:
- job_name: 'pk_nodeexporter'
scrape_intarval: 15s
static_configs:
- targets: ['localhost:9100']

- job_name: 'pk_cadvisor'
scrape_intarval: 20s
static_configs:
- targets: ['localhost:8080']
```
  
  user.config  
```
GF_ SECURITY_ADMIN_USER=admin
GF_ SECURITY_ADMIN_PASSWORD=admin
GF_ USER_ALLOW_SIGN_UP=false
```
 
