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
docker pull -d --name db mongo:4.4.12-rc0-focal # elijo esta por ser más liviano
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

> Nota: No hace falta que el contenedor esté corriendo para copiar archivos.


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

-> ubuntu:hello-world </br>
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


# Desarrollando Aplicaciones con Docker

Ejemplo de aplicación en node:

```Dockerfile
FROM node:12

COPY [".", "/usr/src/"]

WORKDIR /usr/src/

RUN npm install

EXPOSE 3000

CMD ["node", "index.js"]

```

El comando de `CMD` es el que se ejecuta al iniciar el contenedor.

Para construir la imagen:

```sh
docker build -t mynodeapp .
```

Y para correr el contenedor:

```sh
docker run -rm -p 3000:3000 mynodeapp
```

El flag `-rm` es para que el contenedor se destruya al terminar. </br>
El puerto 3000 de mi pc se bindea al puerto 3000 del contenedor.

## Aprovechando el caché de capas

Para agilizar el trabajo, docker cachea las capas que ya se han construido. Al buildear nuevamente la imagen, no se construye nuevamente la capa, sino que se usa la que ya existe en el caché.
Pero al cambiar la versión de node, toda la caché se invalida.
Lo mismo si cambiamos alguna línea de código.

Ésta es una versión más optimizada del Dockerfile:

```Dockerfile
FROM node:14

COPY ["package.json", "package-lock.json", "/usr/src/"]

WORKDIR /usr/src/

RUN npm install

COPY [".", "/usr/src/"]

EXPOSE 3000

CMD ["node", "index.js"]

```

Es interesante saber que el segundo copy no sobreescribe los .json que se copiaron antes.

Si quisiésemos NO buildear ante cada cambio de código, en lugar de copiar los archivos, compartimos ese cacho de file system con el contenedor, y le avisamos a node de la siguiente manera:

Entre las dependencias de node, agregamos nodemon, y el command queda así:

```Dockerfile
FROM node:14

COPY ["package.json", "package-lock.json", "/usr/src/"]

WORKDIR /usr/src/

RUN npm install

COPY [".", "/usr/src/"]

EXPOSE 3000

CMD ["npx", "nodemon", "index.js"]
```

Y ejecutamos el contenedor de la siguiente manera:

```sh
docker run --rm -p 3000:3000 -v /home/user/myapp:/usr/src/ mynodeapp
```

En este momento, el volumen sobreescribe en su capa mutable, la capa en la que se instalaron las dependencias, para solucionar eso, hacemos lo siguiente:

```sh
docker run --rm -p 3000:3000 -v /home/user/myapp/index.js:/usr/src/index.js mynodeapp # Únicamente montamos el index!
```

Si bien es poco práctico, esto lo solucionamos más adelante.

# Docker Networking

Necesitamos que los programas que corren en distintos contenedores puedan conocerse, para ello se pueden crear redes entre los contenedores.

Docker ya viene con algunas networks creadas:

```sh
docker network ls
```

- bridge -> Está únicamente por retrocompatibilidad
- host -> Permite que los contenedores se conecten a través de la misma red/interfaz del host.
- none -> Para que el contenedor no tenga acceso a ninguna red, ni internet.

Podemos crear nuestras propias networks

```sh
docker network create --attachable mynetwork
```

Y podremos ver todos sus detalles mediante el comando:

```sh
docker network inspect mynetwork
```

Para conectar nuestro container con el programa de node, y el contenedor con el mongo:

```sh
docker network connect mynetwork mynodeapp
docker network connect mynetwork db
```

Para luego poder iniciar la app de node de la siguiente manera:

```sh
docker run --rm -p 3000:3000 -e MONGODB_URI=mongodb://db:27017/mydb mynodeapp
```

Ahora el contenedor de node conoce a mongo como "db" y podemos utilizarlo en la uri de mongodb.

# Docker Compose

