```shell
#### Creamos un nuevo pod llamado bookmedik ####
vagrant@podman:~$ podman pod create --name bookmedik -p 8082:80
#### Creamos la nueva imagen con la app de bookmedik ####
vagrant@podman:~/Proyecto-integrado/podman/escenario-bookmedik/Build$ buildah bud -f Dockerfile -t bookmedik-podman
#### Generamos el contenedor con la base de datos ####
vagrant@podman:~$ podman run -d --restart=always --pod=bookmedik -e MYSQL_ROOT_PASSWORD="franciscojavier" -e MYSQL_DATABASE="bookmedik" -e MYSQL_USER="bookmedik" -e MYSQL_PASSWORD="bookmedik" --name=mysql-bookmedik mariadb
#### Generamos el contenedor con la imagen que hemos creado anteriormente ####
vagrant@podman:~$ podman run -d --restart=always --pod=bookmedik --name=bookmedik-app localhost/bookmedik-podman
#### Comprobamos que tenemos un pod con 3 contenedores, mysql, bookmedik y el contenedor infra ####
vagrant@podman:~$ podman ps -a --pod
CONTAINER ID  IMAGE                             COMMAND               CREATED      STATUS          PORTS                 NAMES               POD ID        PODNAME
cdbff5cd897d  k8s.gcr.io/pause:3.2                                    2 hours ago  Up 2 hours ago  0.0.0.0:8082->80/tcp  aed61cb395da-infra  aed61cb395da  bookmedik
093135c6f6bf  localhost/bookmedik-podman        /usr/local/bin/sc...  2 hours ago  Up 2 hours ago  0.0.0.0:8082->80/tcp  bookmedik-app       aed61cb395da  bookmedik
557d0f451949  docker.io/library/mariadb:latest  mysqld                2 hours ago  Up 2 hours ago  0.0.0.0:8082->80/tcp  mysql-bookmedik     aed61cb395da  bookmedik
```

Una vez hayamos creado el pod con los 3 contenedores necesitamos cargar en la base de datos de nuestro contenendor **mysql-bookmedik** los datos para poder usar la app de bookmedik, para ello debemos de cargar el [fichero siguiente](https://github.com/FranJaviMN/Proyecto-integrado/blob/main/podman/escenario-bookmedik/Build/bookmedik/schema.sql) en la base de datos.
```shell
#### Cargamos el fichero .sql en la base de datos de nuestro contenedor mysql-bookmedik ####
vagrant@podman:~$ cat schema.sql | podman exec -i mysql-bookmedik /usr/bin/mysql -u root --password=franciscojavier bookmedik
```
Una vez tengamos ya la base de datos entramos en **IP:8082** y como datos de acceso es usuario: admin y contraseña:admin y debemos de tener lo siguiente:
![imagen de bookmedik](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Proyecto/captura-bookmedik.podman.png)
