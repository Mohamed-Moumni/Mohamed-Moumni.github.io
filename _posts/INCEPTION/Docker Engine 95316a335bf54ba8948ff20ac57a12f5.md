# Docker Engine

## **What is Docker?**

Docker is an open-source project for building, shipping, and running programs. It is a command-line program, a background process, and a set of remote services that take a logistical approach to solving common software problems and simplifying your experience of installing, running, publishing, and removing software. It accomplishes this by using an operating system technology called containers.

## **What are containers?**

Before talking about Docker, I need to say a few words about containerization technology.

**Containers** are a way to package an application and all of its dependencies into a single image. This image runs in an isolated environment that does not affect the host operating system. Containers allow you to separate the application from the infrastructure: developers do not need to think about what environment their application will run in, and whether there will be the necessary settings and dependencies. They simply create an application and package all the dependencies and settings into a single image. This image can then be run on other systems without worrying that the application will not run.

**Docker** is a platform for developing, delivering, and running containerized applications. Docker allows you to create containers, automate their launch and deployment, and manage the life cycle. It allows you to run multiple containers on a single host machine.

Containerization is similar to virtualization, but they are not the same. Virtualization works like a separate computer, with its own virtual hardware and operating system. In this case, you can run another OS inside one OS. In the case of containerization, the virtual environment runs directly from the kernel of the main operating system and does not virtualize the hardware. This means that the container can only run on the same OS as the main one. At the same time, since containers do not virtualize hardware, they consume much fewer resources.

## **What problems does Docker solve?**

Most modern applications have similar setups. They all use a combination of different technologies to assemble a complete application functionality. An example would be an app that uses a combination of following
services:

- Node.js for Webserver
- ReactJs for frontend
- MongoDB as a database
- Messaging system - Redis

These technologies each have a version the application depends on, also the application isn't an isolated thing that just floats around. It needs to run in an environment, since the environment can differ in OS, version, hardware, etc, it's obvious that the application and its technologies with their respective versions should work the same in different environments. Without docker, this means that each environment that the application runs on (local dev environment, a test or production server) needs to be configured with the correct versions of these services so that 
the application can run properly.

**So the following problems arise**

- Compatibility of each service with the underlying OS
- Compatibility of each service with the libraries and dependencies of OS (One service requires version X of the OS library. Another service -
version Y of the same library)
- Every time version of any service updates, you might need to recheck compatibilities with the underlying OS infrastructure
- For a new developer to setup the environment with the right OS and Service versions

**Docker Solution**

- Each service has and can manage its required OS dependencies for itself, bundled and isolated in its own container
- Change the components without affecting the other services
- Change underlying OS without affecting any of the services

As a result, docker should avoid the typical "works on my machine" cases. In the development process, for example, developers and testers will have the identical environment where the application runs since this environment is packaged in docker containers, which just like a file, can be transferred around as an artifact.

![docker_image.png](Docker%20Engine%2095316a335bf54ba8948ff20ac57a12f5/docker_image.png)

## **What is an Image?**

A container image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings. Container images become containers at runtime.

A Docker image is a read-only template that contains a set of instructions for creating a container that can run on the Docker platform. It provides a convenient way to package up applications and preconfigured server environments, which you can use for your private use or share publicly with other Docker users. Docker images are also the starting point for anyone using Docker for the first time.

To create an image you should write your instructions for the image in a dockerfile. So What is a dockerfile?

## **What is a Dockerfile?**

A dockerfile is a text file that contains commands you would normally execute manually to build a docker image. Docker can build images automatically by reading the instructions we have in our dockerfile.
Each of the files that make up a docker image is known as a layer. these layers form a series of images, built on top of each other in stages. Each layer is dependent on the layer immediately below it. The order of your layers is key to the efficiency of the lifecycle management of your docker images.

## **How to create a Docker image?**

You can create a Docker image by creating a dockerfile for the needed image and adding the commands you need to assemble The image. The following commands are the most used for creating a dockerfile:

For example in This project the Dockerfile of NGINX image looks like this:

