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

Adicionalmente vamos a instalar tambien un procesador de ficheros **JSON** en linea de comandos llamado **jq** con el cual nos va a ser mas sencillo ver la informción de las imagenes ya que estas se nos presentan en formato **JSON**
```shell
#### Actualizamos la lista de paquetes e instalamos jq ####
vagrant@skopeo:~$ sudo apt update

vagrant@skopeo:~$ sudo apt install jq
```

## Primeros pasos

Como hemos visto, skopeo nos da la posibilidad de obtener mas información sobre las imagenes que tenemos en local o en registros publicos como **DockerHub**. Si necesitamos guiarnos sobre las opciones que tenemos podemos usar *--help*:
```shell
vagrant@skopeo:~$ skopeo --help
Operaciones con imagenes de contenedores y con registros de imagenes

Uso:
  skopeo [command]

Available Commands:
  copy                                          Copia una imagen de origen a destino(Ej: skopeo copy docker://registry.fedoraproject.org/fedora:latest dir:pruebas_imagen )
  delete                                        Elimina una imagen 
  help                                          Invocamos la ayuda de terminal
  inspect                                       Obtenemos información sobre una imagen
  list-tags                                     Listamos los tags de la imagen sobre un repositorios
  login                                         Nos logeamos en un registro de contenedores (Ej: Docker Hub)
  logout                                        Nos desconectamos de un registro de contenedores
  manifest-digest                               Compute a manifest digest of a file
  standalone-sign                               Create a signature using local files
  standalone-verify                             Verify a signature using local files
  sync                                          Sincroniza una o mas imagenes desde origen a destino

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
```

### Obtener información de imagenes

Skopeo nos da la posibilidad de *inspeccionar* repositorios de imagenes y puede recuperar las capas de esa imagen. Con la opción *inspect* nos da la posibilidad de obtener el manifiesto del repositorio y podemos mostrar la salida por pantalla en formato **JSON**, como podemos hacer tambien con **Docker inspect** o, en nuestro caso, con **Podman inspect**. La opción inspect puede mostrarnos qué etiquetas están disponibles para el repositorio dado, las etiquetas que tiene la imagen, la fecha de creación y el sistema operativo de la imagen y más.
```json
#### Inspeccionamos la imagen de wordpress en docker hub ####
vagrant@skopeo:~$ skopeo inspect docker://wordpress | jq
{
  "Name": "docker.io/library/wordpress",
  "Digest": "sha256:7caa73eb01258d36b2b1c7401ae37869ac4ef4fc4ef2aff91a19075b1e950923",
  "RepoTags": [
    "3.9.1",
    ...
    "php7.4-apache",
    "php8.0-fpm",
    "php8.0"
  ],
  "Created": "2021-05-25T00:23:22.141847003Z",
  "DockerVersion": "19.03.12",
  "Labels": null,
  "Architecture": "amd64",
  "Os": "linux",
  "Layers": [
    "sha256:69692152171afee1fd341febc390747cfca2ff302f2881d8b394e786af605696",
    "sha256:2040822db3250a09a3fee8c4e36a2a87d925b04776a46ea33dd94c111d5af366",
    "sha256:9b4ca5ae9dfad3ef8929b7c016d54518c9d29b6da520c7728e17885936824da8",
    "sha256:ac1fe7c6d9669614ca12d3a7e70ba5aa1a348b8620e251245d7dbf01f8b927b8",
    "sha256:5b26fc9ce030104e3ed1dc2b6231067d9eaa629b129a71c9bc21aed211beb9dd",
    "sha256:3492f4769444367000d2c12c6b1482630c1a6df69440fa64ee6d34e6ce93f2ae",
    "sha256:1dec05775a74fc3e3e317e71f6b606242142211e697a35c9dcfcef9ea4887113",
    "sha256:9ed895bcea3373e0b11f55512f2c9b7a7debb74349c05c929573a359395e0318",
    "sha256:bcc491c2ddc4f4ce32ada9b3a1c40f82be8e298b3b7dd60509627be7b122a4dd",
    "sha256:8469abe55c178684ce64c5d4bd328014677a9a67e10d454f47a6624f3cfde7d5",
    "sha256:d74704ab90e5119dbcaffbc6448fe036b9ad293fb67979951208eb1efb222d5c",
    "sha256:2f6f8b7d4ce2252ea9fcb0239a8129b6b3e332966aff90fefc76179d4d997b53",
    "sha256:3407998a353d94569ec57e88f5ce7165977657cdda4dddd3bf8775b6f5d5d779",
    "sha256:310682739bc5e5c930436e40e3fe76c6e61286cdc92de4bcb48bbf0a0c47210e",
    "sha256:c82f18ace11b8d5c439414729b9aec565202c7aeaa34bbf490e8466a985fd428",
    "sha256:1a3b9d474dbe2bd85daf81485b4019e812fc381dbc9d297b5908c95595f6ce0f",
    "sha256:97060a6e7f081f2ce133fa1ae0ad31a3a5b6ebc2650eb00f500e45cff3b2ebc0",
    "sha256:61f5c6c4243b37af3a3c0ea23b86251e2490a4dbfab62dc6ee665bc1d419050b",
    "sha256:9d18afd88d9835b61728472261cc88f75d530281886b1afaa0a43b3fd951d31b",
    "sha256:2c0d01114559864deda89d2465084a71e31b4ddc6ffd3afcf95d93216f581e88",
    "sha256:2791c3bb8577da9da7212c7d074e99eb26c1fc5655bbce0bba19c0953e39bfd3"
  ],
  "Env": [
    "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
    "PHPIZE_DEPS=autoconf \t\tdpkg-dev \t\tfile \t\tg++ \t\tgcc \t\tlibc-dev \t\tmake \t\tpkg-config \t\tre2c",
    "PHP_INI_DIR=/usr/local/etc/php",
    "APACHE_CONFDIR=/etc/apache2",
    "APACHE_ENVVARS=/etc/apache2/envvars",
    "PHP_EXTRA_BUILD_DEPS=apache2-dev",
    "PHP_EXTRA_CONFIGURE_ARGS=--with-apxs2 --disable-cgi",
    "PHP_CFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64",
    "PHP_CPPFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64",
    "PHP_LDFLAGS=-Wl,-O1 -pie",
    "GPG_KEYS=42670A7FE4D0441C8E4632349E4FDC074A4EF02D 5A52880781F755608BF815FC910DEB46F53EA312",
    "PHP_VERSION=7.4.19",
    "PHP_URL=https://www.php.net/distributions/php-7.4.19.tar.xz",
    "PHP_ASC_URL=https://www.php.net/distributions/php-7.4.19.tar.xz.asc",
    "PHP_SHA256=6c17172c4a411ccb694d9752de899bb63c72a0a3ebe5089116bc13658a1467b2"
  ]
}
```

