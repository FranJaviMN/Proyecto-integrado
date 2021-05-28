Para este escenario podemos seguir la documentación sobre la imagen que estamos usando, en este caso usamos la imagen de wordpress y de mariadb.

Podemos ver mas parametros y variables a configurar en la documentacion:
* Mariadb: [documentación de la imagen](https://hub.docker.com/_/mariadb)
* Wordpress: [Documentación de la imagen](https://hub.docker.com/_/wordpress)

```shell
#### Ejecutamos el fichero docker-compose.yml con podman-compose ####
vagrant@podman:~/.local/bin$ ./podman-compose -t 1podfw -p wp_compose -f /home/vagrant/Proyecto-integrado/podman/podman-compose/wordpress/docker-compose.yml up 

#### Veremos que tenemos 3 contenedores ####
vagrant@podman:~/.local/bin$ podman ps -a --pod
CONTAINER ID  IMAGE                               COMMAND               CREATED             STATUS             PORTS                 NAMES               POD ID        PODNAME
ed3a940dd61f  k8s.gcr.io/pause:3.2                                      2 minutes ago       Up 44 seconds ago  0.0.0.0:8080->80/tcp  ee6df39a4ee5-infra  ee6df39a4ee5  wp_compose
94b706df3c83  docker.io/library/wordpress:latest  apache2-foregroun...  About a minute ago  Up 10 seconds ago  0.0.0.0:8080->80/tcp  servidor_wp         ee6df39a4ee5  wp_compose
8582bf54eca4  docker.io/library/mariadb:latest    mysqld                45 seconds ago      Up 11 seconds ago  0.0.0.0:8080->80/tcp  servidor_mysql      ee6df39a4ee5  wp_compose
```

Y ahora si entramos en http://ip-maquina:8080 veremos el instalador de wordpress para terminar de tenerlo listo para usar:

![wordpres podman-compose](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Proyecto/captura-worpress-podman-compose.png)