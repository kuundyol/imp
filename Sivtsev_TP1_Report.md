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


## Database
Made directory for the TP: ` igors\Documents\efrei\imp\tp1\database `

Created Dockerfile.txt with following content:
```
FROM postgres:14.1-alpine

ENV POSTGRES_DB=db \
   POSTGRES_USER=usr \
   POSTGRES_PASSWORD=pwd
```

Saved the file and deleted .txt, now I have the Dockerfile

Built the image and gave it the name:
``` 
docker build -t kuundyol/tp1-database .
```

Checked the list of existing images:
``` 
docker images
```

Created network:
``` 
docker network create app-network
```

Checked the list of networks:
``` 
docker network ls
```

Run the image of the database connecting it with existing network:
```
docker run -d --name database --network app-network kuundyol/tp1-database
	# -d -> detached, allowes to still use cmd without falling into the container
```

Run the adminer image also connecting it to the network:
```
docker run -d --network app-network -p 8080:8080 -e ADMINER_PLUGINS='tables-filter tinymce' --name adminer adminer
```
	# -p -> port 8080:8080 recommended
	* -e ADMINER_PLUGINS='tables-filter tinymce'-> adding environmantal parameter is recommended

#Opened in browser http://localhost:8080/ I should see the Adminer login page.
	#System - PostgreSQL
	#Server - IP or name of database container (database)
	#Username - usr
	#Password - pwd
	#Database - db (as in Dockerfile)

	#We can also obtain ID from container running:
	$ docker inspect CONTAINER_NAME
	$ docker inspect database