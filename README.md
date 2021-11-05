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
 -- containerization
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
docker run -p 8085:80 -v /tmp/containerization/docker1/src:/var/www/html php:apache
```

* The php:apache parameter indicates that we want to use this image from the official repository
  * There are many images to use, to browse them you can browse: https://hub.docker.com/
* The "-p" flag is used to publish a container port to a host port. 
  * In this case is publishing the port 80 of the container to the port 8085 in the host.
* The "-v" flag is used to mount a host path to a path in the container.
  * In this case is mounting the /tmp/containerization/docker1/src path from the host into the /var/www/html in the container.
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
 -- containerization
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
 -- containerization
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
       - /tmp/containerization/dockerCompose1/storage/mysql:/var/lib/mysql
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

* When we started mysql we used this environment variables to create a database, and a user with permissions to access it
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

# **Kubernetes**

## **Introduction**

* With docker and docker-compose is very easy to divide an application into multiple containers and run it as a whole.
* Now, imagine that you have a very popular application deployed with docker-compose, and you want to:
  *  Add more containers and distribute traffic due to high load.
  *  Release a new version of your application without downtime.
  *  Detect if a container dies to start another container of the application.
* For the cases detailed before, and many others Kubernetes is a very good  solution as it provides tools for distributing and escalating applications.

## **What is Kubernetes?**
* Kubernetes (k8s) is an open-source system for automating deployment, scaling, and management of containerized applications.
* It groups containers that make up an application into logical units for easy management and discovery.
* It provides a way to group existing or new containers and make them run as a single logical entity thus make it easy to automate operations like deployment and scaling.
* It provides a way to orchestrate your containers.
* Container orchestration is about managing life cycles of containers.
   * In modern application development, a single application is divided into multiple units that are usually installed in containers.
   * Container orchestration helps to run these individual units as a single app.
   * Orchestration helps in provisioning and deployment of containers, resource allocation, load balancing, and many other things.

## **Kubernetes Architecture**
* A Typical Kubernetes installation consists of a master node and many worker nodes.
  * A node is a virtual or physical machine.

### **Master Node**

* In the master runs what is called the ***Control Plane*** which is the nerve center that houses the components that control the cluster.
* The ***Control Plane*** is composed by the following components:
  * **API Server:**
    * A REST API based server that allows the Administrator to perform several operations in the cluster like Pods creation, deployment etc.
  * **Scheduler:** Watches for newly created Pods with no assigned node, and selects a node for them to run on.
  * **Etcd:** Consistent and highly-available key value store used as Kubernetes' backing store for all cluster data.
  * **Controller Manager:** It is responsible to manage different kinds of controllers offered by Kubernetes, some types are:
     * **Node controller:**
       * Responsible for noticing and responding when nodes go down.
     * **Job controller:**
       * Watches for Job objects that represent one-off tasks, then creates Pods to run those tasks to completion.
     * **Endpoints controller:**
        * Populates the Endpoints object (that is, joins Services & Pods).
     * **Service Account & Token controllers:**
       * Create default accounts and API access tokens for new namespaces

### **Worker Node**

* In the worker nodes run the following components:
  * **Kubelet:**
    * The main service on a node, regularly taking in new or modified pod specifications (primarily through the kube-apiserver) and ensuring that pods and their containers are healthy and running in the desired state.
    * This component also reports to the master on the health of the host where it is running.
  * **Kube-Proxy:**
    * A proxy service that runs on each worker node to deal with individual host subnetting and expose services to the external world.
    * It performs request forwarding to the correct pods/containers across the various isolated networks in a cluster.
  * **Container Runtime:**
    * The container runtime is the software that is responsible for running containers.
    * Kubernetes supports several container runtimes: Docker, containerd, CRI-O, and any implementation of the Kubernetes CRI (Container Runtime Interface)

## **Kubernetes Concepts**

* Kubernetes uses different abstractions to represent the state of the system, such as nodes, services, pods, volumes, namespaces, and deployments.
  * **Node:**
    * A node may be a virtual or physical machine, depending on the cluster. Each node is managed by the control plane and contains the services necessary to run Pods.
  * **Pod:**
    * Refers to one or more containers that should be controlled as a single application. A pod encapsulates application containers, storage resources, a unique network ID and other configuration on how to run the containers.
  * **Service:**
    * A Service is an abstract way to expose an application running on a set of Pods as a network service.
    * Pods are volatile, that is Kubernetes does not guarantee a given physical pod will be kept alive (for instance, the replication controller might kill and start a new set of pods).
    * Instead, a service represents a logical set of pods and acts as a gateway, allowing (client) pods to send requests to the service without needing to keep track of which physical pods actually make up the service.
  * **Volume:**
    * A volume is a directory, possibly with some data in it, which is accessible to the containers in a pod.
    * Kubernetes supports many types of volumes. A Pod can use any number of volume types simultaneously with different access modes.
      * **Ephemeral volumes:** Have a lifetime of a pod, and when a pod ceases to exist, Kubernetes destroys them.
      * **Persistent volumes:** Exist beyond the lifetime of a pod and data is preserved across container restarts.
  * **Namespace:**
     * Kubernetes supports multiple virtual clusters backed by the same physical cluster. These virtual clusters are called namespaces.
     * Namespaces are intended to be used in environments where many users spread across multiple teams, or projects.
     * Resources inside a namespace must be unique and cannot access resources in a different namespace.
     * A namespace can be allocated a resource quota to avoid consuming more than its share of the physical cluster’s overall resources.
  * **Deployment**
    * A Kubernetes Deployment is used to tell Kubernetes how to create or modify instances of the pods that hold a containerized application.
    * Deployments can scale the number of replica pods, enable rollout of updated code in a controlled manner, or roll back to an earlier deployment version if necessary.
    * Since the Kubernetes deployment controller is always monitoring the health of pods and nodes, it can replace a failed pod or bypass down nodes, replacing those pods to ensure continuity of critical applications.
    * Deployment is a resource to deploy a stateless application, if using a PVC, all replicas will be using the same Volume and none of it will have its own state.
  * **Statefulset**
    * Manages the deployment and scaling of a set of Pods, and provides guarantees about the ordering and uniqueness of these Pods.
    * Like a Deployment, a StatefulSet manages Pods that are based on an identical container spec.
    * Unlike a Deployment, a StatefulSet maintains a sticky identity for each of their Pods. These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.
    * Statefulsets are used for Stateful applications, each replica of the pod will have its own state, and will be using its own Volume.

## **Installation**

* Installing a Kubernetes cluster involve many steps in different servers or virtual machines.
* Fortunately, Kubernetes provides a tool called Minikube that is a single cluster based environment for local development and testing.
* To install Minikube in your local workstation you can follow this documentation: https://minikube.sigs.k8s.io/docs/start/
* In this presentation we are going to install Minikube on Linux using the Ubuntu 18 distribution.

## **Installation on Linux - Ubuntu 18**

* **Download and install Minikube using the Docker Driver**
  * The docker driver allows running Kubernetes in a local workstation in a very lightweight fashion.
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube start --driver=docker
minikube config set driver docker
```

