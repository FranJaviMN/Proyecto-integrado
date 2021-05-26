Para este escenario podemos seguir la documentación sobre la imagen que estamos usando, en este caso usamos la imagen de nextcloud y mariadb.

Podemos ver mas parametros y variables a configurar en la documentacion:
* Mariadb: [documentación de la imagen](https://hub.docker.com/_/mariadb)
* Nextcloud: [documentacion de la imagen](https://hub.docker.com/_/nextcloud)

```shell
#### Ejecutamos el fichero docker-compose.yml con podman-compose ####
vagrant@podman:~/.local/bin$ ./podman-compose -t 1podfw -p nextcloud -f /home/vagrant/Proyecto-integrado/podman/podman-compose/nextcloud/docker-compose.yml up 

#### Veremos que tenemos 3 contenedores ####
vagrant@podman:~/.local/bin$ podman ps -a --pod
CONTAINER ID  IMAGE                               COMMAND               CREATED             STATUS             PORTS                 NAMES               POD ID        PODNAME
3e037da4b25a  k8s.gcr.io/pause:3.2                                      About a minute ago  Up 10 seconds ago  0.0.0.0:8080->80/tcp  76046accd89d-infra  76046accd89d  nextcloud
6cf62600d918  docker.io/library/mariadb:latest    --transaction-iso...  About a minute ago  Up 9 seconds ago   0.0.0.0:8080->80/tcp  nextcloud_db_1      76046accd89d  nextcloud
f37c53968d2d  docker.io/library/nextcloud:latest  apache2-foregroun...  46 seconds ago      Up 9 seconds ago   0.0.0.0:8080->80/tcp  nextcloud_app_1     76046accd89d  nextcloud
```

Si entramos en http://ip-maquina:8080 veremos que nos saldra el instalador de nextcloud, seguimos los pasos y solo debemos entrar y veremos ya el dash:

![nextcloud con podman-compose](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Proyecto/captura-nextcloud-podman-compose.png)