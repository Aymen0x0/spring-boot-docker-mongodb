# Containerizing Spring Boot REST API using Docker
## Prerequisites

```
JDK 1.8 or later
Apache Maven 3.6.3
MongoDB (4.2)
Docker CLI (in order to be able to connect to our docker host).
Note : Before proceeding further, verify you can run docker commands from your terminal.
See https://docs.docker.com/install/ for details on setting Docker up for your machine.
```

## Installing
Download and unzip the source repository for this guide, or clone it using Git:
```
git clone https://github.com/Aymen0x0/spring-boot-rest-api-mongodb.git
```
Note : I've used my previous project named spring-boot-rest-api-mongodb, just to show how to dockernize it using Docker.

Full instructions can be found on https://github.com/Aymen0x0/spring-boot-rest-api-mongodb

 ## Deploying an Instance of MongoDB as a Container and setting up the database
1. Open a terminal and type this command :
```
docker run -d --name db mongo
```
Note : A mongoDB instance is running with default port 27017 (option -d means that a docker container runs in the background of your terminal)

2. Create your database using the interactive mode.
```
docker exec -it <container_id (first four digits) Or container_name> mongo
```
Then type the following command :
```
use MongoDbDemo
```

3. To be able to establish a connection to MongoDB from the application.

3.1 type the following command :
```
docker inspect <container_id (first four digits) Or container_name>
```
The inspect command will list the complete information of the container.

3.2 Search for :
```
"IPAddress"
```
In my case, it shows the following :
```
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
```

3.3	Then, Open src/main/resources/application.properties in order to set the database name and the host ip. It should have these following lines : 
```
spring.data.mongodb.host=172.17.0.2
spring.data.mongodb.port=27017
spring.data.mongodb.database=MongoDbDemo
spring.data.mongodb.repositories.enabled=true
```
At this point, we've linked our application to the database.
## Build and generate the JAR file for the application

Make sure that maven is installed and working correctly in your machine, alternatively you can use your IDE (such as eclipse, IntelliJ, etc.) to build and generate the JAR file for the application.

You can type the following command (If you are using Maven) :
```
mvn clean package
``` 
Note : The JAR file is generated under the target file.

## Create Docker image using Dockerfile
1. Create a new file called Dockerfile that will be located in the project’s root folder, and copy the following instructions :
```
FROM openjdk:8-jdk-alpine
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```
This Dockerfile Tell Docker that our image will be based on another image called openjdk:8 that is available on docker repository (Docker Hub).

COPY : this instruction will copy-paste the generated JAR file to the container.

ENTRYPOINT : the last instruction tell docker to run and execute the application.

2. Then, go back to the terminal and type this command:
```
docker build -t springboot-mongo .
```
This command builds an image and tags it as springboot-mongo.
## Running and Testing the application
1 . Check the available Docker images by typing the following command :
```
docker images 
```
or
```
docker image ls 
```
You should get similar output:
```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
springboot-mongo    latest              266f68dc5f4c        4 hours ago         130MB
mongo               latest              c5e5843d9f5f        6 days ago          387MB
```
2. To run the image you’ve just created use following command:
```
docker run -d -p 80:8080 springboot-mongo
```
The -p option map the port number of the container to the port number of the Docker host in order to access to the application externally.

Note : By default, Tomcat listens on port 8080

3. Check the running containers, By typing this command :
```
docker ps
```
You should see the following : 
```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                     NAMES
3c7d564cef2b        springboot-mongo    "java -jar /app.jar"     4 hours ago         Up 4 hours          0.0.0.0:80->8080/tcp    hopeful_mendeleev
cf48a9baba82        mongo               "docker-entrypoint.s…"   About a minute ago   Up 21 seconds       27017/tcp                 db
```
4. Open your Web Browser or a REST client app such as (Postman, Advanced REST Client, etc.) and type :
```
<Your docker host ip address>:80/client
```
You should normally get the results in JSON format.

The app defines the following CRUD APIs.
```
GET /client
POST /client/add
GET /client/{clientId}
PUT /client/update/{clientId}
DELETE /client/delete/{clientId}
```
## Author

* **Aymen Mankari** - [Aymen0x0](https://github.com/Aymen0x0)
