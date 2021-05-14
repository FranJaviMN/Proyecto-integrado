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

## Pods con Podman

### ¿Qué es un Pod?

Antes de empezar vamos a ver que es un pod.

Debemos de tener en cuneta que los pods es un concepto que introdució Kubernetes pero el concepto en Podman es muy similar al que maneja Kubernetes. Un Pod (como en una vaina de ballenas o vaina de guisantes) es un grupo de uno o más contenedores (como contenedores Docker o Podman), con almacenamiento/red compartidos, y unas especificaciones de cómo ejecutar los contenedores. Los contenidos de un Pod son siempre coubicados, coprogramados y ejecutados en un contexto compartido y este contiene uno o más contenedores de aplicaciones relativamente entrelazados.

El contexto compartido de un Pod es un conjunto de namespaces de Linux, cgroups y, potencialmente, otras facetas de aislamiento, las mismas cosas que aíslan un contenedor Docker o Podman. Dentro del contexto de un Pod, las aplicaciones individuales pueden tener más subaislamientos aplicados. Los contenedores dentro de un Pod comparten dirección IP y puerto, y pueden encontrarse a través de localhost. También pueden comunicarse entre sí mediante comunicaciones estándar entre procesos. Los contenedores en diferentes Pods tienen direcciones IP distintas y no pueden comunicarse por IPC sin configuración especial. Estos contenedores normalmente se comunican entre sí a través de las direcciones IP del Pod.

En términos de Docker, un Pod se modela como un grupo de contenedores de Docker con namespaces y volúmenes de sistemas de archivos compartidos.

![Imagen de un pod](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/a7c12c4da2fedd5e8a4c7f5bd9ddc2b01dab9774/Proyecto/pod.svg)

Un Pod de múltiples contenedores que contiene un extractor de archivos y un servidor web que utiliza un volumen persistente para el almacenamiento compartido entre los contenedores.

## Concepto de pod en Podman

Ahora, si hablamos de los Pods en Podman los cuales, son similares al concepto que tiene Kubernetes sobre estos, podemos fijarnos en este esquema:

