+++
title = "docker"
author = ["Sherlockes"]
date = 2023-08-30T00:00:00+02:00
lastmod = 2024-06-05T18:36:28+02:00
tags = ["docker"]
categories = ["apps"]
draft = false
weight = 5
thumbnail = "images/docker.png"
toc = true
+++

Docker es un proyecto de código abierto que automatiza el despliegue de aplicaciones dentro de contenedores de software, proporcionando una capa adicional de abstracción y automatización de virtualización de aplicaciones en múltiples sistemas operativos.

<!--more-->


## Instalación {#instalación}

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
chmod u+x get-docker.sh
sudo ./get-docker.sh
sudo usermod -aG docker $(whoami)
```


### Ejecutar sin sudo {#ejecutar-sin-sudo}

```bash
sudo groupadd docker
sudo usermod -aG docker <usuario>
su - <usuario>
```


## Comandos básicos {#comandos-básicos}

-   Formatear la salida (json) -&gt; `comando --format '{{ json . }}' | jq`
-   Parar un contenedor - docker stop xxxxxxxxx
-   Parar todos contenedores - docker stop \`docker ps -q\`
-   Listar todos contenedores - `docker container ls`
-   Lanzar un docker-compose.yml -&gt; `docker compose up`
-   Lanzar un docker-compose.yml (Segundo plano) -&gt; `docker compose up -d`


### Sistema {#sistema}

-   Obtener info del sistema -&gt; `docker system info --format 'El número de CPU es: {{ .NCPU }}'`
-   Obtener info del sistema (json) -&gt; `docker system info --format '{{ json .}}' | jq`
-   Estadísticas (Instantáneas) -&gt; `docker container stats --no-stream`
-   Estadísticas (Tiempo real) -&gt; `docker container stats`
-   Espacio usado -&gt; `docker system df`


### Imágenes {#imágenes}

-   Descargar -&gt; ~docker image pull &lt;imagen&gt;
-   Listar (Todas) -&gt; `docker image ls`
-   Listar (filtro) -&gt; `docker image list filtro`
-   Borrar (Todas) -&gt; `docker rmi -f $(docker images -q)`
-   Buscar -&gt; `docker search nginx`
-   Inspeccionar -&gt; `docker image inspect hello-world --format '{{ json .Config }}' | jq`


### Contenedores {#contenedores}

-   Listar (Todos que están corriendo) -&gt; `docker container ls`
-   Listar (Todos) -&gt; `docker container ls --all`
-   Borrar -&gt; `docker container rm id`
-   Borrar (Todos) -&gt; `docker rm -f $(docker ps -aq)`
-   Borrar (Todos forzando) -&gt; `docker container rm -f $(docker container ls -aq)`
-   Arranque en modo interactivo -&gt; `docker container run -it <imagen>`
-   Arranque desacoplado -&gt; `docker container run --detach <imagen>` (o tambien `-d`)
-   Parar un contenedor -&gt; `docker container stop`
-   Consultar log y esperar -&gt; `docker container logs --follow <contenedor>`
-   Consultar log -&gt; `docker container logs <contenedor>`
-   Consultar log (parámetros) -&gt; `-since` (Desde), `-tail` (Ultimos), `-until` (Hasta)
-   IP asignada -&gt; `docker container inspect <id> --format '{{ json .NetworkSettings.Networks.bridge.IPAddress }}'`


### Redes {#redes}

-   Listar -&gt; `docker network ls`
-   Inspeccionar -&gt; `docker network inspect <nombre>`
-   Contenedores conectados a red -&gt; `docker network inspect bridge --format='{{ json .Containers }}'`
-   Crear nueva red "bridge" -&gt; `docker network create --driver bridge <nombre>`
-   Eliminar una red -&gt; `docker network rm <nombre>`


### Otros {#otros}
