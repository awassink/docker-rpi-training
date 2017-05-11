# Docker hands on training

## Part 1 Getting familiar with Docker containers

In the following steps we will learn the basics of Docker.

### Step 1
Use `docker help` and `docker help <cmd>` for information about the possibilities of docker. 
Important commands are `docker images` to get a list with the current images, `docker ps` to get a list with de current active containers and `docker ps -a` to get a list with all the containers including the inactive.

### Step 2
Use `docker run` to start a container using image `buildserver:5000/rpi-nginx` and make it available at port 88 on the host system. 
Use the option `-d` to run the server as daemon process in the backgound.
Use the option `-p host-port:container-port` for port mapping. 
The nginx process will serve its webserver at port 80 in the container. 
Use `—name <container-name>` to give your container a manageable name in addition to the random ID. 
Use a browser to access the nginx webserver <http://<dockerhost>:88/>.

### Step 3
Use `docker exec -ti <container-id> bash` to open a shell in the active container. 
Take a look at; the filesystem with `ls`, the active processes with `ps -ef`, the hostname with `hostname` and the assigned ip-address of the container with `hostname -i`. (end shell with exit)

### Step 4
Stop the docker container with `docker stop`. check that the webserver is not available anymore using the browser.
With `docker ps -a` you can see the containers (active and not active).
Every time you execute `docker run` a new container gets created. 
After a run you can run the same container with the assigned container id: `docker start <container-id>` 
This can be handy to prevent the creation of a huge amount of containers. 
With the flag -d the container runs as a daemon process in the background and you need to stop it explicitly. Restart the container now.

### Step 5
Check the local available docker images with `docker images`.
Check the image layering with `docker history`.

### Step 6
Remove the previously started docker container with `docker rm`. 
Use the `-f` option if the container is still running.
Remove the `buildserver:5000/rpi-nginx` image with `docker rmi`. 
This only works if the containers based on the image are removed. 
If this is not the case, use `docker ps -a` and `docker rm` to remove the remaining.

### Step 7
We are going to do something more exiting, we are going to serve our own (static) webpage located in the `docker-workshop/web-app/` folder from the nginx container. 
Start a new container with a volume mount to `/usr/share/nginx/html` use `-v host-folder:container-folder:mountoptions`. 
In this case you can use `ro` for mountoptions, so it’s read-only. 
Do NOT use relative paths, but the full paths.
Check the webpage with the browser.

## Part 2 Creating docker images

In this part of the workshop we will be deploying a three tier application that comprises of a web server hosting an angular web application, a Java
based REST service and a mysql database. In this part of the workshop we will create a Docker image for the frontend and backend layer. There
is no dedicated Docker image for de database layer needed as the database tables of the application will be created by the hibernate layer of the
REST service upon startup, and the database can be created by the docker image it self.

### Step 1. Run MySQL

The Java REST service expects a mysql database with the name cddb_quintor and a user with the same name and quintor_pw as password.

MYSQL_ROOT_PASSWORD=quintor_pw
MYSQL_DATABASE=cddb_quintor
MYSQL_USER=cddb_quintor
MYSQL_PASSWORD=quintor_pw

Use the standard version 5.6 mysql image from Docker hub (https://hub.docker.com/_/mysql/) for this. There are environment variables available to let the mysql docker container create the database and an user. Give the mysql container the name cddb_mysql so it can be linked later on. Bind the
mysql port (3306) to a local port and check that the database is available.

### Step 2. Create an docker image of the Java REST service

The Java REST service is a simple JavaEE web app that is ready available in the three-tier-app/backend folder. This WAR file can be run with
tomcat 8.5, so create a Docker image based on https://hub.docker.com/_/tomcat/ using a Docker file. Copy the war file to /usr/local/tomcat/webapps/cddb.war in the image.

Link the cddb_mysql container to mysql so
the application can connect to mysql by using the link option: --link cddb_mysql:mysql. Give the backend container the name cddb_backend so it can be linked later on. Bind the tomcat port
(8080) to a local port and check that the REST service is available using a browser or other tool (http://<dockerhost>:<bindport>/cddb/rest/).

### Step 3. Create an Docker image of the Angular web app

The web application is a simple Angular application compising only of static files to be served. This can be done easily using nginx and copy the webapp content.
COPY ./ /usr/share/nginx/html
To make the REST endpoints easily accessible form the browser an nginx configuration is provided (nginx.conf) to create a reverse proxy that proxies the
REST backend service.
COPY nginx.conf /etc/nginx/
Therefor also link the cddb_backend container to cddb_backend so the nginx reverse proxy can connect to the Java
backendby using the link option: --link cddb_backend:cddb_backend. Bind the nginx port (80) to a local port and check that the web application is available and working using a browser (http://<dockerhost>:
<bindport>/).

### Step 4. Use volume mounting to store the mysql data locally.

After removing the mysql container the stored data should be available again when a new container is create with the same volume mount. 
Use the `docker run` volume mounting option `-v` to store the mysql data on directory on the host system.

## Part 3 Use Docker Compose to run the three tier application at once

Docker compose give you the possibility to run multiple containers that are depended on each other at once. Command line options that are
needed to link containers, mount volumes and expose ports can be defined is one configuation file.

Create a docker compose yaml configuration file to start up the three tier application at once. Documentation can be found here: https://docs.dock
er.com/compose/overview/