Y a continuación vamos a ver la configuración de la misma imagen añadiendo *--config*:
```json
vagrant@skopeo:~$ skopeo inspect --config docker://wordpress | jq
{
  "created": "2021-05-25T00:23:22.141847003Z",
  "architecture": "amd64",
  "os": "linux",
  "config": {
    "ExposedPorts": {
      "80/tcp": {}
    },
    "Env": [
      "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
      "PHPIZE_DEPS=autoconf \t\tdpkg-dev \t\tfile \t\tg++ \t\tgcc \t\tlibc-dev \t\tmake \t\tpkg-config \t\tre2c",
      "PHP_INI_DIR=/usr/local/etc/php",
      "APACHE_CONFDIR=/etc/apache2",
      "APACHE_ENVVARS=/etc/apache2/envvars",
      "PHP_EXTRA_BUILD_DEPS=apache2-dev",
      "PHP_EXTRA_CONFIGURE_ARGS=--with-apxs2 --disable-cgi",
      "PHP_CFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64",
      "PHP_CPPFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64",
      "PHP_LDFLAGS=-Wl,-O1 -pie",
      "GPG_KEYS=42670A7FE4D0441C8E4632349E4FDC074A4EF02D 5A52880781F755608BF815FC910DEB46F53EA312",
      "PHP_VERSION=7.4.19",
      "PHP_URL=https://www.php.net/distributions/php-7.4.19.tar.xz",
      "PHP_ASC_URL=https://www.php.net/distributions/php-7.4.19.tar.xz.asc",
      "PHP_SHA256=6c17172c4a411ccb694d9752de899bb63c72a0a3ebe5089116bc13658a1467b2"
    ],
    "Entrypoint": [
      "docker-entrypoint.sh"
    ],
    "Cmd": [
      "apache2-foreground"
    ],
    "Volumes": {
      "/var/www/html": {}
    },
    "WorkingDir": "/var/www/html",
    "StopSignal": "SIGWINCH"
  },
  "rootfs": {
    "type": "layers",
    "diff_ids": [
      "sha256:02c055ef67f5904019f43a41ea5f099996d8e7633749b6e606c400526b2c4b33",
      "sha256:d2252dfd3726823303a8f54483e10f3ed7959f33ba63ea3a19bd0fc46587bc8f",
      "sha256:6c63b3f282cb455ac2e456fb99cdceab42a876603b54cc97087c17b9999b780b",
      "sha256:8e859e616115511355eadb05cbdcf86333434621f77c3ac80c410257ea39f115",
      "sha256:652f02132f2db1e1e65572f6bef8cccfeb879e9fc4b88662dad574154145bc00",
      "sha256:f8b772487fdc0a3dc28c00f882637c1dc77be22c041aa9a966b786ba4eda59b5",
      "sha256:75426ca6aa9c06f3f5eef14ec5f47da06035ab266b0e36b4401585b00c2682b3",
      "sha256:8a1d55fcf58ea2354eaa038197ef356872c9a565b79b281fd24806bc36666400",
      "sha256:4e0b8c77bcc3f10eb0046faa0347db5067d5e499b9547a2d21fbda8457476ead",
      "sha256:b63d491f00b6bf15d36d8518f32e6acf6da524c170eafd15784763d27f402341",
      "sha256:191e1c00984b32fc3aa9f1c54f4d774c67ce35799cc55b8836a7fdbf76ddff84",
      "sha256:ed2b71ab32f37d35d38c91e1c265872e499ecdf7bc6c3362b8fc2a3a486a2a73",
      "sha256:9d36eae86a28026ea9ece02220cdf2630b1492469c7488c69e2d601c09866940",
      "sha256:97a54d9096e38763bec8b13cba0639aab5f678142f0298a7207176861997cf4a",
      "sha256:ae32d5fd238e3c0111a89c7173edf2639cb5c6ac5c0b3041674e9392d7e2375c",
      "sha256:0934026db75c35abec48a0996713485e84914a98d06ea4440fd9845d1a2a5280",
      "sha256:d4fbc135b31a935df6083b07d2f1a881d9a366f032b58f6946b4bbdf6239fdf3",
      "sha256:ba81158cca039568ad72421987d2afbc4741440821d0f11cf3261d8fd79234a2",
      "sha256:7a22b82077d1fffc8ea88e9a899e949dcbc74e1aab667e9c8d8915b4232174e7",
      "sha256:9d205b44b84d24405a2424614ea837186135fbfd811b6f43504e4ff44cdf914e",
      "sha256:228a987eb0faf022977b382ab11fa98c16ca5456eb0d1dc21536f5950b49ace0"
    ]
  },
  "history": [
    {
      "created": "2021-05-12T01:21:22.128649612Z",
      "created_by": "/bin/sh -c #(nop) ADD file:7362e0e50f30ff45463ea38bb265cb8f6b7cd422eb2d09de7384efa0b59614be in / "
    },
    {
      "created": "2021-05-12T01:21:22.552826465Z",
      "created_by": "/bin/sh -c #(nop)  CMD [\"bash\"]",
      "empty_layer": true
    }
  ]
}
```