* **Download and install kubectl**
  * kubectl is the Kubernetes command-line tool, kubectl, allows you to run commands against Kubernetes clusters.
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

## Example 1: PHP Application with Mysql in Kubernetes

* Now It's time to migrate the application that we built using docker-compose to Kubernetes.

### Build the Docker images in Minikube

* Generally the nodes in a Kubernetes cluster have access to a company docker registry where the images are deployed after being built with a tool like Jenkins.
* This allows Kubernetes to download docker images from this registry.
* In our case we don't have a docker registry to upload the images, so we are going to build them inside Minikube.
* Fortunately when you install Minikube, you also get a pre-installed docker.
* The strategy that we are going to use is build the docker images using the docker inside Minikube.

#### Step 1: Configure your terminal to use the docker inside Minikube
```
eval $(minikube docker-env)
```
* After this command is executed, any docker command that you execute in the terminal will run against the docker inside minikube

#### Step 2: Configure the files for the image that we are going to build
 * For this example we will have the following directory structure
```
 /tmp
 |
 -- containerization
  |
  Dockerfile
  -- kubernetes
   |
   -- src
    |
    -- index.php
```

* Complete the Dockerfile with these contents:
  * This is the same exact content that we have for the docker-compose example
```
FROM php:7.3-apache
RUN apt-get update
RUN docker-php-ext-install pdo pdo_mysql mysqli
RUN a2enmod rewrite
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
RUN php composer-setup.php --install-dir=. --filename=composer
RUN mv composer /usr/local/bin/
COPY src/ /var/www/html/
EXPOSE 80
```

