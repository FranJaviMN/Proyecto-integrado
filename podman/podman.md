# Contenedores podman

En el siguiente post vamos a ver como podemos encontrar una alternativa a los contenedores **Docker** sin necesidad de tener que aprender nuevos comandos y sintaxis, para ello, la solución son los contenedores **Podman**

## ¿Qué son los contenedores Podman?

Lo primero de lo que vamos a hablar es de donde surgio este motor de contenedores, sus caracteristicas mas representativas y las diferencias que podemos ver con su rival mas directo, **Docker**.

Podman es la abreviatura de *Pod Manager* y fue desarrollado por la empresa **Red Hat** en el año 2018. Este motor fue diseñado en la experiencia de **Docker**, pero en un principio, **Podman** fue desarrollado como herramienta de depuración para **CRI-O**(infraestructura que usa kubernetes para usar cualquier tiempo de ejecución compatible con **OCI** como tiempo de ejecución del contenedor para ejecutar pods.) por lo que no estaba previsto que este fuera un nuevo motor de contenedores.

Este motor de contenedores **Podman** tiene una gran cantidad de similitudes con su competidor **Docker** e incluso, podemos encotrar gente que lo que realiza es una instalación de **Podman** en linux y a continuación crea un alias de esta forma:
```shell
alias docker=podman
```

Obviamente es una broma que uno de los ingenrios de **Red Hat** hizo a la hora de prensentar **Podman** lo que nos hace entender que la migración a Podman es muy sencilla, ya que los ingenieros de RedHat han prestado especial atención en usar la misma nomenclatura a la hora de ejecutar comandos de Podman. Por lo que, si somos un usuario de Docker, ya conocemos la mayoría de los comandos en Podman. Así que, efectivamente, es tan sencillo como, en lugar de ejecutar **docker run**, ejecutar **podman run**, y el resultado será exactamente el mismo.

Por lo que vemos las diferencias que hay con **Docker** parecen ser minimas, pero su caracteristica mas representativa en lo liviano que es en comparación con **Docker** ya que **Podman**, carece de un *demonio central* para cada uno de los contenedores, en su lugar, han descentralizado todos los componentes necesarios para la gestión de contenedores y los han individualizado en componentes más pequeños que se utilizarán solo cuando sean necesarios. Esta descentralización nos ofrece un gran número de ventajas como el uso de esta herramienta sin la necesidad del uso del **root**.

## Funciones principales de Podman

Ademas de que **Podman** no hace uso de un *demonio central*, otra caracterista muy importante de **Podman** es la posibilidad que tenemos de ejecutar **pods**(son la fusión de varios contenedores en un namespace común de Linux que comparten recursos concretos. De esta manera, se pueden combinar una amplia variedad de aplicaciones virtualizadas.) al igual que hacemos con **Kubernetes** porque, como bien hemos visto, principalmente se desarrolló teniendo en mente que estaria enfocado en **Kubernetes**. Por lo que vemos que con **Podman** podemos hacer todas las cosas que podemos hacer con **Docker** pero tambien nos brinda la posibilidad de la ejecución de **pods** como hacemos en **Kubernetes**

Ademas, mencionado anteriormente, el uso de **Podman no requiere privilegios de root por lo que puede ser ejecutado por un usuario normal**. Aunque si es verdad que dentro de los contenedores de **Podman** todo y cada uno de los procesos **se ejecutan bajo root** y esto es posible ya que, podman hace uso de de los **namespacede usuario del nucleo linux**, que asigna privilegios especiales y un ID de usuario a los procesos. El hecho de que los contenedores en realidad se ejecuten como administrador dota al entorno virtualizado Podman de un elevado estándar de seguridad.

## Introducción a Podman

Visto todo lo que es **Podman** y sus caracteristicas, a continuación vamos a usarlo para ver su potencial, para ello vamos a hacer uso de una maquina virtual creada en **Vagrant** usando **libvirt**. El fichero **Vagrantfile** lo podemos encontrar a continuación:
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |podman|
  podman.vm.box = "generic/debian10"
  podman.vm.hostname = "podman"
  podman.vm.synced_folder ".", "/vagrant", disabled: true
  podman.vm.provider :libvirt do |libvirt|
    libvirt.uri = 'qemu+unix:///system'
    libvirt.host = "podman"
    libvirt.cpus = 1
    libvirt.memory = 512
  end
