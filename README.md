# Docker

## Estructura de Docker

Docker es una plataforma que permite construir, ejecutar y compartir aplicaciones mediante contenedores.

Docker en su core posee el Docker Daemon, es un servicio del sistema operativo que maneja todas las entidades con las que trabaja docker, que se comunica mediante una API Rest con el cliente de docker (Docker CLI).

Sobre Docker se trabaja sobre principalmente 4 entidades:

- Contenedores: Allí correrán las aplicaciones.
- Imágenes: Son los artefactos que usa docker para empaquetar contenedores (y el código).
- Volúmenes: Es la forma en la que Docker nos permite acceder con seguridad al File System de la máquina anfitriona.
- Network: Permite a los contenedores comunicarse entre sí y con el mundo exterior.




# Comandos básicos

## Versión de docker
```sh
docker --version
```

## Información sobre Docker
```sh
docker info
```


## Hello World

```sh
docker run hello-world
```

Al no encontrar la imagen localmente, procederá a descargarla desde Docker Hub.
Posteriormente ejecuta el contenedor y muestra una salida como la siguiente:

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
2db29710123e: Pull complete 
Digest: sha256:2498fce14358aa50ead0cc6c19990fc6ff866ce72aeb5546e1d59caac3d0d60f
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

En la próxima ejecución, no descargará la imagen de DockerHub.

# Contenedores

Un contenedor es una agrupación lógica de procesos que corren de manera nativa sobre la máquina anfitriona pero están aisladas del resto del sistema. Es mucho más liviano que una máquina virtual.

# Comandos Básicos II

Para ver los contenedores activos en el sistema:
```sh
docker ps
```

Para ver todos los contenedores que hay en el sistema:
```sh
docker ps -a
```

Al ejecutar este último comando, si ejecutamos dos veces `docker run hello-world` veremos que existen 2 contenedores distintos que ejecutaron el hello world. Es decir, `docker run` crea un nuevo contenedor en cada ejecución.

Para tener más información acerca de un contenedor:
```sh
docker inspect <container_id>
```

Además del container_id, cada container posee un name o alias que se puede usar para identificarlo.

## Seteando un nombre al contenedor
```sh
docker run -it --name <nombre> hello-world
```

Si se utiliza otra vez el comando anterior, daría error, porque no permite que haya dos contenedores en el sistema con el mismo nombre.

## Renombrando un contenedor
```sh
docker rename <old> <new>
```
Hasta ahora los contenedores estaban frenados, pero existían y tenían un lugar en el sistema, pero podríamos borrarlos con el siguiente comando:

## Eliminar un contenedor
```sh
docker rm <container_id/name>
```

## Eliminar todos los contenedores parados
```sh
docker container prune
```

## Para eliminar los últimos containers utilizados:

Por ejemplo si quiero eliminar los últimos 2 containers que se crearon con el hello world puedo hacer:

```sh
docker ps -a | awk -v DELETE=2 '{if(NR > 1 && NR <= (DELETE + 1)) print $1}' | xargs docker rm
```

En donde `DELETE` es la variable en donde especificaré el número de containers que quiero eliminar.

# Modo interactivo

## Instalamos ubuntu

```sh
docker run ubuntu
```

Por defecto, estará con un estado `EXITED`, ya que el command que ejecutó fue `/bin/bash`.
Al no tener ningun input, y no tener nada para hacer, la shell se cerrará y el contenedor se detendrá.

Para conectarnos al ubuntu y poder interactuar con el, habilitamos el modo interactivo:

```sh
docker run -it ubuntu
```

O bien ejecutamos un comando de la siguiente manera:

```sh
docker run -it ubuntu <comando>
```

# Ciclo de Vida de un Contenedor

Cada vez que un contenedor se ejecuta, ejecuta un proceso del sistema operativo.
Es proceso o _main process_ determina si el contenedor está corriendo o no.
Por defecto un contenedor corre siempre y cuando su proceso principal esté corriendo.

Si no queremos que se apague podemos hacer lo siguiente

```sh
docker run --name alwaysup -d ubuntu tail -f /dev/null
```

Y al hacer

```
docker ps
```

esta será la salida:

```
CONTAINER ID   IMAGE     COMMAND               CREATED         STATUS        PORTS     NAMES
4a052b3214ba   ubuntu    "tail -f /dev/null"   4 seconds ago   Up 1 second             alwaysup
```

Ahora puedo ejecutar comandos en el contenedor mediante `docker exec`:

```sh
docker exec -it alwaysup <comando>
```

```sh
docker exec -it alwaysup bash
```

Una vez dentro, puedo ejecutar el siguiente comando para ver los procesos en ejecución:

```sh
ps -aux
```

Y tendremos una salida como la siguiente:

```sh
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   2552   516 ?        Ss   03:10   0:00 tail -f /dev/null
root           7  0.0  0.0   4116  3416 pts/0    Ss   03:12   0:00 bash
root          22  0.0  0.0   5904  2728 pts/0    R+   03:16   0:00 ps -aux
```
En donde vemos el main process, pero tambien vemos los otros procesos que creamos recién.

Para matar ese proceso:

```sh
docker inspect -f '{{.State.Pid}}' alwaysup | xargs sudo kill -9 # Podría omitirse el sudo en caso de que el usuario sea root
```

Entonces ahora `docker ps -a` nos informará que el container se ha detenido

# Exponiendo contenedores






















