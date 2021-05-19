```shell
  #### Creamos el pod que tendra de nombre "servidor-wp" ####
  vagrant@podman:~$ podman pod create --name servidor-wp -p 8080:80
  
  #### Una vez creado vamos a generar un contenedor que sera la base de datos ####
  vagrant@podman:~$ podman run -d --restart=always --pod=servidor-wp -e MYSQL_ROOT_PASSWORD="wp-password" -e MYSQL_DATABASE="wp" -e MYSQL_USER="wordpress" -e MYSQL_PASSWORD="user-password"  --name=mysql-wp mariadb
  
  #### Creamos por ultimo el contenedor con wordpress ####
  vagrant@podman:~$ podman run -d --restart=always --name=mywordpress --pod=19b301d2965b -e WORDPRESS_DB_NAME="wp" -e WORDPRESS_DB_USER="wordpress" -e WORDPRESS_DB_PASSWORD="user-password" -e   WORDPRESS_DB_HOST="127.0.0.1" --name my-wordpress wordpress
  
  #### Comprobamos que tenemos 3 contenedores en un mismo pod, el contenedor infra, wordpress y mysql ####
  vagrant@podman:~$ podman ps -a --pod
  CONTAINER ID  IMAGE                               COMMAND               CREATED        STATUS            PORTS                 NAMES                     POD ID        PODNAME
  e86e5dd06397  k8s.gcr.io/pause:3.2                                      5 minutes ago  Up 5 minutes ago  0.0.0.0:8080->80/tcp  294226c2dbf3-infra        294226c2dbf3  servidor-wp
  d0c06bdf6d54  docker.io/library/wordpress:latest  apache2-foregroun...  5 minutes ago  Up 5 minutes ago  0.0.0.0:8080->80/tcp  servidor-wp-my-wordpress  294226c2dbf3  servidor-wp
  f51e666c6cce  docker.io/library/mariadb:latest    mysqld                5 minutes ago  Up 5 minutes ago  0.0.0.0:8080->80/tcp  servidor-wp-mysql-wp      294226c2dbf3  servidor-wp
  
  #### Generamos el fichero .yaml para kubernetes ####
  vagrant@podman:~$ podman generate kube servidor-wp >> escenario-wp.yaml
```