Para poder realizar todas las operaciones anteriores en conjunto, en lugar de escribir un comando con todas las opciones, puede crearse un archivo de configuración de docker-compose.

   
```yaml
version: "3.8" ## Versión del compose file. Obligatorio aclararla.

services: ## Son los servicios que componen la aplicación.
   app: ## Implica no sólamente un contenedor, sino que cada servicio puede tener varios contenedores.
      image: mynodeapp 
      ports:
         - "3000:3000"
      environment:
         - MONGODB_URI=mongodb://db:27017/mydb
      depends_on:
         - db
      ports:
         - "3000:3000"

   db:
      image: mongo
```

Además por defecto va a crear una network que interconecte ambos contenedores. Y se pueden comunicar utilizando el nombre del servicio.


## Subcomandos del Docker Compose

```sh
docker-compose up # Construye e inicia todos los contenedores.
docker-compose down # Destruye todos los contenedores.
docker-compose logs # Muestra los logs de todos los contenedores.
docker-compose logs app # Muestra los logs del contenedor app.
docker-compose logs -f app # Muestra los logs del contenedor app haciendo follow.
docker-compose logs -f app db # Muestra los logs del contenedor app y db haciendo follow.
docker-compose exec app bash # Ejecuta un shell en el contenedor app. No hizo falta poner -it.
```

## Desarrollando con Docker Compose

En lugar de que utilice una imagen de docker, podemos decirle que contruya la imagen con el Dockerfile creado antes:

      
```yaml
services:
   app:
      build: .
      ports:
         - "3000:3000"
      environment:
         - MONGODB_URI=mongodb://db:27017/mydb
      depends_on:
         - db
      ports:
         - "3000:3000"
   db:
      image: mongo
```

Luego para que tome los cambios del código:

```yaml
services:
   app:
      build: .
      ports:
         - "3000:3000"
      environment:
         - MONGODB_URI=mongodb://db:27017/mydb
      depends_on:
         - db
      volumes:
         - .:/usr/src/ # Agregamos el volumen con la carpeta actual
         - /usr/src/node_modules # Le decimos que NO pise esta carpeta.
      command: npx nodemon index.js # Overrideamos el comando que se ejecuta en el contenedor.
      ports:
         - "3000:3000"
   db:
      image: mongo
```

## Compose Override

Para poder usar compose colaborativamente, si no queremos que ciertos cambios queden versionados, podemos crear un archivo aparte que sobreescriben cierta parte del archivo de compose file:

```sh
touch docker-compose.override.yml
```

```yaml
version: "3.8"

services:
   app:
      environment:
         - MONGODB_URI=mongodb://db:27017/mydb
      volumes:
         - .:/usr/src/ # Agregamos el volumen con la carpeta actual
         - /usr/src/node_modules # Le decimos que NO pise esta carpeta.
      command: npx nodemon index.js # Overrideamos el comando que se ejecuta en el contenedor.
```
Y ponemos en este archivo todos los parámetros que son particulares del desarrollo local.

> Nota: No es aconsejable usar ports en el override, a menos que se especifique únicamente allí.

# Escalando Contenedores

Puedo querer tener 2 instancias de mi contenedor:

```sh
docker-compose up -d --scale app=2
```

Por cómo venimos trabajando esto fallaría, pero podemos especificar en el mapeo de puertos un rango que pueda tomar:

```yaml
services:
   app:
      build: .
      ports:
         - "3000:3000"
      environment:
         - MONGODB_URI=mongodb://db:27017/mydb
      depends_on:
         - db
      volumes:
         - .:/usr/src/ # Agregamos el volumen con la carpeta actual
         - /usr/src/node_modules # Le decimos que NO pise esta carpeta.
      command: npx nodemon index.js # Overrideamos el comando que se ejecuta en el contenedor.
      ports:
         - "3000-3001:3000"
   db:
      image: mongo
```
Esto creará 3 contenedores, 2 apps escuchando en el 3000 y 3001 respectivamente y un mongo. Esto luego requerirá de un load balancer configurado.

