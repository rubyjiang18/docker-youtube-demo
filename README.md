# Selected Notes from [Docker Tutoriaks for Beginners](https://www.youtube.com/watch?v=3c-iBn73dDE)
Check the default notes too.

## 1. What is Docker, a Container, an image?
Docker is a way to package applications with all necessary dependencies and configuration, which is portable artifact (Docker images) and easily shared and moved around. 

Docker Container is composed of layers of images (mostly Linux Base Image, because of small in size), with application image on top. Contrainer images live in container repository, which could be public (DockerHub) and private. To run a public container locally, for example:
    ```
    docker run postgres:9.6
    ```
To list all docker containers:
    ```
    docker ps
    ```

You will see that postgres:9.6 is an image. Docker images are artifacts that can be moved around. Docker container actually starts the application, where the container environment is created. In short, container is a running environment for image.

## 2. Docker vs Virtual Machine
Application <-> OS Kernel <-> Hardware.
Docker virtualizes the application layer, it uses the kernel of the host because it does not have its own kernel. Whereas VM has the application layer and its own kernel, so it virtualize the complete operating system.

Consequences:
* Size: size of docker image is much smaller as it only has to implement one layer (MB vs GB).
* Speed: Docker containers runs much faster.
* Compatibility: VM of any OS can run on any OS host, but you cannot do that with Docker. Solution is Docker tookbox.

## 3. Basic Commands
Container has a port that is binded to it, which makes it possible to talk to the applications. Container also has its own abstraction of an operating system including the file system and the environment, different from the host machine. 
    ```
    docker poll redis (docker poll [image])
    docker images
    docker run redis or docker run -d redis
    docker ps
    docker stop [CONTAINER ID]
    docker start [CONTAINER ID]
    ```

To remove a docker image `docker rmi [IMAGE ID or NAME]`. If the image is currently being used by a container, say 'my-app', you have to first delete the container:
    ```
    docker ps -a | grep my-app
    docker stop [CONTAINER ID]
    docker rm [CONTAINER ID]
    docker rmi [IMAGE ID]
    ```

To show all docker containers that are running and stopped `docker ps -a`

To use the containers you started, you need to create binding between a port that your laptop (host machine) has and the container. After that you can connect to the running container using the **port of the host**. Your laptop has only certain ports available, and there will be conflict when use the same port on you host machine for different containers. This is specified in the run command, where 6000 is local port:
    ```
    docker ps
    docker stop [CONTAINER ID]
    docker run -p6000:6379 -d redis
    docker ps
    ```

To see what logs a container is producing:
    ```
    docker ps
    docker logs [CONTAINER ID] or docker logs [CONTAINER NAME]
    ```

To specify the name of a container:
    ```
    docker run -p6000:6379 --name redis-older -d redis:4.0
    ```

To debug the container:
    ```
    docker exec -it [CONTAINER ID or NAME] /bin/bash
    ls
    pwd
    ...
    ```

## 4. Practical developing demo with a JS application that uses mongoDB database
Check README.md for more instructions.
4.1 `docker pull mongo` (pull the official mongodb image on docker hub to local machine)
4.2 `docker pull mongo-express`
4.3 Docker Networks `docker network ls`, `docker network create mongo-network`
4.4 Start mongo 
    ```docker run -d \
            -p 27017:27017 \
            -e MONGO_INITDB_ROOT_USERNAME=admin \
            -e MONGO_INITDB_ROOT_PASSWORD=password \
            --name mongodb \
            --net mongo-network mongo
    ```
4.5 Start mongo-express
    ```
    docker run -d \
    -p 8081:8081 \
    -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \
    -e ME_CONFIG_MONGODB_ADMINPASSWORD=password \
    --net mongo-network \
    --name mongo-express \
    -e ME_CONFIG_MONGODB_SERVER=mongodb mongo-express   
    ```
4.6 Open mongo-express from browser, and create `user-account` _db_ and `users` _collection_ in mongo-express

## 5. Docker Compose - Running multiple services
Docker compose lets you to write all the run commands for each container in a structured way. See the `docker-compose.yaml`. Note that Docker Compose takes care of creating a common Network.

To start the containers using docker-compose.yaml:
    ```
    docker-compose -f docker-compose.yaml up
    ```

To stop all containers, which also removes the network:
    ```
    docker-compose -f docker-compose.yaml down
    ```

## 6. Dockerfile - dockerize Node.js app
This step is to dockerize your own application and deploy it somewhere. Dockerfile is a blueprint for building images. All the commands in Dockerfile will apply to the container environment, and not affect the local host. Except the COPY command, which execute on the host machine (COPY [source] [destination]).

| Image Environment Blueprint | DOCKERFILE           |
|-----------------------------|----------------------|
| install node                | __FROM__ node        |
| set MONGO_DB_USERNAME=admin        | __ENV__ MONGO_DB_USERNAME=admin \ |
| set MONGO_DB_PWD=password          |         MONGO_DB_PWD=password     |
| create /home/app folder            | __RUN__  mkdir -p /home/app       |
| copy all files in app to /home/app | __COPY__ ./app /home/app          |
| set work dir                           | __WORKDIR__ /home/app             |
| start the app with "node server.js"    | __CMD__ ["node", "server.js"] |

In order to build an image using Dockerfile, we need to provide two parameters:
* an image name: -t my-app:1.0
* location of the Dockerfile: as we are in the same folder as Dockerfile, just `.` to specify the current directory
-> ```docker build -t my-app:1.0 .```
Then `dicker run my-app:1.0`, and see the logs `docker log [CONTAINER ID]`. To get more insight, let's get the command line terminal of the container `docker exec -it [CONTAINER ID] /bin/sh` (try /bin/bash too), then you can `ls`,`exit`, etc.

If Dockerfile is modified, to rebuild follow these steps:
* stop the container
* remove the container
* remove the image
* docker build the image
* docker run the container

## 7. Private Docker Repo and Deploy
Check the youtube video.

## 8. Volumes Demo - keep the data
Check the youtube video, and I may update this part later if needed. The Docker Volume Locations in Mac OS is `/var/lib/docker/volumes`. 