### Copiando imagenes

A la hora de copiar imagenes con **skopeo**, tenemos varias posibilidades a la hora de copiar estas imagenes, eligiendo entre multiples mecanismos de almacenamiento:

* Registro de contenedores:
  * Como puede ser **The Quay, Docker Hub, OpenShift, GCR, Artifactory ...**

* Almacenamiento *backends*:
  * Almacenamiento del demonio de docker

* Almacenamiento en directorios locales

* Directorios locales **OCI-layout**

```shell
#### Copiamos del registro de docker a un directorio local ####
vagrant@skopeo:~/imagenes/wordpress$ skopeo copy docker://wordpress dir:./
Getting image source signatures
Copying blob 69692152171a done  
Copying blob 2040822db325 done  
Copying blob 9b4ca5ae9dfa done  
Copying blob ac1fe7c6d966 done  
Copying blob 5b26fc9ce030 done  
...

vagrant@skopeo:~/imagenes/wordpress$ ls
0adda6ed742f6c79261fa2a49e24e5909be720a9b144c1ffb0431262b8bae9c8  3407998a353d94569ec57e88f5ce7165977657cdda4dddd3bf8775b6f5d5d779  9d18afd88d9835b61728472261cc88f75d530281886b1afaa0a43b3fd951d31b
1a3b9d474dbe2bd85daf81485b4019e812fc381dbc9d297b5908c95595f6ce0f  3492f4769444367000d2c12c6b1482630c1a6df69440fa64ee6d34e6ce93f2ae  9ed895bcea3373e0b11f55512f2c9b7a7debb74349c05c929573a359395e0318
1dec05775a74fc3e3e317e71f6b606242142211e697a35c9dcfcef9ea4887113  5b26fc9ce030104e3ed1dc2b6231067d9eaa629b129a71c9bc21aed211beb9dd  ac1fe7c6d9669614ca12d3a7e70ba5aa1a348b8620e251245d7dbf01f8b927b8
2040822db3250a09a3fee8c4e36a2a87d925b04776a46ea33dd94c111d5af366  61f5c6c4243b37af3a3c0ea23b86251e2490a4dbfab62dc6ee665bc1d419050b  bcc491c2ddc4f4ce32ada9b3a1c40f82be8e298b3b7dd60509627be7b122a4dd
2791c3bb8577da9da7212c7d074e99eb26c1fc5655bbce0bba19c0953e39bfd3  69692152171afee1fd341febc390747cfca2ff302f2881d8b394e786af605696  c82f18ace11b8d5c439414729b9aec565202c7aeaa34bbf490e8466a985fd428
2c0d01114559864deda89d2465084a71e31b4ddc6ffd3afcf95d93216f581e88  8469abe55c178684ce64c5d4bd328014677a9a67e10d454f47a6624f3cfde7d5  d74704ab90e5119dbcaffbc6448fe036b9ad293fb67979951208eb1efb222d5c
2f6f8b7d4ce2252ea9fcb0239a8129b6b3e332966aff90fefc76179d4d997b53  97060a6e7f081f2ce133fa1ae0ad31a3a5b6ebc2650eb00f500e45cff3b2ebc0  manifest.json
310682739bc5e5c930436e40e3fe76c6e61286cdc92de4bcb48bbf0a0c47210e  9b4ca5ae9dfad3ef8929b7c016d54518c9d29b6da520c7728e17885936824da8  version
```

