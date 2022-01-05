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

Para matar ese proceso (y por consiguiente detener el contenedor):

```sh
docker inspect -f '{{.State.Pid}}' alwaysup | xargs sudo kill -9 # Podría omitirse el sudo en caso de que el usuario sea root
```

Para detener el contenedor, otra forma es la siguiente:
```sh
docker stop alwaysup
```

Si quisiese borrarlo aún estando en ejecución:

```sh
docker rm -f alwaysup
```

Entonces ahora `docker ps -a` nos informará que el container se ha detenido

# Exponiendo contenedores

Por ejemplo si usamos el container de nginx:

```sh
docker run -d --name proxy nginx
docker ps -a # Está escuchando en el puerto 80 del CONTENEDOR
```

Por defecto nginx escucha en el puerto 80 del contenedor, esto no implica que pueda acceder a él desde el exterior.
Para mapear ese puerto a algún puerto de nuestra PC podemos hacer lo siguiente:

```sh
docker stop proxy
docker run -d --name proxy -p 8080:80 nginx
docker ps -a # Está escuchando en el puerto 80 del contenedor pero en el puerto 8080 de nuestra PC
```

Y luego podemos entrar a http://localhost:8080

# Logs

Para poder ver los logs de un contenedor:

```sh
docker logs <container_name>
```

Para poder ver los logs de un contenedor en tiempo real:

```sh
docker logs -f <container_name>
```

Para ver los logs en tiempo real limitando a las últimas 10:

```sh
docker logs -f --tail 10 <container_name>
```

# Trabajando con Datos

Los procesos de los contenedores por defecto no pueden acceder al file system del host. Pero a veces podemos necesitar que accedan a ellos por distintos motivos, por ejemplo para persistirlos.

Por ejemplo si trabajamos con mongodb

```sh
docker pull -d --name db mongo:4.4.12-rc0-focal # #lijo esta por ser más liviano
```
Vamos a insertar algunos datos:

```sh	
docker exec -it db bash
mongo # Ejecuto el cliente de DB
show dbs
db.users.insert({"name":"swaxtech"}) # Insertamos un dato para crear la db
db.users.find() # Nos devuelve el usuario insertado
# CTRL + C
docker stop db
docker rm -f db
docker run -d --name db mongo:4.4.12-rc0-focal
docker exec -it db bash
mongo # Ejecuto el cliente de DB
db.users.find() ## No devuelve nada
```

Queremos tener un directorio en el host que refleje aquello que pasa en el contenedor.

## Bind Mounts

Para poder hacer un bind mount sobre un directorio:

```sh	
mkdir mongodata
pwd ## Nos dice el directorio actual
docker run -d --name db -v /home/alee/mongodata:/data/db mongo:4.4.12-rc0-focal
```

Si no ponemos el path completo el bind mount será en `/var/lib/docker/volumes/mongodata`

Cada cambio hecho en el directorio montado del contenedor se verá reflejado en el directorio especificado del host

El problema que tiene esto es que le estamos dando acceso a docker a una parte de nuestro file system y no sabemos qué puede tener ese contenedor en su interior. Por lo tanto, Docker nos ofrece otras formas más seguras de manejar esto.

## Volúmenes

Es una evolución más segura de los bind mounts. La parte del disco que le entregamos como volumen es manejada íntegramente por docker y no tenemos acceso a menos que seamos usuarios privilegiados.

Para listas los volúmenes que Docker se encuentra manejando:

```sh
docker volume ls
```

Para crear un volumen:

```sh
docker volume create dbdata # Sino se crea con el siguiente comando
```

Creamos el contenedor con este volumen:

```sh
docker run -d --name db --mount src=dbdata,dst=/data/db mongo:4.4.12-rc0-focal
```

## Tmpfs mount

Todos los archivos creados en tmpfs mount no tendrán ningun tipo de persistencia en el disco.

# Manejo de Archivos

Creamos un archivo para insertar en el contenedor:

```sh	
touch test.txt
docker run -d --name copytest ubuntu tail -f /dev/null
docker exec -it copytest bash
mkdir testing
exit

# Para copiar el archivo
docker cp test.txt copytest:/testing/test.txt
docker exec -it copytest ls
```

