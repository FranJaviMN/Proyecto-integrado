Para este escenario podemos seguir la documentación sobre la imagen que estamos usando, en este caso usamos la imagen de prestashop y mariadb.

Podemos ver mas parametros y variables a configurar en la documentacion:
* Mariadb: [documentación de la imagen](https://hub.docker.com/_/mariadb)
* Prestashop: [documentacion de la imagen](https://hub.docker.com/r/prestashop/prestashop)

```shell
#### Ejecutamos el fichero docker-compose.yml con podman-compose ####
vagrant@podman:~/.local/bin$ ./podman-compose -t 1podfw -p presta -f /home/vagrant/Proyecto-integrado/podman/podman-compose/prestashop/docker-compose.yml up 

#### Veremos que tenemos 3 contenedores ####
vagrant@podman:~/.local/bin$ podman ps -a --pod
CONTAINER ID  IMAGE                                   COMMAND               CREATED        STATUS            PORTS                                        NAMES                   POD ID        PODNAME
2846fc21f98e  k8s.gcr.io/pause:3.2                                          5 minutes ago  Up 2 minutes ago  0.0.0.0:8082->80/tcp, 0.0.0.0:8443->443/tcp  8e8e983e887a-infra      8e8e983e887a  presta
2d8da104621a  docker.io/library/mariadb:latest        mysqld                5 minutes ago  Up 2 minutes ago  0.0.0.0:8082->80/tcp, 0.0.0.0:8443->443/tcp  presta_prestashop_db_1  8e8e983e887a  presta
701babaf993b  docker.io/prestashop/prestashop:latest  /tmp/docker_run.s...  4 minutes ago  Up 2 minutes ago  0.0.0.0:8082->80/tcp, 0.0.0.0:8443->443/tcp  presta_prestashop_1     8e8e983e887a  presta
```

Para ello entramos en http://ip-maquina:8082 y veremos el instalador de prestashop, solo seguimos los pasos y entramos en la pagina:

![imagen de prestashop podman-compose](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Proyecto/captura-pshop-podman-compose.png)