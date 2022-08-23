#### DOCKER
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
* ```docker exec [options] *container* command [arg...]```  Удаленно выполнить команду в уже зупещенном контейнере;
* ```docker rename *name* *newname*```  Переименовать контейнер;
* ```docker cp [options] *name*:src_path dest_path``` Копирование файлов из контейнера на локальную машину;
* ```docker cp [options] src_path| - *name*:dest_path``` Копирование файлов из локальной машины в контейнер;
* ```docker pause/unpause *name*``` Пауза / снятие с паузы; 
* ```docker create [options] *image* [command] [arg...]``` Еще один уровень контейнера из образа;
* ```docker commit [options] *container* [repository:tag]``` Новый образ из контейра;
* ```docker diff *container*``` Изменения в контейнере;
#### DOCKER COMPOSE
* ```docker-compose up```  запускаем все контейнеры, видим stdout всех контейнеров, а для остановки используем Ctrl+C 
* ```docker-compose up -d```  запуск в режиме демона  
* ```docker-compose stop```    для остановки используем  
* ```docker-compose down```   для остановки с удалением контейнеров   
* ```docker-compose -f docker-compose.db.develop.yml up -d``` для файлов с не штатным названием  


* Разбиение на строки
``` 
docker
run \
--rm \
-d \
nginx 
``` 

https://github.com/bcicen/ctop  
https://github.com/jesseduffield/lazydocker  
https://www.youtube.com/watch?v=4KbL5lbjK-M&list=PLU2ftbIeotGoGFC_2lj-OplT_cItXfu48&index=4&ab_channel=letsCode   
  
https://github.com/vegasbrianc/prometheus  
