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
  -- docker1
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
* If you browse: http://localhost:8085 you should see the legend "Hello World"

### Creating a PHP container to run our application with a Dockerfile

#### Introduction
 
* In the previous example we mounted a path in the host to provide the source files for our application to the container.
* It is better to create an image of all the application files needed to run our container. 
  * The reason is that we can use the same image and release it in multiple environments changing only the configuration.
* To create a Docker Image we need a Dockerfile

#### Create a Dockerfile

* A Dockerfile is like a recipe to create an image.
* Each line in a Dockerfile is an instruction that is executed to build an image.
* So first, let's copy our last example in a new directory docker2.
* Then create an empty Dockerfile inside docker2 directory. 

```
 /tmp
 |
 -- containersDemo
  |
  -- docker2
  |
   -- Dockerfile
   -- src
    |
    -- index.php  
```

* Edit the Dockerfile and paste this content:

```
FROM php:apache
COPY src/ /var/www/html/
EXPOSE 80
```

* Note that there are 3 instructions in this Dockerfile
  * The first line (FROM instruction) will create an image starting from the image: php:apache of the official repository
  * The second line (COPY instruction) will copy the src/ path in the host to the /var/www/html/ path in the image
  * The third line (EXPOSE) functions as a type of documentation between the person who builds the image and the person who runs the container, about which ports are intended to be published.

#### Build the image

* After you have the Dockerfile, is time to build the image with the "docker build" command.

```
docker build -t php_img .
```

* The "-t" flag is the tag or name that we want to assign to this image
* Note the dot at the end of the command, this means that we want to build the image from the current directory.
* Once the image is built you can verify it by running docker images command

```
docker images
```
```
REPOSITORY   TAG        IMAGE ID       CREATED         SIZE
php_img      latest     3bd85bbab5db   7 days ago      417MB
```

#### Run a Container with the created image

* Finally, we execute this docker command:
```
docker run -p 8085:80 php_image
```

* The php_image parameter indicates that we want to use the image that we just built.
* In this case we have all the files inside the image that's why we don't need to mount a volume like in the previous example.  
* The "-p" flag is used to publish a container port to a host port.
  * In this case is publishing the port 80 of the container to the port 8085 in the host.
* If you browse: http://localhost:8085 you should see the legend "Hello World"

# **Docker Compose**

## **What is Docker Compose?**

* Docker Compose is a tool to define and run multi-container docker applications.

## **How it Works?**

Using docker compose is a two-step process
* Create a YAML file to configure your application services.
* Run the command docker-compose up to start the services defined in the YAML file created before.  

## **Installation**

* Docker Compose can be installed on Linux, Windows and macOS, for instructions refer to: https://docs.docker.com/compose/install/

## **Examples**

### PHP Application with Mysql

#### Create the directory structure

* In this example we can create the following directory structure, and a file index.php inside the src directory.
* Note that the directory structure is arbitrary, built just for explanation purposes.
```
 /tmp
 |
 -- containersDemo
  |
  -- dockerCompose1
  |
  -- Dockerfile
  -- docker-compose.yml
  -- src
   |
   -- index.php
  -- storage
   |
   -- mysql
```

#### Build the PHP Application Image

**index.php**

* First we are going to do edit the index.php to update the code to query a table of mysql database and show the results:

```
<?php
echo "<html>";
$conn = new mysqli("mysql8-service", "db_user", "db_password", "my_db");
if ($conn->connect_error) {
  die("Connection failed: " . $conn->connect_error);
}
$sql = "SELECT name FROM user";
$result = $conn->query($sql);
if ($result->num_rows > 0) {
  while($row = $result->fetch_assoc()) {
    echo $row['name']."<br>";
  }
} else {
  echo "0 results";
}
$conn->close();
echo "</html>";
?>
```

**Dockerfile**

* Then we are going to edit the Dockerfile and add the following contents:
  * As you notice the difference with the previous Dockerfile is this line: RUN docker-php-ext-install pdo pdo_mysql mysqli
    * Docker provides docker-php-ext-install command to install PHP extensions and we want to have PHP installed for mysql.

```
FROM php:apache
RUN apt-get update
RUN docker-php-ext-install pdo pdo_mysql mysqli
COPY src/ /var/www/html/
EXPOSE 80
```

