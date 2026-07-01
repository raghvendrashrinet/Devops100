
# Createing a Docker compose file 
Create a container named httpd using a docker compose file /data/docker/docker-compose.yml (please use the exact name for file),
-  container is named as httpd; 
- Map 80 number port of container with port 6300 of docker host.
-  Map container's /usr/local/apache2/htdocs volume with /opt/data volume of docker host

 ```yaml
services:
  web-app: 
  - image: httpd
    name: httpd
    ports:
    - 6300:80
    volumes:
    - /opt/data:/usr/local/apache2/htdocs
 ```
