* ```docker run *name*```  Запуск с подключением к контейнеру 
* ```docker -d run *name*``` Запуск без подключением к контейнеру 
* ```docker run -d --name my_name_nginx *name*``` Запуск без подключением к контейнеру с именем "my_name_nginx"
* ```docker run --rm *name*``` Запуск и после остановки удаление контейнера 
* ```docker stop *name*``` Остановка контейнера 
* ```docker container prune``` Удаление остановленных контейнеров
* ```docker container inspect *name* | grep IPAddress``` просмотр настроек docker
* ```docker kill *name*``` Моментальная остановка контейнера 
* ```docker exec -it *name* bash``` Запуск процесса внутри контейнера
* ```docker run -d -p 9999:80 nginx``` Открытие порта внешнего 9999 на внтуренний контенера 80
* ```docker run -d -v ${PWD}:/usr/share/nginx/html -p 9999:80 nginx``` Подключения тома ${PWD} - путь к локальной папке : /usr/share/nginx/html - путь внутри контейнера

* ```docker build . -t *my_image*:*teg*``` Создание образа с именем образа и тегом (lates)

* Разбиение на строки
``` 
docker
run \
--rm \
-d \
nginx 
``` 
