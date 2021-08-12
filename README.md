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
FROM php:apache
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

####Step 1: Create a file: /tmp/containerization/kubernetes/webserver.yaml with the contents:
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

####Step 1: Create a file: /tmp/containerization/kubernetes/mysql-pvc.yaml with the contents:
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
        imagePullPolicy: Never
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












  




  


