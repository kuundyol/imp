# Discover Docker

link: http://school.pages.takima.io/devops-resources/

### Goals
* Good practice
* Do not forget to document what you do along the steps, the documentation provided will be evaluated as your report. 
* Create an appropriate file structure, 1 folder per image.

### Target application
3-tiers application:
* HTTP server
* Backend API
* Database
For each of those applications, we will follow the same process: choose the appropriate docker base image, create and configure this image, put our application specifics inside and at some point have it running. Our final goal is to have a 3-tier web API running.


# Database
Made directory for the TP: ` igors\Documents\efrei\imp\tp1\database `

Created Dockerfile.txt with following content:
````docker
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd
````

Saved the file and deleted .txt, now I have the Dockerfile

Built the image and gave it the name:
````docker
docker build -t kuundyol/tp1-database .
````

Checked the list of existing images:
```docker
docker images
```

Created network:
```docker
docker network create app-network
```

Checked the list of networks:
```docker
docker network ls
```

Run the image of the database connecting it with existing network:
```docker
docker run -d --name database --network app-network kuundyol/tp1-database
	# -d -> detached, allowes to still use cmd without falling into the container
```

Run the adminer image also connecting it to the network:
```docker
docker run -d --network app-network -p 8080:8080 -e ADMINER_PLUGINS='tables-filter tinymce' --name adminer adminer
    # -p -> port 8080:8080 recommended
    # -e ADMINER_PLUGINS='tables-filter tinymce'-> adding environmantal parameter is recommended
```

Opened in browser http://localhost:8080/ I should see the Adminer login page.
	
* System - PostgreSQL
* Server - IP or name of database container (database)
* Username - usr
* Password - pwd
* Database - db (as in Dockerfile)

We can also obtain ID from container running:

````docker
docker inspect CONTAINER_NAME
docker inspect database
````

Adding initial data.
Created two sql files in /database directory:

* First with the name CreateScheme.sql and content:

````sql
CREATE TABLE public.departments
(
 id      SERIAL      PRIMARY KEY,
 name    VARCHAR(20) NOT NULL
);

CREATE TABLE public.students
(
 id              SERIAL      PRIMARY KEY,
 department_id   INT         NOT NULL REFERENCES departments (id),
 first_name      VARCHAR(20) NOT NULL,
 last_name       VARCHAR(20) NOT NULL
);
````

* Second with the name InsertData.sql and content:
````sql
INSERT INTO departments (name) VALUES ('IRC');
INSERT INTO departments (name) VALUES ('ETI');
INSERT INTO departments (name) VALUES ('CGP');


INSERT INTO students (department_id, first_name, last_name) VALUES (1, 'Eli', 'Copter');
INSERT INTO students (department_id, first_name, last_name) VALUES (2, 'Emma', 'Carena');
INSERT INTO students (department_id, first_name, last_name) VALUES (2, 'Jack', 'Uzzi');
INSERT INTO students (department_id, first_name, last_name) VALUES (3, 'Aude', 'Javel');
````

Rebuilt the image (I created a new image):

````docker
docker build -t kuundyol/tp1-database-filled .
````

Stopped the running database container:

````docker
docker stop database
````

Run the container of the new image:

````docker
docker run -d --name database-filled --network app-network kuundyol/tp1-database-filled
````

It is connected to the same network

Updated the localhost page and logged in again. Now I see the filled tables.

## Data percistency.

### 1. Saving files locally.

Removed the container

````docker
docker rm -f database-filled
````

Created the `directory\igors\Documents\efrei\imp\tp1\database\persist-data`

Opened the database directory with the Visual Studio Code. Copied the full path to the directory persist-data.

Re-run the container binding it with the local volume directory:

````docker
docker run -d --name database-filled --network app-network -v C:/Users/igors/Documents/efrei/imp/tp1/database/persist-data:/var/lib/postgresql/data kuundyol/tp1-database-filled
````

Went to localhost:8080, opened Adminer, logged into the database. Commited changes (created a new test table).