Para recuperar el archivo (también funciona para directorios):

```sh
docker cp copytest:testing/ testingcopied
```

Nota: No hace falta que el contenedor esté corriendo para copiar archivos.


# Imágenes

Vienen a solucionar los problemas de construcción y distribución del software
Son plantillas/moldes en los que docker crea contenedores.
Contiene todo lo necesario para que un contenedor pueda ejecutarse.

```sh
docker image ls ## Listar imágenes
```

Vemos que tienen distintos tamaños, y pueden tener distintos tags, por defecto latest.

Cada imagen se compone de distintas capas, dotandolo de mayor eficiencia.

Las imágenes se descargan de hub.docker.com

Para descargar una imagen de dockerhub (o algún repositorio distinto):

```sh	
docker pull ubuntu:20.04 # O la última
```

Al listar las imágenes vamos a ver dos casos de ubuntu:

```sh
docker image ls
```

```
ubuntu                20.04              ba6acccedd29   2 months ago   72.8MB
ubuntu                latest             ba6acccedd29   2 months ago   72.8MB
```
Ambos IMAGE_ID coinciden, ambas imágenes son el mismo archivo.
Como 20.04 es la última, latest apunta a esa versión.

# Creando nuestra propia Imagen

Para crear nuestras imágenes vamos a necesitar un archivo Dockerfile.

```sh
vim Dockerfile
```

```Dockerfile
FROM ubuntu:latest

RUN touch /usr/src/hello-world.txt
```

En este archivo especificamos que como base, usaremos la última versión disponible de ubuntu. Y en tiempo de *build*, se ejecutará el comando que escribimos en RUN.

Para construir la imagen:

```sh
docker build -t ubuntu:hello-world .
```

Hay 2 pasos en este build. Setear la imagen base, y correr el comando.
Cada paso tiene asociado un ID. Es decir, cada paso se encuentra asociada a una CAPA.
Cada instrucción genera una nueva layer.
Y cada layer es una *diferencia* con la capa anterior. Con lo que se va formando un arbol de dependencias.

Para crear un nuevo contenedor a partir de esta imagen:

```sh
docker run -it ubuntu:hello-world ls /usr/src/
```

Para publicar esta imagen en dockerhub

```sh
docker login
docker tag ubuntu:hello-world swaxtech/ubuntu:hello-world
docker push swaxtech/ubuntu:hello-world
```

En este momento tenemos dos tags:

-> ubuntu:hello-world
-> swaxtech/ubuntu:hello-world

Ambas apuntan a la misma imagen, cuya última capa corresponde al step de `RUN touch ...`

Al pushear vemos que únicamente subió una de las capas, es decir la última. El resto de las capas ya existen en dockerhub.

# El sistema de Capas

Como vimos antes, docker genera un sistema de capas como un árbol de dependencias, en donde cada capa guarda la diferencia con la capa anterior. Para ver discriminada cada una de las capas:

```sh
docker history ubuntu:hello-world
```

Otra manera para ver cómo está construida una imagen de docker, se puede usar [dive](https://github.com/wagoodman/dive)

Corriendo el siguiente comando:

```
dive ubuntu:hello-world
```

Si al Dockerfile creado recién agregamos la siguiente línea:


```Dockerfile
FROM ubuntu:latest

RUN touch /usr/src/hello-world.txt

RUN rm /usr/src/hello-world.txt
```

Al buildear esta imagen, hay una nueva capa basada en la anterior que lo que hace es borrar el archivo creado, es decir, hace un uso ineficiente del sistema de capas. Esto es frecuente al momento de instalar ciertas dependencias, que es necesario borrar algunos archivos luego.

Si se van a crear archivos que luego deben ser borrados, es conveniente realizar todo en la misma operación, para que solo ocupe una única capa.

Tener en cuenta que las capas son inmutables, para asegurar que es completamente reutilizable.

Cada vez que un contenedor se ejecuta se crea una nueva capa mutable, cuyas diferencias (es decir, archivos creados, etc) pueden rescatarse con las herramientas vistas antes.

Para persistir esta última capa se puede usar el comando `docker commit`