* Complete the index.php with these contents:
  * Note that the only difference with the example that we used for docker-compose is the host that in this case is: "mysql8-service"
    * We will see why shortly!
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

#### Step 3: Build the image
  * This command will build the image inside the docker in Minikube
```
docker build -t usersapp:latest .
```

### Create and deploy a deployment definition for the webserver

#### Step 1: Create a file: /tmp/containerization/kubernetes/webserver.yaml with the contents:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webserver
  labels:
    app: apache
spec:
  replicas: 3
  selector:
    matchLabels:
      app: apache
  template:
    metadata:
      labels:
        app: apache
    spec:
      containers:
      - name: userapp
        image: usersapp:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
```
* Notes:
  * ***replicas: 3*** To define that we want to use 3 replicas of the application.
  * ***containersPort: 80*** To define that the port that will be exposed is the port 80 of the container.
  * ***selector field*** defines how the Deployment finds which Pods to manage.
    * In this case, you select a label that is defined in the Pod template (app: apache)
  * ***imagePullPolicy: IfNotPresent*** Is used to avoid kubernetes to try to download the image of a repository as we build it manually into the Docker minikube.

#### Step 2: Make sure that you are in the minikube context
```
kubectl config use-context minikube
```
#### Step 3: Apply the definition to the cluster
```
kubectl create -f webserver.yaml
```

#### Step 4: Verify that deployment and pods are running
* We verify that we have 3/3 Pods Ready
```
kubectl get deployment
```
```
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
webserver   3/3     3            3           92m
```
* We verify that all pods are running
```
kubectl get pod
```
```
NAME                        READY   STATUS    RESTARTS   AGE
webserver-5c495dfdc-6z84g   1/1     Running   0          5s
webserver-5c495dfdc-jpl5g   1/1     Running   0          5s
webserver-5c495dfdc-w7stq   1/1     Running   0          5s
```

### Create a persistent volume claim for mysql

* We need to create a persistent volume for mysql to preserve data across container restarts.

#### Step 1: Create a file: /tmp/containerization/kubernetes/mysql-pvc.yaml with the contents:
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: mysql8
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```
#### Step 2: Apply the definition to the cluster
```
kubectl create -f mysql-pvc.yaml
```
#### Step 3: Verify that the Persistent Volume Claim & Persistent Volume is created
```
kubectl create -f mysql-pvc.yaml
```
```
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-pv-claim   Bound    pvc-ccb79793-a9f4-4494-8965-9239a35c8da9   5Gi        RWO            standard       3s
```
```
kubectl get pv
```
```
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   REASON   AGE
pvc-ccb79793-a9f4-4494-8965-9239a35c8da9   5Gi        RWO            Delete           Bound    default/mysql-pv-claim   standard                80s
```

* In this case what we need to check is the status that in this case is "Bound" meaning that it has been successfully allocated to the application.


### Create and deploy a deployment definition for mysql

#### Step 1: Create a file: /tmp/containerization/kubernetes/mysql.yaml with the contents:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql8
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql8
    spec:
      containers:
      - image: mysql:8.0
        name: mysql
        imagePullPolicy: IfNotPresent
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: test
        - name: MYSQL_DATABASE
          value: my_db
        - name: MYSQL_USER
          value: db_user
        - name: MYSQL_PASSWORD
          value: db_password
        args: ["--default-authentication-plugin=mysql_native_password"]
        ports:
        - containerPort: 3306
          name: mysql8
        volumeMounts:
          - name: mysql-persistent-storage
            mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