**Build**

* Now that we have the application that we want to build, and the Dockerfile we can build the image:

```
sudo docker build -t usersapp:latest .
```

* This command will build the image and tag it as usersapp:latest 

#### Creating the docker-compose.yml file to run the application

* Now that we have the image of our application, lets write the docker-compose.yml file:

**docker-compose.yml**

```
version: '2'
services:
  website:
    container_name: usersapp
    image: usersapp:latest
    ports:
      - 8088:80
  mysql:
     image: mysql:8.0
     container_name: mysql
     command: --default-authentication-plugin=mysql_native_password
     volumes:
       - /tmp/containersDemo/dockerCompose1/storage/mysql:/var/lib/mysql
     environment:
       - MYSQL_ROOT_PASSWORD=test
       - MYSQL_DATABASE=my_db
       - MYSQL_USER=db_user
       - MYSQL_PASSWORD=db_password
```

* The format of the file is yaml and each block has an indentation of two spaces.
* The very first line is about the version of docker-compose itself. In this case is version 2.
* The ***services section*** will have a list of the applications that you want to run.
  * I decided to use website for the php application and mysql to the mysql server. You can use any name you want.
  * This name is going to identify the containers and to communicate between them.
* Then on each service the meaning of some attributes:
  * ***container_name:*** Is the name that we want to name the container, if not specified the name of the directory will be prepended.
  * ***image:*** Is the image that you want to use to launch the container.
  * ***ports:*** List of ports that you want to associate from the host to the container.
    * In this case we want to associate the port 8080 of the host with the port 80 in the container.  
  * ***command:*** The arguments that we want to send to the startup command in the container.
    * In this case we want to use the mysql_native_password authenticator plugin for mysql.  
  * ***volumes:*** The list of host paths that we want to mount on the container.
    * We want to mount a host path in this case to store the mysql database. If we rebuild the container, the database data will survive.  
  * ***environment:*** List of key/values that we want to set as environment variables in the container.
    * In the mysql image these are used to set up the root password and the first database.


#### Starting the application

* Now that we have written the docker-compose file we can start it with the docker-compose up command

```
docker-compose up -d
```
```
Creating usersapp ... done
Creating mysql    ... done
```
  
* After we start the application, we can see that the containers are running with the docker ps command

```
docker ps
```
```
CONTAINER ID   IMAGE             COMMAND                  CREATED          STATUS          PORTS                  NAMES
d2ae52d25131   mysql:8.0         "docker-entrypoint.s…"   36 seconds ago   Up 35 seconds   3306/tcp, 33060/tcp    mysql
8d49687a60b1   usersapp:latest   "docker-php-entrypoi…"   36 seconds ago   Up 34 seconds   0.0.0.0:8088->80/tcp   usersapp
```

#### Configuring the database

* When we started mysql we used this environment variables to create a database and a user with permissions to access it
```  
  - MYSQL_ROOT_PASSWORD=test
  - MYSQL_DATABASE=my_db
  - MYSQL_USER=db_user
  - MYSQL_PASSWORD=db_password
```

* What we need to do now is to create a table into that database and add some users to it. 
  
***Access the running mysql container***
```  
docker exec -it mysql bash
```
```
root@14590145fdef:/# 
```

***Access mysql in the container***
* Use root user and password provided in the environment variable MYSQL_ROOT_PASSWORD
```  
root@14590145fdef:/# mysql -uroot -ptest
```
```
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.26 MySQL Community Server - GPL

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>  
```

***Create table user in my_db database and add two users in it***
* The database my_db is created when we start the container because the docker image check for the environment variables:
  * MYSQL_USER=db_user
  * MYSQL_PASSWORD=db_password
* In the mysql prompt copy and paste the following
```  
USE my_db;
CREATE TABLE user (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL
);
INSERT into user (name) values ("Michael");
INSERT into user (name) values ("Tony");
```

***Exit mysql container***
* Type exit two times to exit mysql and then the container
```
mysql> exit
Bye
root@14590145fdef:/# exit
```

#### Check that the application is running

* Finally, we can access the application at http://localhost:8085 and check that is displaying the list of users:
```
Michael
Tony
```












  




  

