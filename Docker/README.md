# Docker cheat sheet

* [Docker Commands](#docker-commands)
* [Docker Volumes](#docker-volumes)
* [Docker Network](#docker-network)
* [Dockerfile](#dockerfile)
* [Docker-Compose](#dokcer-compose)
* [Other Resources](#other-resources)

## Docker Commands
* Stop, Start or Restart container by ID or name
```
$ docker stop/start/restart [CONTAINER]
```

* Inspect Container information
```
$ docker inspect [CONTAINER]
```

* run commands inside the container and access the docker container
```
$ docker exec <option> <container ID or Name> <Command> <Args>
$ docker exec -it 09ca6feb6efc bash
```

* See logs of container to debug or track failure
```
$ docker logs [CONTAINER]
```

> use -f flag to show live logs of running daemon container

* Remove image (rmi) or Container (rm)
```
$ docker rm/rmi [CONTAINER/IMAGE]
```

* Remove all unused images
```
$ docker rmi $(docker images -q -f "dangling=true")
```

* Delete all stopped containers
```
$ docker rm $(docker ps --filter status=exited -q)
```

* Show exposed ports of a container
```
$ docker port c7337
```

* docker commit command saves changes in a container as a new image
```
$ docker commit -m "[build notes]" -a "creator info" [container name or ID] [username/imagename]:[version tag]
```

* Upload a docker image with the image name mentioned in the command on the dockerhub or local Hub
```
$ docker push [username/imagename]:[version tag]
```

* Example: 
```
// pull nginx image and apply some chanes on it
$ docker commit -m="This a test image" nginx_cont milad72t/nginx_test
//login into docker hub
$ docker push milad72t/nginx_test
```

* Save created image into file
```
$ docker save -o <ImageFile.tar> <ImageName>
// or
$ docker save <ImageName> | gzip > <ImageFile.tar> 
```

* Load image from file
```
$ docker load -i <ImageFile.tar>
```

* Example:
```
$  docker save -o alpine-elinks.tar alpine-elinks-installed
$  docker load –i alpine-elinks.tar
$  docker run -i -t –name elinks-container alpine-elinks-installed
```

* use --restart flag to configure the restart policy
```
$ docker run -dit --restart always redis // no, on-failure, unless-stopped
```

> A restart policy only takes effect after a container starts successfully
No: Do not automatically restart the container. (the default)
on-failure: Restart the container if it exits due to an error, which manifests as a non-zero exit code.
Always: Always restart the container if it stops. If it is manually stopped, it is restarted only when Docker daemon restarts or the container itself is manually restarted
unless-stopped: Similar to always, except that when the container is stopped (manually or otherwise), it is not restarted even after Docker daemon restarts.


* Use –e,--env or --env-file to set environment variables in command or file 
```
$ docker run –d –e DB_NAME=mydb --env-file env.list mysql 
```

* Docker allow restrict resources (CPU, Mem, GPU, …) on each container
```
$ docker run –d --memory=2g --cpus=".5" Jenkins // or 2048b, 1024k, 256m
```

> --memory-reservation: Allows you to specify a soft limit smaller than --memory   which is activated when Docker detects contention or low memory on the host machine.


* Use  ``docker stats`` command to Display a live stream of container(s) resource usage statistics

* Attach local standard input, output, and error streams to a running container
```
$ docker attach [OPTIONS] CONTAINER
```


## Docker Volumes
When we start a container, Docker takes the read-only images and adds a read-write layer on top. Changes are applied into the top-most read-write layer.A Docker volume is a directory that lives on the host file system

* Create and manage volumes
```
$ docker volume create my-vol
$ docker volume ls
$ docker volume inspect my-vol
$ docker volume rm my-vol
```

> Volume Path:/var/lib/docker/volumes/my-vol/_data   
* Start a container with a volume
```
$ docker run -d --name nginx  --mount source=myvol2,destination=/app nginx:latest
//or
$ docker run -d --name nginx -v myvol2:/app nginx:latest
```

> ro and readonly to make volume readonly on container side => -v nginx-vol:/usr/share/nginx/html:ro & --mount source=nginx-vol,destination=/usr/share/nginx/html,readonly

* To remove all unused volumes and free up space:
```
$ docker volume prune
```

### Backup, restore, or migrate data volumes
Volumes are useful for backups, restores, and migrations. Use the --volumes-from flag to create a new container that mounts that volume.

* Backup a container
```
$ docker run -v /dbdata --name dbstore ubuntu /bin/bash
$ docker run --rm --volumes-from dbstore -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata
```

> in windows and powershell use ${PWD} to get current directory

* Restore container from backup
```
$ docker run -v /dbdata --name dbstore2 ubuntu /bin/bash
$ docker run --rm --volumes-from dbstore2 -v $(pwd):/backup ubuntu bash -c "cd /dbdata && tar xvf /backup/backup.tar --strip 1"
```

## Docker Network
when install Docker Engine creates three networks automatically: bridge, host, none

in bridge Each container is assigned its own private IP address which is not reachable from the outside by default  
On user-defined containers can not only communicate by IP address, but can also resolve a container name to an IP address
* Creating a bridge network
```
$ docker network create --driver bridge my-network
```
* add the –network parameter to the run command
```
$ docker run -it --name alpine1 --network my-network alpine bash
```
* Inspecting a network
```
$ docker network inspect bridge
```

When a container is connected to the host network, it gets the same network configuration as in the host OS 

In none network mode, containers are not attached to any network and do not have any access to the external network or other containers


## Dockerfile
* Build an image from Dockerfile in current directory
```
$ docker build --tag myimage .
```

### Basic Commands
* FROM – Defines the base image to use and start the build process.
* RUN – It takes the command and its arguments to run it from the image.
* CMD – Similar function as a RUN command, but it gets executed only after the container is instantiated (specifies arguments that will be fed to the ENTRYPOINT).
* ENTRYPOINT – specifies a command that will always be executed when the container starts.
* ADD – It copies the files from source to destination (inside the container).
* ENV – Sets environment variables.

> CMD vs ENTRYPOINT: the main point to remember is CMD is used to provide default executable and arguments, which can be overridden, while ENTRYPOINT is used to specify specific executable and arguments, to constraint the usage of image

### MongoDB dockerfile example
```
# Set the base image to Ubuntu
FROM ubuntu

# Update the repository sources list and install gnupg2
RUN apt-get update && apt-get install -y gnupg2

# Add the package verification key
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10

# Add MongoDB to the repository sources list
RUN echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' > tee /etc/apt/sources.list.d/mongodb.list

# Update the repository sources list
RUN apt-get update

# Install MongoDB package (.deb)
RUN apt-get install -y mongodb

# Create the default data directory
RUN mkdir -p /data/db

# Expose the default port
EXPOSE 27017

# Default port to execute the entrypoint (MongoDB)
CMD ["--port 27017"]

# Set default container command
ENTRYPOINT usr/bin/mongodb
```

### Simple node dockerfile example 

* Dockefile
```sh
FROM node:latest
RUN mkdir node
WORKDIR /node
COPY package.json socketIo.js ./
ENV REDIS_URL='localhost'
RUN npm install
RUN npm install pm2 -g
EXPOSE 8695
CMD ["pm2-runtime","socketIo.js"]
```

* package.json 
```js
{
  "name": "socketIo_server",
  "version": "1.0.0",
  "description": "socketIo server for notification",
  "author": "milad72t",
  "license": "MIT",
  "main": "socketIo.js",
  "keywords": [
    "nodejs",
    "socket",
    "redis",
    "express"
  ],
  "dependencies": {
    "express": "^4.16.4",
    "redis": "^2.8.0",
    "socket.io": "^2.2.0",
    "ioredis": "^4.9.0",
    "pm2": "^3.4.1"
  }
}
```

* socketIo.js
```js
var app = require('express')();
var http = require('http').Server(app);
var io = require('socket.io')(http);
var ioRedis = require('ioredis');
var client = new ioRedis(6379,process.env.REDIS_URL);
client.subscribe('message', function(err, count) {});
client.on('message', function(channel, message) {
    console.log('Message Recieved: ' + message);
    message = JSON.parse(message);
    io.emit(message.channel, message.data);
});
http.listen(8695, function(){
    console.log('Listening on Port 8695 ');
});
```

## Docker Compose

#### Basic config example
docker-compose.yml
```YAML
version: '3'

services:
  web:
    build: .
    context: ./Path
    dockerfile: Dockerfile
    environment:
      - RACK_ENV=development
    env_file: .env
    ports:
      - "5000:5000"
    volumes:
      - .:/code
    depends_on:
      - redis
  redis:
    image: redis
    restart: always
    volumes:
      - /var/lib/redis
      - ./_data:/var/lib/redis
```

#### Commands
Starts existing containers for a service.
>docker-compose start

Stops running containers without removing them.
>docker-compose stop

Pauses running containers of a service.
>docker-compose pause

Unpauses paused containers of a service.
>docker-compose unpause

Lists containers.
>docker-compose ps

Builds, (re)creates, starts, and attaches to containers for a service.
>docker-compose up

Stops containers and removes containers, networks, volumes, and images created by up.
>docker-compose down

#### Network
By default Compose sets up a single network for your app. For Example:
``` YAML
version: "3"
services:
  web:
    build: .
    ports:
      - "8000:8000"
  db:
    image: postgres
    ports:
      - "8001:5432"
```
When you run docker-compose up, the following happens:

1. A network called ``myapp_default`` is created.
2. A container is created using ``web``’s configuration. It joins the network ``myapp_default`` under the name ``web``.
3. A container is created using ``db``’s configuration. It joins the network ``myapp_default`` under the name ``db``.

It is important to note the distinction between ``HOST_PORT`` and ``CONTAINER_PORT``. In the above example, for db, the ``HOST_PORT`` is ``8001`` and the container port is ``5432``.

Within the ``web`` container, your connection string to ``db`` would look like ``postgres://db:5432``, and from the host machine, the connection string would look like ``postgres://{DOCKER_IP}:8001``.

Instead of just using the default app network, you can specify your own networks with the top-level ``networks`` key.
```YAML
version: "3"
services:

  proxy:
    build: ./proxy
    networks:
      - frontend
  app:
    build: ./app
    networks:
      - frontend
      - backend
  db:
    image: postgres
    networks:
      - backend

networks:
  frontend:
    # Use a custom driver
    driver: custom-driver-1
  backend:
    # Use a custom driver which takes special options
    driver: custom-driver-2
    driver_opts:
      foo: "1"
      bar: "2"
```
 The proxy service is isolated from the db service, because they do not share a network in common - only app can talk to both.


## Other Resources

* [Official Doc](https://docs.docker.com/get-started/)
* [docker-cheat-sheet](https://github.com/wsargent/docker-cheat-sheet)
* [Play with Docker](https://labs.play-with-docker.com/#)
* [Docker Curriculum](https://docker-curriculum.com/)