```
* Notes:
  * The "env" key is used to send environment variables to the container needed by the mysql image to set up the database.
  * The "args" key is used to send arguments to the command executed by the image when it starts.
  * The "volumeMounts" key is used to mount persistent volumes to a path in the container
  * The "volumes" key is used to link a persistent volume claim with a name, so it can be used in this definition.
  * In this case we are mounting the Persistent Volume defined by the Persistent Volume Claim: mysql-pv-claim in the path: /var/lib/mysql

#### Step 2: Apply the definition to the cluster
```
kubectl create -f mysql.yaml
```

### Create and deploy a service definition for the webserver

* A service is responsible to make possible accessing multiple pods in a way that the end-user does not know which application instance is being used. When a user tries to access an app, for instance, a web server here, it actually makes a request to a service which itself then check where it should forward the request.

#### Step 1: Create a file: /tmp/containerization/kubernetes/webserver-svc-nodeport.yaml with the contents:
```
apiVersion: v1
kind: Service
metadata:
  name: web-service
  labels:
    run: web-service
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: apache
```
* Notes:
  * We use "type: NodePort" because we want to access this service from outside the cluster using a port
  * In the selector we are defining that this service applies to the pods with the tag "app: apache"

#### Step 2: Apply the definition to the cluster
```
kubectl create -f webserver-svc-nodeport.yaml
```

#### Step 3: Verify that the service is created
```
kubectl get service
```
```
NAME          TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
web-service   LoadBalancer   10.96.149.205   <pending>     80:32757/TCP   3s
```
* Note that the External-IP is pending. This is because we are using Loadbalancer and since we are on minikube instead of some Cloud Provider, it will remain pending. In case if you run on Google Cloud or Azure you will get an IP for it.

### Create and deploy a service definition for the mysql database

* We are creating a service also for mysql

#### Step 1: Create a file: /tmp/containerization/kubernetes/mysql-svc.yaml with the contents:
```
apiVersion: v1
kind: Service
metadata:
  name: mysql8-service
  labels:
    app: mysql8
spec:
  type: ClusterIP 
  ports:
  - port: 3306
    protocol: TCP
  selector:
    app: mysql8
```
* Notes
  * We use "type: ClusterIP" because we want to connect the MySQL client with the DB inside the cluster.

#### Step 2: Apply the definition to the cluster
```
kubectl create -f mysql-svc.yaml
```

#### Step 3: Verify that the service is created
```
kubectl get service
```
```
NAME             TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
mysql8-service   NodePort       10.105.224.230   <none>        3306:31846/TCP   6s
web-service      LoadBalancer   10.96.149.205    <pending>     80:32757/TCP     16m
```

### Create a table into that database and add some users to it.

#### Step 1: List Running Pods
```
kubectl get pod
```
```
NAME                        READY   STATUS    RESTARTS   AGE
mysql-788fd8dc55-zdkkp      1/1     Running   0          114m
webserver-5c495dfdc-b4vpq   1/1     Running   0          7m43s
webserver-5c495dfdc-grbdg   1/1     Running   0          7m44s
webserver-5c495dfdc-qgl4r   1/1     Running   0          7m43s
```

#### Step 2: Access the console of mysql pod with kubectl
  * With the command exec we can access a running pod.
  * We use the flag -i to access an interactive shell.
  * We use the flag -t to use a terminal.
  * We use the command bash to execute this command in the container that will bring a shell.
``` 
kubectl exec -it mysql-788fd8dc55-zdkkp bash
```
```
root@mysql-788fd8dc55-zdkkp:/#
```

#### Step 3: Access mysql in the container
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

#### Step 4: Create table user in my_db database and add two users in it
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

#### Step 5: Exit mysql container
* Type exit two times to exit mysql and then the container
```
mysql> exit
Bye
root@14590145fdef:/# exit
```

### Test the application

* Finally, we have our web application running connected with a mysql database, lets test it.

#### Step 1: Find out Minikube external IP
* We use the minikube ip command
* In my case it returned 192.168.49.2 but in your case can be a different IP
```
minikube ip
```
```
192.168.49.2
```

#### Step 2: Find out the external port of the webserver service
```
kubectl get service
```
```
NAME             TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes       ClusterIP      10.96.0.1        <none>        443/TCP          17d
mysql8-service   NodePort       10.105.224.230   <none>        3306:31846/TCP   102m
web-service      LoadBalancer   10.96.149.205    <pending>     80:32757/TCP     118m
```
* In my case the web-service service is exposing the port 32757 to the internal pod port 80

#### Step 3: Access the application
* In your favourite web browser access  https://{MinikubeIP}:{externalWebServicePort}
** In this case http://192.168.49.2:32757

### Configure an Ingress to access your application using a URL
* A Kubernetes Ingress is an API object that provides routing rules to manage access to the services within a Kubernetes cluster.
* We will use it to add a rule to match a url name to our web-service

#### Step 1: Create a file: /tmp/containerization/kubernetes/ingress.yaml with the contents:
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: usersapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: www.usersapp.net
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-service-nodeport
                port:
                  number: 80
```
* Notes
  * With this ingress definition we are matching the host name: www.usersapp.net to the service web-service-nodeport at port 80