Kill the database container:

````docker
docker rm -f database-filled
````

Re-run the container binding it with the local volume directory:

````docker
docker run -d --name database-filled --network app-network -v C:/Users/igors/Documents/efrei/imp/tp1/database/persist-data:/var/lib/postgresql/data kuundyol/tp1-database-filled
````

Checked that the changes (test table) are still there. Success!!

### 2. Saving files on Docker volume

Create volume: 

````docker
docker volume create database-volume
````

Re-run the container binding it with the Docker volume:

````docker
docker run -d --name database-filled --network app-network -v database-volume:/var/lib/postgresql/data kuundyol/tp1-database-filled
````

Commited changes to the database (created a new test table)

Killed the database container:

````docker
docker rm -f database-filled
````

Re-run the container binding it with the Docker volume:

````docker
docker run -d --name database-filled --network app-network -v database-volume:/var/lib/postgresql/data kuundyol/tp1-database-filled
````

Checked that the changes (test table) are still there. Success!!


## Backend API

### Basics

Created new directory: `C:\Users\igors\Documents\efrei\imp\tp1\backend`

Move terminal to this directory

In this directory created file Main.java with content:

````java
public class Main {

   public static void main(String[] args) {
       System.out.println("Hello World!");
   }
}
````

Compiled the java document using console command:

````bash
javac Main.java
````

It created a compiled file Main.class

Created Dockerfile with content:

````docker
#Choose a java JRE
FROM openjdk:11-jre

#Add the compiled java
COPY Main.class .
	#COPY puts from.. to.., so first I need the file name and then put a dot as a relative path

#Run the Java with 'java Main'
CMD java Main
````

Built an image:

````docker
docker build -t kuundyol/tp1-backend .
````

Run the container:

````docker
docker run --name javaHelloWorld kuundyol/tp1-backend
````

I could see:
>Hello world!

'Hello world!' shows that container works correctly

Do not put `-d` here, you want to stay in the container to receive a message

### Springboot

Went to link https://start.spring.io/

Put following configuration
* Project: Maven
* Language: Java 17
* Sring Boot: 2.7.6
* Packaging: Jar
* Dependencies: Spring Web

Generated the project

Put the project into the directory: `C:\Users\igors\Documents\efrei\imp\tp1\springboot`

Extracted the zip project file into the folder

Opened the project with Visual Studio Code

Under `src/main/java/...` created new file GreetingController.java with content:

````java
package com.example.demo; #changed the package here

import org.springframework.web.bind.annotation.*;

import java.util.concurrent.atomic.AtomicLong;

@RestController
public class GreetingController {

   private static final String template = "Hello, %s!";
   private final AtomicLong counter = new AtomicLong();

   @GetMapping("/")
   public Greeting greeting(@RequestParam(value = "name", defaultValue = "World") String name) {
       return new Greeting(counter.incrementAndGet(), String.format(template, name));
   }

   class Greeting {

       private final long id;
       private final String content;

       public Greeting(long id, String content) {
           this.id = id;
           this.content = content;
       }

       public long getId() {
           return id;
       }

       public String getContent() {
           return content;
       }
   }
}
````

Saved the new file

Went to the project folder (demo)

Created here the Dockerfile.txt with content:

````docker
#Build
FROM maven:3.8.6-amazoncorretto-17 AS myapp-build
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY pom.xml .
COPY src ./src
RUN mvn package -DskipTests

#Run
FROM amazoncorretto:17
ENV MYAPP_HOME /opt/myapp
WORKDIR $MYAPP_HOME
COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar

ENTRYPOINT java -jar myapp.jar
````

Saved the file as Dockerfile

Move terminal to this directory (demo)

Built the image:
````docker
docker build -t kuundyol/tp1-springboot .
````

Run the container connecting with the port
````docker
docker run -p 8080:8080 --name springboot kuundyol/tp1-springboot
````

Opened the localhost:8080 in the browser. It showed: 
>{"id":1,"content":"Hello, World!"}

