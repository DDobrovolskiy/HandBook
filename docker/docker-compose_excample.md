```
version: '3'
services:
  db:
    image: postgres:11.4-alpine
    container_name: postgres
    ports:
      - 5432:5432
    volumes:
      - ./pg_data:/var/lib/postgresql/data/pgdata
    environment:
      POSTGRES_PASSWORD: 123
      POSTGRES_DB: docker_test
      PGDATA: /var/lib/postgresql/data/pgdata
    restart: always
  app:
    image: drucoder/web-server
    ports:
      - 3000-3005:3000
    environment:
      POSTGRES_HOST: db
    restart: always
    links:
      - db
  nginx:
    image: nginx:1.17.2-alpine
    container_name: nginx
    volumes:
      - ./default.conf:/etc/nginx/conf.d/default.conf
    links:
      - app
    ports:
      - 8989:8989
```