#### Step 2: Apply the definition to the cluster
```
kubectl create -f ingress.yaml
```

#### Step 3: Add the entry into your /etc/hosts file
  * This is needed so that your host knows that the domain name www.usersapp.net will be resolved to the IP of your minikube
```
sudo vi /etc/hosts
```
Add:
```
192.168.49.2    www.usersapp.net
```
* Change the IP for your Minikube IP

#### Step 4: Test your application
* Access your application at: http://www.usersapp.net

### Configure a ConfigMap & Secrets for the Mysql Deployment

* A **Secret** is an object that contains a small amount of sensitive data such as a password, a token, or a key. Such information might otherwise be put in a Pod specification or in a container image. Using a Secret means that you don't need to include confidential data in your application code.
* A **ConfigMap** is an API object used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.
A ConfigMap allows you to decouple environment-specific configuration from your container images, so that your applications are easily portable.

* We are going to secure mysql passwords into a Secret and move the user database information to a configmap.

#### Step 1: Create two secrets to store mysql passwords

* Note that we can create secrets using a declarative approach, here we are using the imperative approach for convenience purposes.

```
kubectl create secret generic db-root-pass \
  --from-literal=MYSQL_ROOT_PASSWORD=test

kubectl create secret generic db-user-pass \
  --from-literal=MYSQL_USER=db_user \
  --from-literal=MYSQL_PASSWORD=db_password
```

#### Step 2: Create a configmap to store the user database information

```
kubectl create configmap userdb-config \
  --from-literal=MYSQL_DATABASE=my_db
```

#### Step 3: Copy the file ./mysql.yaml to ./mysqlWithSecretAndConfigMap.yaml and update to use the secrets and configmap

* The "env" section will be modified as follows:

**before**
```
env:
        - name: MYSQL_ROOT_PASSWORD
          value: test
        - name: MYSQL_DATABASE
          value: my_db
        - name: MYSQL_USER
          value: db_user
        - name: MYSQL_PASSWORD
          value: db_password
```

**after**
```
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-root-pass
              key: MYSQL_ROOT_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            configMapKeyRef:
              name: userdb-config
              key: MYSQL_DATABASE
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: db-user-pass
              key: MYSQL_USER
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-user-pass
              key: MYSQL_PASSWORD

```

#### Step 4: Delete the mysql deployment and apply the new one
```
kubectl delete deployment mysql
kubectl create -f ./mysqlWithSecretAndConfigMap.yaml
```

### Just for Fun: Mount a configMap as a file to replace the home page of our application

#### Step 1: Create a copy of index.php and update it

* We are going to copy index.php to indexOverride.php
* In this new file we are going to add message: This is an updated version of the app using a configmap at the top of the page

```
<?php
echo "<html>";
echo "<h1>This is an updated version of the app using a configmap</h1>";
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

#### Step 2: Create a configmap using this file
```
kubectl create configmap webapp-home-override --from-file=./indexOverride.php
```

#### Step 3: Copy the file ./webserver.yaml to ./webserverWithConfigMap.yaml and update to use the configmap

* First: Add the volumes area to define the volume config-volume using the configmap webapp-home-override
* Second: Add a volumeMounts are to mount the indexOverride.php file to the path: /var/www/html/index.php in the container

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webserver
  labels:
    app: apache
spec:
  replicas: 3
  selector:
    matchLabels:
      app: apache
  template:
    metadata:
      labels:
        app: apache
    spec:
      containers:
      - name: userapp
        image: usersapp:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80
        volumeMounts:
          - name: config-volume
            mountPath: /var/www/html/index.php
            subPath: indexOverride.php
      volumes:
        - name: config-volume
          configMap:
            name: webapp-home-override
```

#### Step 4: Delete the webserver deployment and apply the new one
```
kubectl delete deployment webserver
kubectl create -f ./webserverWithConfigMap.yaml
```

### Use a StatefulSet instead of a deployment for the webserver

#### Step 1: Create a definition for a Headless service that will be in charge of controlling the network domain

* A StatefulSet needs a service to control the network domain
  * For example to be able to ping from one pod to the other using a domain name like:
  * webserver-1.web-service.default.svc.cluster.local
  * In this case: web-service is the name of the service that we are going to create

