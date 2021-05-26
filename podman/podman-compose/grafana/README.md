Para este escenario podemos seguir la documentación sobre la imagen que estamos usando, en este caso usamos la imagen de grafana.

Podemos ver mas parametros y variables a configurar en la documentacion:
* Grafana: [documentación de la imagen](https://hub.docker.com/r/bitnami/grafana)

```shell
#### Ejecutamos el fichero docker-compose.yml con podman-compose ####
vagrant@podman:~/.local/bin$ ./podman-compose -t 1podfw -p grafana -f /home/vagrant/Proyecto-integrado/podman/podman-compose/grafana/docker-compose.yml up 

#### Veremos que tenemos 3 contenedores ####
vagrant@podman:~/.local/bin$ podman ps -a --pod
CONTAINER ID  IMAGE                        COMMAND  CREATED             STATUS            PORTS                   NAMES               POD ID        PODNAME
b5d5f3ae0b3e  k8s.gcr.io/pause:3.2                  About a minute ago  Up 6 seconds ago  0.0.0.0:3000->3000/tcp  206518765e68-infra  206518765e68  grafana
91c75f8d41b1  docker.io/bitnami/grafana:7           33 seconds ago      Up 5 seconds ago  0.0.0.0:3000->3000/tcp  grafana_grafana_1   206518765e68  grafana
```

Ahora si entramos en http://ip-maquina:3000 veremos el login de grafana que, por defecto es admin/admin pero se pueden cambiar esos parametros en el fichero de *docker-compose.yml*:

![imagen de grafana sobre podman-compose](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Proyecto/captura-grafana-podman-compose.png)