# Skopeo

Cuando hablamos de contenedores tambien podemos hablar de las imagenes que se usan para la creación de estos, las imagenes normalmente las descargamaos desde registros como son [Docker Hub](https://hub.docker.com/) o sobre registros privados creados en local. 

Si queremos ver que tipo de imagenes tenemos en estos registros, ver si la imagen existe, obtener informacion sobre una imagen... Para ello tenemos la herramienta de **Skopeo** que es una utilidad línea de comandos que realiza varias operaciones en imágenes de contenedores y registros de imágenes. Con **Skopeo** podemos gestionar las imágenes generadas por Buidah y subirlas a diferentes registros.

## Sobre Skopeo

Una de las principales ventajas que tiene **Skopeo** es que puede ser ejecutado sin ser usuario root al igual que pasa con podman y tambien, como caracteristica principal es que no tiene ningun *demonio* ejecutandose para su funcionamiento. 

Como caracteristica sobre la compatibilidad de Skopeo con imagenes, este es compatibles con imagenes de tipo **OCI**(Open Container Initiative) y con imagenes de docker v2.

Teniendo en cuenta estas caracteristicas, Skopeo tiene las siguientes operaciones:

* Copiar una imagen desde y hacia varios mecanismos de almacenamiento. Por ejemplo, podemos copiar imágenes de un registro a otro, sin necesidad de privilegios.

* Inspeccionar una imagen remota para que muestre sus propiedades, incluidas sus capas, sin que sea necesario que se descargue la imagen

* Eliminar una imagen de un registro que nosotros le indiquemos, ya sea privado o en docker hub.

* Sincronizar un repositorio de imágenes externo con un registro interno.

* Cuando lo requiera el repositorio, Skopeo puede pasar las credenciales y certificados apropiados para la autentificación.

## Escenario para Skopeo

Para ello vamos a usar en una maquina independiente creada con vagrant mediante el siguiente fichero:
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |skopeo|
  skopeo.vm.box = "generic/debian10"
  skopeo.vm.hostname = "skopeo"
  skopeo.vm.synced_folder ".", "/vagrant", disabled: true
  skopeo.vm.provider :libvirt do |libvirt|
    libvirt.uri = 'qemu+unix:///system'
    libvirt.host = "skopeo"
    libvirt.cpus = 1
    libvirt.memory = 1024
  end
end
```

### Instalación en debian 10

Para ello, en debian 10 aun no se encuentra en repositorios el paquete de **Skopeo** que es el que queremos instalar, para ello debemos de añadir los repositorios de backports:
```shell
#### Añadimos los repositorios de backports ####
root@skopeo:/home/vagrant# echo 'deb http://deb.debian.org/debian buster-backports main' >> /etc/apt/sources.list
root@skopeo:/home/vagrant# echo 'deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/Debian_10/ /' > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list

#### Descargamos la clave de los repositorios ####
(Instalar si no se tiene el paquete de gnupg)
vagrant@skopeo:~$ sudo apt install gnupg

vagrant@skopeo:~$ curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/Debian_10/Release.key | sudo apt-key add -
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1093  100  1093    0     0   3149      0 --:--:-- --:--:-- --:--:--  3140
OK

#### Actualizamos la lista de paquetes y descargamos Skopeo ####
vagrant@skopeo:~$ sudo apt update

vagrant@skopeo:~$ sudo apt install -t buster-backports skopeo
```

De esta forma ya tendriamos **Skopeo** en nuestra maquina de pruebas instalado, si queremos instalarlo en otra distribución que no sea debian podemos seguir los pasos del siguiente [enlace](https://github.com/containers/skopeo/blob/master/install.md)

## Primeros pasos

Como hemos visto, skopeo nos da la posibilidad de obtener mas información sobre las imagenes que tenemos en local o en registros publicos como **DockerHub**. Si necesitamos guiarnos sobre las opciones que tenemos podemos usar *--help*:
```shell
vagrant@skopeo:~$ skopeo --help
Operaciones con imagenes de contenedores y con registros de imagenes

Uso:
  skopeo [command]

Available Commands:
  copy                                          Copia una imagen de origen a destino(Ej: skopeo copy docker://registry.fedoraproject.org/fedora:latest dir:pruebas_imagen )
  delete                                        Elimina una imagen ()
  help                                          Help about any command
  inspect                                       Inspect image IMAGE-NAME
  list-tags                                     List tags in the transport/repository specified by the REPOSITORY-NAME
  login                                         Login to a container registry
  logout                                        Logout of a container registry
  manifest-digest                               Compute a manifest digest of a file
  standalone-sign                               Create a signature using local files
  standalone-verify                             Verify a signature using local files
  sync                                          Synchronize one or more images from one location to another

Flags:
      --command-timeout duration   timeout for the command execution
      --debug                      enable debug output
  -h, --help                       help for skopeo
      --insecure-policy            run the tool without any policy check
      --override-arch ARCH         use ARCH instead of the architecture of the machine for choosing images
      --override-os OS             use OS instead of the running OS for choosing images
      --override-variant VARIANT   use VARIANT instead of the running architecture variant for choosing images
      --policy string              Path to a trust policy file
      --registries.d DIR           use registry configuration files in DIR (e.g. for container signature storage)
      --tmpdir string              directory used to store temporary files
  -v, --version                    Version for Skopeo

Use "skopeo [command] --help" for more information about a command.