**Create a file with name: webserver-svc-headless.yaml**
```
apiVersion: v1
kind: Service
metadata:
  name: web-service
  labels:
    run: web-service
spec:
  clusterIP: None
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: apache
```

#### Step 2: Apply the Headless service definition
```
kubectl create -f ./webserver-svc-headless.yaml
```

#### Step 3: Copy the file ./webserverWithConfigMap.yaml to ./webserverWithConfigMapStatefulset.yaml and update it to change the definition to a StatefulSet

* In this case only two changes are needed:
  * replace Kind of the object to StatefulSet
  * add a serviceName key to point to the service created in step 2
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: webserver
  labels:
    app: apache
spec:
  serviceName: web-service
...
```

#### Step 4: Delete webapp deployment and Apply the statefulset definition
```
kubectl delete deployment webserver
kubectl create -f ./webserverWithConfigMapStatefulset.yaml
```

#### Results

* Now, if you execute a kubectl get pod, each webserver pod will have a unique id
  * Before with a Deployment the pods were interchangeable and have a hash that was changed if the pod is deleted for example.
```
kubectl get pod
NAME                    READY   STATUS    RESTARTS   AGE
mysql-db6674d57-hx224   1/1     Running   0          23h
webserver-0             1/1     Running   0          19m
webserver-1             1/1     Running   0          19m
webserver-2             1/1     Running   0          19m
```

* Another consequence of using a statefulset is that pods can communicate among them using a constant domain name as shown in the following example:
  * Note that "web-service" is the domain is the name of the service created in step 2
```
kubectl exec -it webserver-0 bash
apt-get install iputils-ping
ping webserver-0.web-service.default.svc.cluster.local
ping webserver-1.web-service.default.svc.cluster.local
ping webserver-2.web-service.default.svc.cluster.local
```

#### Cleanup

* To delete everything create so far execute the following commands

```
kubectl delete statefulset webserver
kubectl delete deployment mysql
kubectl delete service mysql8-service
kubectl delete service web-service
kubectl delete service web-service-nodeport
kubectl delete ingress usersapp-ingress
kubectl delete pvc mysql-pv-claim
kubectl delete configmap userdb-config
kubectl delete configmap webapp-home-override
kubectl delete secret db-root-pass
kubectl delete secret db-user-pass
```


# **Helm Charts**

A chart is a collection of files that describe a related set of Kubernetes resources. A single chart might be used to deploy something simple, like a memcached pod, or something complex, like a full web app stack with HTTP servers, databases, caches, and so on.

Charts are created as files laid out in a particular directory tree. They can be packaged into versioned archives to be deployed.

What we are going to do is to migrate our usersapp application to two Helm Charts, one for the web application and other for the mysql.

## **Installation**

For install instructions refer to: https://helm.sh/docs/intro/install/
Basically in Linux the procedure is downloading a tgz and installing it in the /usr/local/bin path

```
wget https://get.helm.sh/helm-v3.7.1-linux-amd64.tar.gz
gunzip ./helm-v3.7.1-linux-amd64.tar.gz
tar xvf ./helm-v3.7.1-linux-amd64.tar
cp ./linux-amd64/helm /usr/local/bin/helm371
sudo chmod ugo+x /usr/local/bin/helm371
```

## Migrate the web application to a Helm Chart

* We are going to create two charts
  * One for the website
  * One for the mysql


### Migrate the web application to a Helm Chart


#### Step 1: Create the directory structure

```
 /tmp
 |
 -- containerization
  |
  Dockerfile
  -- kubernetes
   |
   -- helm
    |
    -- webapp-chart
     |
     dev.yaml
     -- webapp
      |
      Chart.yaml
      values.yaml
      templates
