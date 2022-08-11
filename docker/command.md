* ```docker run *name*```  Запуск с подключением к контейнеру 
* ```docker -d run *name*``` Запуск без подключением к контейнеру 
* ```docker run -d --name my_name_nginx nginx``` Запуск без подключением к контейнеру с именем "my_name_nginx"
* ```docker stop *name*``` Остановка контейнера 
* ```docker container prune``` Удаление остановленных контейнеров
* ```docker container inspect *name* | grep IPAddress``` просмотр настроек docker
* ```docker kill *name*``` Моментальная остановка контейнера 
* ```docker exec -it *name* bash``` Запуск процесса внутри контейнера
