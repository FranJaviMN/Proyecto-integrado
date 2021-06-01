# Buildah

**Buildah** es una herramienta de linea de comandos que nos permite la *construcción* de imagenes para contenedores del tipo **OCI**(Open Container Initiative).

Buildah es una herramienta de creación que le permite controlar cómo se disponen las capas de imágenes y se accede a los datos durante las creaciones. Para utilizar este programa, no se requiere ser el usuario **root** por lo que, al igual que ocurre con podman, es una herramienta **rootless** y podemos usarla siendo usuarios normales del sistema. Buildah no incluye herramientas de creación dentro de la imagen misma, lo cual reduce el tamaño de las imágenes que creamos.

El paquete de **buildah** nos proporciona en linea de comandos una herramienta que nos permite:

* Crear un contenedor, ya sea desde cero o usando una imagen como punto de partida.
* Crear una imagen, ya sea desde un contenedor o mediante las instrucciones en un Dockerfile.
* Las imágenes se pueden crear en formato de imagen OCI o en el formato de imagen tradicional de docker.
* Montar el sistema de archivos raíz de un contenedor en funcionamiento para su manipulación.
* Desmontar el sistema de archivos raíz de un contenedor en funcionamiento.
* Usar el contenido actualizado del sistema de archivos raíz de un contenedor como una capa del sistema de archivos para crear una nueva imagen.
* Eliminar un contenedor o una imagen.
* cambiar el nombre de un contenedor.

# Buildah y Podman 

Buildah y Podman son dos proyectos complementarios de código abierto que están disponibles en la mayoría de las plataformas Linux y ambos proyectos residen en GitHub.com con Buildah [aquí](https://github.com/containers/buildah) y Podman [aquí](https://github.com/containers/podman). Tanto Buildah como Podman son herramientas de línea de comandos que funcionan con imágenes y contenedores de Open Container Initiative (OCI). Los dos proyectos se diferencian en su especialización.

Buildah se especializa en la construcción de imágenes OCI. Los comandos de Buildah replican todos los comandos que se encuentran en un Dockerfile. Esto permite crear imágenes con y sin Dockerfiles sin requerir ningún privilegio de root. El objetivo final de Buildah es proporcionar una interfaz coreutils de nivel inferior para crear imágenes. La flexibilidad de crear imágenes sin Dockerfiles permite la integración de otros lenguajes de scripting en el proceso de creación. Buildah sigue un modelo simple de fork-exec y no se ejecuta como un demonio, sino que se basa en una API completa en golang, que se puede integrar con otras herramientas.

Podman se especializa en todos los comandos y funciones que le ayudan a mantener y modificar imágenes OCI, como extraer y etiquetar. También le permite crear, ejecutar y mantener los contenedores creados a partir de esas imágenes. Para crear imágenes de contenedores a través de Dockerfiles, Podman usa la API golang de Buildah y se puede instalar de forma independiente de Buildah.

Una diferencia importante entre Podman y Buildah es su concepto de contenedor. Podman permite a los usuarios crear "contenedores tradicionales". Si bien los contenedores de Buildah en realidad solo se crean para permitir que el contenido se vuelva a agregar a la imagen del contenedor. Una forma fácil de pensar en ello es que el comando **buildah** run emula el comando **RUN** en un **Dockerfile** mientras que el comando **podman run** emula el comando **docker run** en su funcionalidad. Debido a esto y a sus diferencias de almacenamiento subyacentes, no puede ver los contenedores de Podman desde Buildah o viceversa.