```

#### Step 2: Fill Chart.yaml file

* This file has just metadata information for the chart to be installed like description, name, version, etc.

```
apiVersion: v1
appVersion: "1.0"
description: The web application
name: webapp
version: 0.1.0
```

#### Step 3: Create statefulset definition template file 

* Copy the file: ./kubernetes/webserverWithConfigMapStatefulset.yaml ./kubernetes/helm/webapp-chart/webapp/templates/statefulset.yaml and apply some changes

* Changes:
  * Replace "replicas: 3" to use a helm variable defined in a configuration file.
  * Add the resources section to define memory and cpu to use with values defined in a configuration file.
    * We are setting resources also with helm variables.

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: webserver
  labels:
    app: apache
spec:
  serviceName: web-service
  replicas: {{ .Values.webapp.replicas }}
  selector:
    matchLabels:
      app: apache
  template:
    metadata:
      labels:
        app: apache
    spec:
      containers:
      - name: userapp
        image: {{ .Values.webapp.image }}
        imagePullPolicy: IfNotPresent
        resources:
          requests:
            memory: "{{ .Values.webapp.requestMemory }}"
            cpu: "{{ .Values.webapp.requestCpu }}"
          limits:
            cpu: "{{ .Values.webapp.limitCpu }}"
            memory: "{{ .Values.webapp.limitMemory }}"
        ports:
        - containerPort: 80
        volumeMounts:
          - name: config-volume
            mountPath: /var/www/html/index.php
            subPath: indexOverride.php
      volumes:
        - name: config-volume
          configMap:
            name: webapp-home-override
```

#### Step 4: Create the service definition template file

* Copy the file: ./kubernetes/webserver-svc-headless.yaml to ./kubernetes/helm/webapp-chart/webapp/templates/service.yaml
* No changes are needed in this file.

#### Step 5: Create the ingress definition template file

* Copy the file: ./kubernetes/webserver-svc-headless.yaml to ./kubernetes/helm/webapp-chart/webapp/templates/service.yaml
* No changes are needed in this file.

#### Step 6: Create the webapp override configmap definition template file

* Create a file called: ./kubernetes/helm/webapp-chart/webapp/templates/webapp-override-config.yaml
* Fill the file with the following contents:
  * Note that in the last example we generated this definition using an imperative command, in this case we are using a declarative way of generating the configmap

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: webapp-home-override
data:
  indexOverride.php: |
    <?php
    echo "<html>";
    echo "<h1>This is an updated version of the app using a configmap (version3) </h1>";
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

#### Step 7: Fill the default configuration file (values.yaml)

* The default configuration to apply to the chart templates is written in the file values.yaml located in: ./kubernetes/helm/webapp-chart/webapp/values.yaml
* For this example, fill the file: ./kubernetes/helm/webapp-chart/webapp/values.yaml with this contents

```
webapp:
  image: usersapp:latest
  requestCpu: "0.4"
  requestMemory: "0.5Gi"
  limitCpu: "0.5"
  limitMemory: "1Gi"
  replicas: 3
```

* In this case for example we are defining webapp.image = usersapp:latest.
  * When the chart is applied to the cluster, in the statefulset.yaml defined before "{{ .Values.webapp.image }}" will be replaced for: usersapp:latest
    * Note that configuration files have a tree structure and to refer to a value we start with ".Values"

```
- name: userapp
  image: {{ .Values.webapp.image }}
  imagePullPolicy: IfNotPresent
```

#### Step 8: Fill a custom environment configuration file (dev.yaml)

* When we deploy a chart, we usually specify an environment configuration file
* This configuration will overwrite all values defined in the file values.yaml
* For our example will are going to fill this file with the following contents:

```
webapp:
  image: usersapp:latest
  requestCpu: "0.3"
  requestMemory: "0.5Gi"
  limitCpu: "0.5"
  limitMemory: "1Gi"
  replicas: 3
```

#### Step 9: Install the webapp chart in the cluster

* Now that we have generated our chart directory structure, create the templates and the configuration files we can install the chart in the cluster with just one command
* It's very important to change to the chart directory that we want to install

```
cd /tmp/containerization/kubernetes/helm/webapp-chart
helm install webapp ./webapp -f ./dev.yaml
```

* After this command is executed, all the needed objects for the webapp will be installed to the cluster

### Migrate the mysql application to a Helm Chart

#### Step 1: Create the directory structure

```
 /tmp
 |
 -- containerization
  |
  Dockerfile
  -- kubernetes
   |
   -- helm
    |
    -- mysql-chart
     |
     dev.yaml
     -- mysql
      |
      Chart.yaml
      values.yaml
      templates
```

#### Step 2: Fill Chart.yaml file

* This file has just metadata information for the chart to be installed like description, name, version, etc.

```
apiVersion: v1
appVersion: "1.0"
description: The mysql service
name: mysql
version: 0.1.0
```

