## The compose should deploy two services (web and DB),
1. For web service:
 a. Container name = php_aweb.
 b.  image = php
 c. Map php_apache container's port 80 with host port 3003
 d. Map php_apache container's /var/www/html volume with host volume /var/www/html.


2. For DB service:
 a. Container name=mysql_web.
 b. image mariadb
 c. Map mysql_apache container's port 3306 with host port 3306
 d. Map mysql_apache container's /var/lib/mysql volume with host volume /var/lib/mysql.
e. Set MYSQL_DATABASE=database_apache and use any custom user ( except root ) with some complex password for DB connections.


After running docker-compose up you can access the app with curl command curl <server-ip or hostname>:3003/

```yaml
services:
  web-app:
    image: php:8.2-apache
    container_name: php_web
    ports:
    - "3003:80"
    volumes:
    - /var/www/html:/var/www/html
   
  db-app:
    image: mariadb
    container_name: mysql_web
    ports:
    - "3306:3306"
    volumes:
    - /var/lib/mysql:/var/lib/mysql
    environment:
    - MARIADB_ROOT_PASSWORD=SuperSecureRootPass2026!
    - MARIADB_DATABASE=database_web
    - MARIADB_USER=dbuser123
    - MARIADB_PASSWORD=Ch@ngMe_2026!

```
