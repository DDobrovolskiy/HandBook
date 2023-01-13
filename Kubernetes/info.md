Установка:  
* https://tproger.ru/articles/kak-ustanovit-kubernetes-s-minikube-na-linux/
* https://habr.com/ru/company/flant/blog/333470/
* https://habr.com/ru/company/vk/blog/648117/

Команды:  
- kubectl version --short
- minikube start
- kubectl get pods --all-namespaces //список запущенных в кластере подов
- kubectl get nodes //список запущенных нод
- kubectl get pods
- kubectl get deployments
- kubectl expose deployment hello-minikube --type=NodePort //Для доступа к сервису hello-minikube нужно открыть ему внешний IP командой 
необходимо использовать тип NodePort, т.к. Minikube не поддерживает сервис LoadBalancer
- kubectl get services //можно убедиться, что сервис стал открыт
- minikube service hello-minikube --url //узнать его внешний IP и порт
- kubectl delete service,deployment hello-minikube

Пример:  
- kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.10 --port=8080

Настройки памяти и процессора с помощью команды:
- $ minikube config set memory 6144
- $ minikube config set cpus 4
- $ minikube start --memory 6144 --cpus 4
- $ minikube start --memory=max --cpus=max

Указание версии Kubernetes:
- $ minikube start --kubernetes-version=v1.19.0
Также имеет смысл указать версию Kubernetes, когда вы тестируете несколько кластеров, работающих на разных версиях. Для этого используйте флаг --profile:
- $ minikube start -p dev --kubernetes-version=v1.19.0
- $ minikube start -p stage --kubernetes-version=v1.18.0

Локальные образы Docker
Если кластер построен на основе среды исполнения контейнера Docker (в отличие от cri-o или containerd), укажите для терминала использование внутреннего демона Docker в кластере с помощью команды:
- $ eval $(minikube docker-env)

Чтобы включить другие дополнения, выполните команду:
- $ minikube addons enable <название дополнения>

Пример YML (pod.yml):
``` yml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod-1
spec:
  Containers:
  - image: quay.io/testing-farm/nginx:1.12
    name: nginx
    ports:
    - containerPort: 80
```