![Pods en podman](https://raw.githubusercontent.com/FranJaviMN/elementos-grado/main/Proyecto/podman-pod-architecture.png)

Todos los pods que vamos a crear con Podman vamos a tener un container "*Infra*". Este contenedor no hace nada. El proposito es el de sostener los *namespaces* asociados en el pod y permite que podman conecte con otros contenedores del pod. Esto le permite iniciar y detener contenedores dentro del pod y el pod seguirá funcionando donde, si el contenedor primario controlara el pod, esto no sería posible. Por defecto, el contenedor "*infra*" esta basado en la imagen **k8s.gcr.io/pause**. A menos que diga explícitamente lo contrario, todos los pods tendrán un contenedor basado en la imagen predeterminada.


## CLI: podman pod

Para ello vamos a tener la siguiente guia sobre los comandos:
```shell
#### Comando por defecto para tratar con pods en podman ####

podman pod - Gestiona los pods.

Los pods son un grupo de uno o varios contenedores compartiendo la misma red, PID y espacio de nombres.

USO:
  podman pod comando [comando opciones] [argumentos...]

COMANDOS:
  create        Crea un nuevo pod vacio.
  exists        Comprueba si el pod existe
  inspect       Saca por pantalla la información acerca del pod
  kill          Envia una señal "kill" al pod
  pause         Pausa uno o mas pods
  ps, ls, list  Lista los pods 
  restart       Reinicia uno o mas pods
  rm            Elimina uno o mas pods
  start         Inicia uno o mas pods
  stats         Saca por pantalla el consumo de CPU, memory, network I/O, block I/O and PIDs de los contenedores en uno o mas pods
  stop          Para uno o mas pods
  top           muestra por pantalla los procesos de los contenedores en un pod
  unpause       Quita de pausa uno o mas pods

OPCIONES:
  --help, -h Muestra las opciones de los comandos. Ej: podman pod create --help
```

### Creación de pods

La manera mas sencilla que tenemos con podman para crear un pod, como hemos visto en los comandos anteriores, es usando **podman pod create**:
```shell

#### Comando para la creación de pods ####

podman pod create - Crea un nuevo pod vacio

USO: 
  podman pod create [comando opciones] [argumentos...]

OPCIONES:
  --cgroup-parent [value]      Set parent cgroup for the pod
  --infra                    Crea un contenedor infra en el nuevo pod
  --infra-command [value]      Comando que se va a ejecutar en el contenedor infra cuando el pod se inicie (default: "/pause")
  --infra-image [value]        Le indicamos cual es la imagen del contendor "infra" que va a usar (Por defecto: "k8s.gcr.io/pause:3.1")
  --label [value], -l [value]    Fija los metadatos en el pod (default [])
  --label-file [value]         Lee en un archivo las etiquetas delimitadas por lineas (default [])
  --name [value], -n [value]     Asigan un nombre al contenedor, si no se le añade, le da uno aleatorio
  --pod-id-file [value]        Inyecta el ID del pod en un fichero
  --publish [value], -p [value]  Indicamos puertos o un rango de puertos en el host (default [])
  --share [value]              Una lista de los namespaces que el pod va a usar, separado por comas (default: "cgroup,ipc,net,uts")
```

Ahora que hemos visto un poco las opciones que ofrece podman para crear pods, vamos a empezar por lo mas basico que es la creacion de un pod "vacio" para ir viendo todo lo que podemos ir haciendo con este pod.

Lo primero que debemos de hacer es crear el pod, que le vamos a da un nombre concreto, para ello vamos a usar el siguiente comando:
```shell
#### Creamos el pod con el nombre "pod_prueba" ####
vagrant@podman:~$ podman pod create --name pod_prueba
b11f104a9e60af2339f297cd33ba008239b3c4c8e91ba041a61386b35ecf968d
```

Como vemos, al crear tambien nos saca por pantalla el id de ese pod que tambien podriamos usar a parte del nombre que le hemos pues al pod. Ahora mismo este pod que hemos creado esta vacio y, como hemos visto anteriormente, solo contiene el contenedor "infra":
```shell
#### Listamos los pods que tenemos creados  ####
vagrant@podman:~$ podman pod list
POD ID        NAME        STATUS   CREATED         INFRA ID      # OF CONTAINERS
b11f104a9e60  pod_prueba  Created  35 seconds ago  50747b98c542  1

#### Comprobamos los contenedores que tenemos dentro del pod ####
vagrant@podman:~$ podman ps -a --pod
CONTAINER ID  IMAGE                 COMMAND  CREATED         STATUS   PORTS   NAMES               POD ID        PODNAME
50747b98c542  k8s.gcr.io/pause:3.2           52 seconds ago  Created          b11f104a9e60-infra  b11f104a9e60  pod_prueba
```

### Añadir nuevos contenedores a un Pod

Visto la creacion de un pod "vacio" ahora vamos a proceder a añadir un contenedor a este pod, para ello debemos de tener claro a que pod vamos a agregar el nuevo contenedor por lo que, o bien usamos el id del pod o usamos su nombre.

En estos ejemplos vamos a añadir un nuevo contenedor que va a correr con una imagen de debian 10 y como proceso va a correr un "top" y un "ls":
```shell
#### Creamos un contenedor en el pod de "prueba_pod ####
vagrant@podman:~$ podman run -dt --pod b11f104a9e60 docker.io/library/alpine:latest top
2a67924007260c8f663fda6bd7734b098e317774b6f59fa2e57cadef9b48ab01

#### Listamos los contenedores del pod ####
vagrant@podman:~$ podman ps -a --pod
CONTAINER ID  IMAGE                            COMMAND  CREATED             STATUS                    PORTS   NAMES                 POD ID        PODNAME
50747b98c542  k8s.gcr.io/pause:3.2                      About a minute ago  Up 18 seconds ago                 b11f104a9e60-infra    b11f104a9e60  pod_prueba
2a6792400726  docker.io/library/alpine:latest  top      18 seconds ago      Up 17 seconds ago                 recursing_pascal      b11f104a9e60  pod_prueba
```

Como vemos tenemos el nuevo contenedor en el pod, con un nombre aleatorio y se esta ejecutando el comando top, que nos muestra los procesos y carga de la maquina por lo que el contenedor se va a quedar levantado hasta que yo le diga lo contrario.

En cambio si lanzamos un contenedor en el pod que solo ejecute un "ls" veremos que al instante el contenedor se para y ya no esta en funcionamiento porque los contenedores solo ejecutan un proceso, cuando este termina, el contenedor tambien:
```shell
#### Creamos un contenedor con debian y ejecutamos "ls" ####
vagrant@podman:~$ podman run -dt --pod b11f104a9e60 docker.io/library/debian:latest ls
3a6171279b402b588a952f277a8e19efab390c0b39441742f9c0b09e9626ce54

#### Comprobamos los contenedores de pod ####
vagrant@podman:~$ podman ps -a --pod
CONTAINER ID  IMAGE                            COMMAND  CREATED             STATUS                    PORTS   NAMES                 POD ID        PODNAME
50747b98c542  k8s.gcr.io/pause:3.2                      About a minute ago  Up 18 seconds ago                 b11f104a9e60-infra    b11f104a9e60  pod_prueba
2a6792400726  docker.io/library/alpine:latest  top      18 seconds ago      Up 17 seconds ago                 recursing_pascal      b11f104a9e60  pod_prueba
3a6171279b40  docker.io/library/debian:latest  ls       3 seconds ago       Exited (0) 3 seconds ago          ecstatic_stonebraker  b11f104a9e60  pod_prueba
```

Como vemos, se ha creado y al tiempo de terminar el proceso de ls, este contenedor se ha parado.