#### Step 3: Create statefulset definition template file

* Copy the file: ./kubernetes/mysqlWithSecretAndConfigMap.yaml ./kubernetes/helm/mysql-chart/mysql/templates/statefulset.yaml and apply some changes

* Changes:
  * Replace the Kind: Deployment for StatefulSet
  * Add a serviceName (due that a statefulset needs a service defined)
  * Add a "resources" area to define required memory and cpu
  * Add a volumeClaimTemplates area to make the request for the PVC in the definition of the statefulset
    * If we do this we don't need to create the PVC, and it will be created automatically upon installation.

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql8-service
  selector:
    matchLabels:
      app: mysql8
  template:
    metadata:
      labels:
        app: mysql8
    spec:
      containers:
      - image: mysql:8.0
        name: mysql
        imagePullPolicy: IfNotPresent
        resources:
          requests:
            memory: "{{ .Values.mysql.requestMemory }}"
            cpu: "{{ .Values.mysql.requestCpu }}"
          limits:
            cpu: "{{ .Values.mysql.limitCpu }}"
            memory: "{{ .Values.mysql.limitMemory }}"
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-root-pass
              key: MYSQL_ROOT_PASSWORD
        - name: MYSQL_DATABASE
          valueFrom:
            configMapKeyRef:
              name: userdb-config
              key: MYSQL_DATABASE
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: db-user-pass
              key: MYSQL_USER
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-user-pass
              key: MYSQL_PASSWORD
        args: ["--default-authentication-plugin=mysql_native_password"]
        ports:
        - containerPort: 3306
          name: mysql8
        volumeMounts:
          - name: mysql-persistent-storage
            mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim-mysql-0
  volumeClaimTemplates:
  - metadata:
      name: mysql-pv-claim
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: "{{ .Values.mysql.storage }}"

```

#### Step 4: Create service definition template file

* Copy the file: ./kubernetes/mysqlWithSecretAndConfigMap.yaml ./kubernetes/helm/mysql-chart/mysql/templates/statefulset.yaml and apply some changes
* No changes are needed

#### Step 5: Create the userdb-config configmap template file

* Create a file called: ./kubernetes/helm/mysql-chart/mysql/templates/userdb-config.yaml
* Fill the file with the following contents:
  * Note that in the last example we generated this definition using an imperative command, in this case we are using a declarative way of generating the configmap

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: userdb-config
data:
  MYSQL_DATABASE: my_db
```

#### Step 7: Fill the default configuration file (values.yaml)

```
mysql:
  image: mysql:8.0
  requestCpu: "0.2"
  requestMemory: "0.5Gi"
  limitCpu: "0.5"
  limitMemory: "1Gi"
  storage: 5Gi
```

#### Step 8: Fill a custom environment configuration file (dev.yaml)

```
mysql:
  image: mysql:8.0
  requestCpu: "0.2"
  requestMemory: "0.5Gi"
  limitCpu: "0.5"
  limitMemory: "1Gi"
  storage: 5Gi
```

#### Step 9: Install the webapp chart in the cluster

* **Important Note**
  * Secrets needed by a chart are applied before the installation of the chart
  * Generally the secrets are applied with an imperative command or with a file that only an operator have
  * In this case if we are starting from scratch we need to apply the secrets before installing the mysql chart

```
kubectl create secret generic db-root-pass \
  --from-literal=MYSQL_ROOT_PASSWORD=test

kubectl create secret generic db-user-pass \
  --from-literal=MYSQL_USER=db_user \
  --from-literal=MYSQL_PASSWORD=db_password
```

* Now that we have the secrets installed we can proceed to install the chart

```
cd /tmp/containerization/kubernetes/helm/mysql-chart
helm install mysql ./mysql -f ./dev.yaml
```

#### Step 10: Fill Mysql database

```
kubectl exec -it mysql-0 bash
mysql -uroot -ptest
USE my_db;
CREATE TABLE user (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL
);
INSERT into user (name) values ("Michael");
INSERT into user (name) values ("Tony");
exit
exit
```

### Test the application

```
curl www.usersapp.net
```

### Uninstall charts & related objects

* To delete charts execute this commands
```
helm delete mysql
helm delete webapp
kubectl delete pvc mysql-pv-claim-mysql-0
kubectl delete secret db-root-pass
kubectl delete secret db-user-pass
```