When updating the page it autoincrements and id grows by 1

When put the name parameter into the URL: http://localhost:8080/?name=Pia
>{"id":2,"content":"Hello, Pia!"}

### Simple API
Downloaded the zip code from the link https://github.com/takima-training/simple-api-student

Extracted the files and opened the folder with VS Code

Adjusted the configuration in `simple-api/src/main/resources/application.yml` :
````yml
datasource:
    url: jdbc:postgresql://database-filled/db # jdbc url jdbc:postgresql://<name of database container>/<name of database>
    username: usr #no braces
    password: pwd #no braces
    driver-class-name: org.postgresql.Driver
````

Copied into the code folder the Dockerfile from Springboot previous exercise

Moved the powershell to this code directory `\igors\Documents\efrei\imp\tp1\simple-api-student-main`

Built an image:
````docker
docker build -t kuundyol/simpleapi .
````

Run the backend API connecting to the network:
````docker
docker run -d --name simpleapi --network app-network -p 8080:8080 kuundyol/simpleapi
````

Run the filled database connecting it to the same network:
````docker
docker run -d --name database-filled --network app-network kuundyol/tp1-database-filled
````

Went to localhost:8080 in browser and saw:
>{"id":19,"content":"Hello, World!"}

Went to http://localhost:8080/departments/IRC/students and saw:
>[{"id":1,"firstname":"Eli","lastname":"Copter","department":{"id":1,"name":"IRC"}}]

Success! API is working!


# HTTP server

## Basics

Created directory `\igors\Documents\efrei\imp\tp1\http`

Mover Powershell to this directory

Created directory `\igors\Documents\efrei\imp\tp1\http\public-html`

In this directory created index.htmp landing page file: `\igors\Documents\efrei\imp\tp1\http\public-html\index.html`

Filled index.html file with the following content:
````html
<!DOCTYPE html>
<html lang ="en">
<head>Hello World!</head>
<body>Here is my landing page :)</body>
</html>
````

In http root project directory created Dockerfile with content:
````docker
#getting base http image
FROM httpd:2.4
COPY ./public-html/ /usr/local/apache2/htdocs/
#got this code from https://hub.docker.com/_/httpd
````

Built the image:
````docker
docker build -t kuundyol/httpserver .
````

Run the container:
````docker
docker run -dit --name my-http-server -p 8080:80 kuundyol/httpserver
````

Went to localhost:8080 and saw the landing page with content:
>Hello World! Here is my landing page :)

Checked the container:
````docker
docker stats my-http-server #shows stats in real time, be careful
docker inspect my-http-server
docker logs my-http-server
````


## Configuration 

To retrieve current configuration I run:
````docker
docker exec -it my-http-server "/bin/bash"
#It load the current configuration
````

Created the file httpd.conf in the project root directory. Put the loaded configuration inside of it. (see the copy file)

Deleted the current container
````docker
docker rm -f my-http-server
````

Edited the Dockerfile, added COPY
````docker
#getting base http image
FROM httpd:2.4
COPY ./public-html/ /usr/local/apache2/htdocs/

COPY httpd.conf /usr/local/apache2/conf/
````

Rebuilt the image
````docker
docker build -t kuundyol/httpserver .
````

Run the container
````docker
docker run -d --name my-http-server -p 8080:80 kuundyol/httpserver
	# -dit stops the container, don't do it
	# -p 8080:80 ports are important
````

Checked the localhost in the browser again. Everything works, I see the landing page:
>Hello World! Here is my landing page :)


## Reverse proxy

Uncommented lines in httpd.conf file:
````conf
	LoadModule proxy_module modules/mod_proxy.so
	LoadModule proxy_connect_module modules/mod_proxy_connect.so
	LoadModule proxy_ftp_module modules/mod_proxy_ftp.so
	LoadModule proxy_http_module modules/mod_proxy_http.so
````