```docker
# base image
FROM    debian:buster

# install nginx and openssl keys and certificates
RUN     apt update && apt -y install nginx && apt install -y openssl && openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt -subj="/CN=mmoumni/O=moumni.1337.ma/C=MA/L=KHOURIBGA"

# copy the default configuration into the container
COPY    ./conf/default /etc/nginx/sites-enabled/default

# running the nginx in daemon mode
CMD     ["nginx", "-g", "daemon off;"]
```

## **Docker Compose**

The ability to run one container could be great if you have a self-contained image that has everything you need for your single use case, where things get interesting is when you are looking to build multiple applications between different container images. For example, if I had a website front end but required a backend database I could put everything in one container but better and more efficient would be to have its container for the database

This is where Docker compose comes in which is a tool that allows you to run more complex apps over multiple containers. A good example is the one given in the project we have 3 services each one inside a container.

![Inception project structure](Docker%20Engine%2095316a335bf54ba8948ff20ac57a12f5/proj_struc.png)

Inception project structure

## **Docker-Compose.yml (YAML)**

The `docker-compose.yml` file is used to configure the services, networks, and volumes for the application. It allows developers to define the services, their dependencies, and the 
configuration options in a single file, and then start, stop and manage the entire application using a single command.

You can see the `Docker-compose.yml` that I used in the project:

```docker
version: '3.5'

services:
    mariadb:
        build: ./requirements/mariadb
        container_name: mariadb
        image: mariadb:inception
        env_file:
            - .env
        restart: always
        volumes:
            - data_db:/var/lib/mysql/
        ports:
            - '3306:3306'
        networks:
            - nat
    nginx:
        build: ./requirements/nginx
        container_name: nginx
        image: nginx:inception
        env_file:
            - .env
        ports:
            - '443:443'
        restart: on-failure
        depends_on:
            - wordpress
        volumes:
            - data:/var/www/html/
        networks:
            - nat
    wordpress:
        build: ./requirements/wordpress
        container_name: wordpress
        image: wordpress:inception
        env_file:
            - .env
        restart: always
        depends_on:
            - mariadb
        ports:
            - '9000:9000'
        volumes:
            - data:/var/www/html/
        networks:
            - nat
    adminer:
        build: ./requirements/bonus/adminer
        container_name: adminer
        image: adminer:inception
        restart: always
        ports:
            - '8081:8081'
        networks:
            - nat
    redis:
        build: ./requirements/bonus/Redis
        container_name: redis
        image: redis:inception
        restart: on-failure
        ports:
            - '6379:6379'
        networks:
            - nat
    cadvisor:
        build: ./requirements/bonus/cadvisor
        container_name: cadvisor
        image: cadvisor:inception
        restart: on-failure
        ports:
            - '8080:8080'
        volumes:
            - "/:/rootfs:ro"
            - "/var/run:/var/run:rw"
            - "/sys:/sys:ro"
            - "/var/lib/docker/:/var/lib/docker:ro"
        networks:
            - nat
    static_stite:
        build: ./requirements/bonus/static_site
        container_name: static_site
        image: static_site:inception
        restart: on-failure
        ports:
            - '80:80'
        networks:
            - nat
    ftp:
        build: ./requirements/bonus/FTP
        container_name: ftp
        image: ftp:inception
        restart: on-failure
        ports:
            - '21:21'
            - '30000-30009:30000-30009'
            - '20:20'
        env_file:
            - .env
        volumes:
            - data:/home/moumniVolum/ftp
        depends_on:
            - wordpress
        networks:
            - nat
volumes:
    data:
        driver: local
        driver_opts:
            device : /Users/mmoumni/Desktop/INCEPTION/srcs/data
            type : none
            o: bind
    data_db:
        driver: local
        driver_opts:
            device: /Users/mmoumni/Desktop/INCEPTION/srcs/data_db
            type: none
            o: bind
networks:
    nat:
        driver: bridge
```

**Docker compose vs Dockerfile script**

- Docker compose is like a collection of docker run commands, and containers and surrounding environments (networks, volumes, etc.) can be created at once.
- A dockerfile script is a tool for creating non-container images, and it is impossible to create environments around containers such as networks and volumes.

## **Data storage in Docker**

