Установка:  
* https://tproger.ru/articles/kak-ustanovit-kubernetes-s-minikube-na-linux/
* https://habr.com/ru/company/flant/blog/333470/

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