end
```

Una vez desplegada nuestra maquina virtual ya podemos empezar a trastear con **Podman**.

### Instalación de podman

Lo primero que debemos de hacer, como es obvio, es instalar el motor de contenedores **Podman** en nuestra maquina virtual, para ello y, siguiendo la documentación oficial, vamos a instalarlo en nuestro sistema, en este caso **Debian 10**:
```shell
#### Debemos de habilitar en espacio de nombre de root ####
echo 'kernel.unprivileged_userns_clone=1' > /etc/sysctl.d/00-local-userns.conf
systemctl restart procps

#### Añadimos los repositorios de backports de Debian 10 ####
echo 'deb http://deb.debian.org/debian buster-backports main' >> /etc/apt/sources.list
echo 'deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/Debian_10/ /' > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list

#### Descargamos e instalamos la clave de los repositorios ####
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/Debian_10/Release.key | sudo apt-key add -

#### Actualizamos la lista de paquetes e instalamos dependencias desde backports ####
sudo apt update
sudo apt -y -t buster-backports install libseccomp2

#### Instalamos podman ####
sudo apt -y install podman
```

Si nuestro sistema es diferente a **Debian 10** en la [documentación oficial](https://podman.io/getting-started/installation) podemos encontrar la forma de instalarlo en nuestro sistema.

### Primer contenedor Podman: Hello-world

Una vez ya tenemos instalado nuestro motor de contenedores, vamos a empezar a crear contenedores para ver su potencial y similitudes respecto a **Docker**.

Para ello vamos a hacer uso de la imagen llamada **Hello-world** que, por defecto, va a descargar desde **docker.io**.

Podemos hacerlo de dos maneras, o bien la descargamos antes de crear el contenedor o directamente creamos el contenedor y podman se encarga de buscar la imagen y bajarla para crear el contenedor:
```shell
#### Bajamos la imagen desde dockerhub en la ultima versión ####
root@podman:~# podman pull hello-world:latest
Resolved "hello-world" as an alias (/etc/containers/registries.conf.d/000-shortnames.conf)
Trying to pull docker.io/library/hello-world:latest...
Getting image source signatures
Copying blob b8dfde127a29 done  
Copying config d1165f2212 done  
Writing manifest to image destination
Storing signatures
d1165f2212346b2bab48cb01c1e39ee8ad1be46b87873d9ca7a4e434980a7726

#### Creamos el contenedor ####
root@podman:~# podman run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
...
```

Una vez hemos creado la imagen debe de salirnos algo como lo anterior que, aunque ponga *hello from docker* estamos haciendo uso de **Podman**. Ahora que hemos creado el contenedor vamos a listar los contenedores para ver el que hemos creado:
```shell
#### Listamos los contenedores en ejecución ####
podman:~# podman ps
CONTAINER ID  IMAGE   COMMAND  CREATED  STATUS  PORTS   NAMES

#### Listamos todos los contenedores ####
root@podman:~# podman ps -a
CONTAINER ID  IMAGE                                 COMMAND  CREATED        STATUS                    PORTS   NAMES
6f3fb0ecd405  docker.io/library/hello-world:latest  /hello   3 minutes ago  Exited (0) 3 minutes ago          nifty_franklin
```

Comprobamos que este contenedor no se está ejecutando. **Un contenedor ejecuta un proceso y cuando termina la ejecución, el contenedor se para.**

Si queremos eliminar un contenedor que tenemos creado, es importante saber que debe de estar inactivo antes de poder eliminarlo, de lo contrario no nos dejara eliminar dicho contenedor:
```shell
#### Paramos el contenedor si estviera en ejecución ####
root@podman:~# podman stop 6f3fb0ecd405
6f3fb0ecd4053de87eda0644f54e0d1ebe86c0bd020f3c6a16e8d2150114e3fe

#### Lo borramos ####
root@podman:~# podman rm 6f3fb0ecd405
6f3fb0ecd4053de87eda0644f54e0d1ebe86c0bd020f3c6a16e8d2150114e3fe
```

Si queremos saber mas acerca de **Podman** podemos seguir los post que tengo en esta misma pagina sobre **Docker** que si bien esta enfocado a Docker, hemos visto que sus comandos son los mismos y sus funciones tambien lo son, por lo que solo debemos de cambiar la palabra *Docker* por la palabra *Podman*:

* Introducción a Docker: https://franjavimn.onrender.com/implantacion/introduccion-docker/

* Almacenamiento en Docker: https://franjavimn.onrender.com/implantacion/almacenamiento-docker/

### Crando nuestro primer pod

Una vez hemos visto como crear contenedores y todo lo relacionado con ello mediante **Podman**, debemos de recordar que este nos daba la opción de poder crear tambien pods, por lo que vamos a empezar a crearlos para ir viendo como funcionan estos con podman