# Administrando el Ambiente de Docker


## Borrar lo que ya no usamos

```sh
docker container prune # Borra todos los contenedores frenados
docker rm -f ${docker ps -aq} # Borra todos los contenedores, aunque estén activos.
docker network prune # Borra todas las redes creadas.
docker volume prune # Borra todos los volúmenes creados.
docker system prune # Borra todos los contenedores, redes y volúmenes creados.
```


## Limitar recursos

```sh
docker stats ## Nos dice cuánto está consumiendo cada contenedor
docker run -d --name mynodeapp --memory 512m -p 3000:3000 mynodeapp
```

## Matando contenedores

Por defecto docker enviará **SIGTERM** al proceso para stoppear los contenedores.
Si el contenedor no responde en un cierto tiempo enviará un **SIGKILL**

Si al hacer `docker ps` vemos que el código de salida es mayor a 128, es porque hubo problemas al manejar la señal de SIGTERM.
Por ejemplo si tuvimos un 137, (137 - 128 = 9) => 9 => El contenedor se frenó por un SIGKILL

Si quiero que directamente envíe un SIGKILL puedo utilizar:

```sh
docker kill <name/id_contenedor>
```
Pero esto evita un graceful shutdown.

Hay que tener en cuenta una cosa:

### Exec Form vs Shell Form

Supongamos que como proceso principal tengo un script que atrapa la señal de SIGTERM.

script.sh
```sh
#!/usr/bin/env bash
trap 'exit 0' SIGTERM
while true; do:; done
```

El proceso con PID 1 será

```sh
/bin/sh -c /script.sh
```

Pero habrá otro proceso:

```sh
bash /script.sh
```

Y el problema es que /bin/sh no reenvia la señal a sus procesos hijos.

Esto sucede únicamente si ese script está puesto de la siguiente manera en el Dockerfile:

```Dockerfile
FROM ubuntu
COPY script.sh /script.sh
CMD /script.sh 
```

Esta forma de ejecutar comandos se lo llama **shell form**, y lo corre como hijo del shell.
Conviene entonces utilizar la forma **exec form**

```Dockerfile
FROM ubuntu
COPY script.sh /script.sh
CMD ["/script.sh"]
```

### EntryPoint vs CMD

```Dockerfile
FROM ubuntu:trusty
CMD ["/bin/ping", "-c", "3", "localhost"]
```

Esto al ejecutarlo realiza 3 pings hacia si mismo.

Podría querer que ese ping pueda utilizarlo para pingear otros hosts, utilizando un ENTRYPOINT.

Un ENTRYPOINT es un comando que se va a ejecutar siempre, y utilizará el CMD como parámetro del ENTRYPOINT.

   
```Dockerfile
FROM ubuntu:trusty
ENTRYPOINT ["/bin/ping", "-c", "3"]
CMD ["localhost"]
```
Y puedo ejecutar lo siguiente:

```sh
docker build -t ping .
docker run --name pinger ping google.com
```

Utilizará el entrypoing pero utilizando google.com en lugar de localhost.

# Multi-stage build

Puedo utilizar multi-stage builds para distintos contextos, por ejemplo:

production.Dockerfile:
```Dockerfile
FROM node:12 as builder
COPY ["package.json", "package-lock.json", "/usr/src"] 
WORKDIR /usr/src
RUN npm install --only=production
COPY [".", "/usr/src"]
RUN npm install --only=development ## Únicamente para reutilizar las capas!
RUN npm run test

## Productive image
FROM node:12
COPY ["package.json", "package-lock.json", "/usr/src"]
WORKDIR /usr/src
RUN npm install --only=production ## Hasta acá utiliza el caché
COPY --from=builder ["/usr/src/index.js", "/usr/src"]
EXPOSE 3000
CMD ["node", "index.js"]
```

```sh
docker build -t prodapp -f production.Dockerfile .
```


















