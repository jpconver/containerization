# **Docker**

## **What is docker?**

Docker is an open source containerization platform. It enables developers to package applications into containers that combine the application source code with the operating system, libraries and dependencies to run that code in any environment.

## **Why Docker?**

* A developer can create a package of an application, the operating system and all dependencies needed to run the application in any environment.
This provides the dream of "build once, run anywhere".
* It's very lightweight compared to a Virtual Machine because it uses the virtualization features of the operating system allowing to run several containers simultaneously in a single server.
* Some Docker Containers Characteristics:
  * Can be limited in the amount of resources they use like CPU and memory.
  * Can share a directory with the host.
  * Can expose or restrict ports.
  * Can interact with each-other.

## **How it Works?**

* Docker uses a client-server architecture
* The Docker client talks to the Docker daemon, which does the heavy lifting of building, running, and distributing your Docker containers.

## **Installation**

* Docker can be installed on Linux, Windows and macOS, for instructions refer to: https://docs.docker.com/engine/install/

## **Architecture**

* **Docker Daemon**
  * Listens for Docker API requests and manage Docker objects
* **Docker Client**
  * Is the way that dockers users have to interact with Docker
* **Docker Registries**
  * Is a place to store docker images. Docker Hub is a public registry that anyone can use and docker is configured to loop for images on this registry by default
* **Docker Objects**
  * **Images**
    * An image is a readonly template with instructions for creating a Docker container.
    * Often an image is based on another image with some additional customization.
      * For example, you can build an image based on ubuntu but install apache web server and your application, as well as the configuration details needed to make your application run.
    * You can use your own images or use those created by others and published in the registry.
    * To build your own image you can create a Dockerfile that is like a recipe to build an image step by step.
    * Each instruction in the Dockerfile creates a layer in the image, if you change the Dockerfile only the layers that have changes are rebuilt.
      * This is what makes docker images lightweight and fast compared to other virtualization technologies.
  * **Containers**
    * A container is the running instance of an image.
    * You can create, start, stop, access, move or delete a container using the Docker Client or the API
    * A container is defined by its image, and the configuration options you provide to it when you create or start it.
    * When a container is removed, any changes to its state that are not stored in persistent storage disappear.

## **Examples**

### Interactive Container with Ubuntu image

* Create a docker container using an image of ubuntu, attach to it interactively and run the command /bin/bash

`docker run -i -t ubuntu /bin/bash`

When you type this command the following happens:
* If you do not have the ubuntu image locally, docker pulls it from the configured docker registry.
* Docker creates a new container using the ubuntu image.
* Docker allocates a read-write filesystem to the container, as its final layer. This allows a running container to create or modify files and directories in its local filesystem.
* Docker creates a network interface to connect to the container to the default network, this includes assigning an IP address to the container.
* Docker starts the container and executes /bin/bash. Because the container is running interactively and attached to your terminal (due to the -i and -t flags), you can provide input using your keyboard while the output is logged to your terminal.
* When you type exit to terminate the /bin/bash command, the container stops but is not removed. You can start it again or remove it.


### Container that stays alive

#### Creating the Container

* We are going to create a container that stays alive some time.
* This emulates having a container that executes an application that keeps running like Apache or Mysql.

`docker run -d ubuntu /bin/bash -c "sleep 3600"`

* The -d flag is used to run container in background and print container ID.
* The sleep 3600 is a bash command that waits a number of seconds before terminating.
* At this point the container keeps running

#### Viewing the containers that are running  

* You can view the running containers with the command: "docker ps"

`docker ps`

```
  CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
  4130774bf35c   ubuntu    "/bin/bash -c 'sleep…"   5 minutes ago   Up 5 minutes             unruffled_poitras  
```

#### Accessing the container that is running

* As the container is running, you can access it using another docker command called "docker exec"

`docker exec -i -t 4130774bf35c bash`

* We use the -i and -t flags to attach to the container interactively and with attached to your terminal.
* To access the container we need to use the ID of the container, or the name in the docker exec command.  
* When you type "exit", the interactive session is terminated, but the container keeps running.

#### Stopping the container

* You can stop the container with the command "docker stop"

`docker stop 413`

* This command stops a container, note that we can use as the argument just some part of the ID

#### Deleting a container

* After we stop a container, it is removed from the list of running containers so if you execute "docker ps" you are not going to be able to find it there

`docker ps`

```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

* To view stopped containers you need to add a flag to the previous command to show all containers (including stopped ones) 

`docker ps -a `

```
CONTAINER ID   IMAGE                                 COMMAND                  CREATED       STATUS                      PORTS     NAMES
4130774bf35c   ubuntu                                "/bin/bash -c 'sleep…"   2 hours ago   Exited (137) 2 hours ago              unruffled_poitras
```

* So if we want to remove the container, we can execute the command "docker rm"

`docker rm -v 413`

* We are using the flag -v to remove anonymous volumes associated with the container


### Simple PHP Application using the official docker image

#### Creating the PHP Application

* In this example we can create the following directory structure, and a file index.php inside the src directory
```
 /tmp
 |
 -- containersDemo
  |
  -- src
   |
   -- index.php  
```

* Then in the index.php file, add this content:
```
<?php echo 'Hello, World!'; ?>
```

#### Creating a PHP container to run our application

* Finally, we execute this docker command:
```
docker run -p 8085:80 -v /tmp/containersDemo/docker1/src:/var/www/html php:apache
```

* The php:apache parameter indicates that we want to use this image from the official repository
  * There are many images to use, to browse them you can browse: https://hub.docker.com/
* The "-p" flag is used to publish a container port to a host port. 
  * In this case is publishing the port 80 of the container to the port 8085 in the host.
* The "-v" flag is used to mount a host path to a path in the container.
  * In this case is mounting the /tmp/containersDemo/docker1/src path from the host into the /var/www/html in the container.
    * This has the effect of adding the file: index.php to the /var/www/html path in the container which is the path where apache looks to find your application start page.
* If you browse: http://8085 you should see the legend "Hello World"

### Creating a PHP container to run our application with a Dockerfile

#### Introduction
 
* In the previous example we mounted a path in the host to provide the source files for our application to the container.
* It is better to create an image of all the application files needed to run our container. 
  * The reason is that we can use the same image and release it in multiple environments.
* To create a Docker Image we need a Dockerfile

#### Create a Dockerfile

* A Dockerfile is like a recipe to create an image.
* Each line in a Dockerfile is an instruction that is executed to build an image.
* In our case we want to create an image that start from the php:apache image and add our application in it
*   





















  




  