In Docker, a volume is a way to store data outside of a container's filesystem. When a volume is created, it exists independently of any container and can be shared among multiple containers. Volumes can be used to persist data even if the container is deleted or to share data between containers. They can also be used to store configuration files or logs.

**Volume Mounting**

So, the reason why we do volume mounting is pretty clear. This is to ensure that even if the Docker gets destroyed or has to be closed in any emergency condition the data is not lost and we can create a new docker container the very next instant and have all the data retained as if the container never went down in the first place.

So, without wasting any time let’s see how to do we create the persistent volume for docker to save its data and where this persistent volume is located on your host machine.

![1*lZg9p7gD80vjZAlvEga6EA.webp](Docker%20Engine%2095316a335bf54ba8948ff20ac57a12f5/1lZg9p7gD80vjZAlvEga6EA.webp)

```docker
docker volume create data_volume
```

For macOS, you run the following command:

```docker
screen ~/Library/Containers/com.docker.docker/Data/vms/0/tty
```

You **should** see a screen, where you can enter the command :

```docker
cd /var/lib/docker
```

Now once that you have created the persistent storage space for dockers our next step will be to link it with the docker container.

```docker
docker run -v data_volume:/var/lib/mysql mysql
```

This command helps us mount the volume inside the docker container i.e. we are linking the data being stored in /var/lib/mysql from the Mysql docker to data_volume which is present at /var/lib/docker/volumes in your docker space.

The trick is here that even if we just type in the command just above us without typing the volume create command, docker would itself create the data_volume automatically for us and link the volume but it is preferred to type in both the commands rather than typing in only the docker run command.

**Bind Mounting**

Moving on to bind mounting let me explain why it is used in the first place. So volume mounting is used to prevent loss of data i.e. when the docker container is shut down due to whatever reason the data should not be lost and must be retained.

![1*0yK3LMnkCpNAniGuBLyGxg.webp](Docker%20Engine%2095316a335bf54ba8948ff20ac57a12f5/10yK3LMnkCpNAniGuBLyGxg.webp)

The main difference between bind mounting and volume mounting is that we use a bind mount to create the persistent volume on the host machine and not in the docker host directory whereas volume mounting does it on the docker host hence you need to provide the entire directory name while mounting.

```docker
docker run -v /data/mysql:/var/lib/mysql mysql
```

This is the command to link the new docker container with the persistent volume that we already have in our local system when we set up the previous step of configuring our volume mounting.

**Docker Network**

In real-life scenarios one docker alone is not very useful and can’t really perform a lot of tasks. So most of the time we end up setting up more than one docker which communicates with other running containers to carry out a task.

To communicate with other dockers we need to set up networking within these for these containers to send and receive data. So, let’s see how many types of docker networks are there and how do they work.

- **Bridge Network**

Docker containers are running on a bridged network when all of them are on the
same network of 172.12.0.0/16. In this way, they can easily communicate among themselves.

![Bridge Network](Docker%20Engine%2095316a335bf54ba8948ff20ac57a12f5/sdf.webp)

Bridge Network

- **Host Network**

In this scenario, the docker container’s port is mapped to the port of the

host machine. As we perform port mapping so so no other container can 
use that port as it is already being used by other containers.

![Host Network](Docker%20Engine%2095316a335bf54ba8948ff20ac57a12f5/sdff.webp)

Host Network

- **Isolated**

This is the case when there is a container running on a totally different network from other docker containers and can’t communicate with any of them.

![Isolated Network](Docker%20Engine%2095316a335bf54ba8948ff20ac57a12f5/sfsdf.webp)

Isolated Network

**Creating User-defined Network**

## **Docker Hub**

Docker Hub is a service provided by Docker that allows users to share and distribute their Docker images. It is a centralized repository for storing and managing Docker images, and it allows users to easily discover and download images that they can use to create and run their own containers.

It's just like Github, you can pull official images of multiple services, but those images are prohibited to use in this project, we were forced to create our own images for each service.

Docker Hub is widely used by developers, DevOps professionals, and companies to distribute and share their Docker images. It is also a great place to find existing images for different applications and services, making it easy for developers and operations teams to get started with containerization.