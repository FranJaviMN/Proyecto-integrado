Para este escenario podemos seguir la documentación sobre la imagen que estamos usando, en este caso usamos la imagen de redmine y la imagen de mysql.

Podemos ver mas parametros y variables a configurar en la documentacion:
* Redmine: [documentación de la imagen](https://hub.docker.com/_/redmine)
* MySQL: [Documentación de la imagen](https://hub.docker.com/_/mysql)

```shell
#### Ejecutamos el fichero docker-compose.yml con podman-compose ####
vagrant@podman:~/.local/bin$ ./podman-compose -t pod1fw -p redmine -f /home/vagrant/Proyecto-integrado/podman/podman-compose/redmine/docker-compose.yml up

#### Veremos que tenemos 3 contenedores ####
vagrant@podman:~/.local/bin$ podman ps -a --pod
CONTAINER ID  IMAGE                             COMMAND               CREATED             STATUS            PORTS                   NAMES               POD ID        PODNAME
43e012373645  k8s.gcr.io/pause:3.2                                    About a minute ago  Up 2 seconds ago  0.0.0.0:8080->3000/tcp  e6ffec6955cb-infra  e6ffec6955cb  redmine
a6791a84006a  docker.io/library/redmine:latest  rails server -b 0...  About a minute ago  Up 2 seconds ago  0.0.0.0:8080->3000/tcp  redmine_redmine_1   e6ffec6955cb  redmine
f49f86009b7e  docker.io/library/mysql:5.7       mysqld                About a minute ago  Up 2 seconds ago  0.0.0.0:8080->3000/tcp  redmine_db_1        e6ffec6955cb  redmine
```

Y si vamos a http://ip-maquina:8080/ tendremos lo siguiente, las credenciales son admin/admin.

![imagen de redmine](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Proyecto/redmine-podman.png)