Added to the httpd.conf configuration file at the end:
````bash
<VirtualHost *:80>
ProxyPreserveHost On
ProxyPass / http://springboot:8080/			#springboot - backend running container name
ProxyPassReverse / http://springboot:8080/		#springboot - backend running container name
</VirtualHost>
````


Rebuilt the kuundyol/httpserver image with new configuration:
````docker
docker build -t kuundyol/httpserver .
````

Re-run the http container connecting it to the network:
````docker
docker run -d --name my-http-server -p 8080:80 --network app-network  kuundyol/httpserver
# -p 8080:80 port is important
````

Run the springboot backend container connecting it to the same network:
````docker
docker run -d --network app-network --name springboot kuundyol/tp1-springboot
````

Went to localhost:8080 and checked that everything works. It showed springboot api:
>{"id":2,"content":"Hello, World!"}

Success!!

Checked what happens if backend is stopped. Stopped the backend:
````docker
$ docker stop springboot
````

Uploaded the localhost:8080 page. It showed:
>Service Unavailable
The server is temporarily unable to service your request due to maintenance downtime or capacity problems. Please try again later.



# Link application

Checked that I have docker-compose:
````docker
docker compose version

> Docker Compose version v2.7.0
````

## Manually linked the applications. 

First, made sure all the names and connections are correct.

Changed the linking in the configuration file httpd.conf :
````conf
<VirtualHost *:80>
ProxyPreserveHost On
ProxyPass / http://simpleapi:8080/
ProxyPassReverse / http://simpleapi:8080/
</VirtualHost>
````

Rebuilt the http server image:
````
docker build -t kuundyol/httpserver .
````

Removed all the current containers

Run all the containers linking to the same network:
````docker
docker run -d --name database-filled --network app-network kuundyol/tp1-database-filled

docker run -d --name simpleapi --network app-network kuundyol/simpleapi

docker run -d --name my-http-server --network app-network -p 8080:80 kuundyol/httpserver
	#connecting only the http server to the port
	#be careful with the container names!! They are hardcoded into the Dockerfile and configuration file!!
````

Went to the localhost:8080 and checked that everything works
In browser I saw:
>{"id":19,"content":"Hello, World!"}

Went to http://localhost:8080/departments/IRC/students and saw:
>[{"id":1,"firstname":"Eli","lastname":"Copter","department":{"id":1,"name":"IRC"}}]

## Making the compose file

Moved powershell to `C:\Users\igors\Documents\efrei\imp\tp1`

Created here docker-compose.yml file with content:
````yml
version: '3.7'

services:
    backend:
        build:
            ./simple-api-student-main
        networks:
            - my-network
        depends_on:
            - database

    database:
        build:
            ./database
        networks:
            - my-network

    httpd:
        build:
            ./http
        ports:
            - 8080:80
        networks:
            - my-network
        depends_on:
            - backend

networks:
    my-network:
````

Changed the container names in the hardcode

`C:\Users\igors\Documents\efrei\imp\tp1\simple-api-student-main\src\main\resources\application.yml`:
````yml
url: jdbc:postgresql://database/db 
# jdbc url jdbc:postgresql://<name of database container>/<name of database>
````

`C:\Users\igors\Documents\efrei\imp\tp1\http`:
````conf
<VirtualHost *:80>
ProxyPreserveHost On
ProxyPass / http://backend:8080/
ProxyPassReverse / http://backend:8080/
</VirtualHost>
````

In powershell run command:
````docker
docker compose up --build --force-recreate
	#build and force recreate are here to force Docker build new image and containers every time
````
Now I can go to the localhost:8080 and see that everything works. Yayy!

# Publish
Your docker images are stored locally, let's publish them, so they can be used by other team members or on other machines.

Login to the docker hub. Link: https://hub.docker.com/

Login in powershell
````docker
docker login
````

Tag the image (give the name and version to the image)
````docker
docker tag tp1_database kuundyol/my-database:1.0
````

Push this tagged repository online
````docker
docker push kuundyol/my-database:1.0
````
Now I sse my image in the list of repositories on my Docker account. Success!!

TP1 finished!