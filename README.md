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
Full instructions can be found on https://github.com/Aymen0x0/spring-boot-rest-api-mongodb

## User-defined bridge
Using a user-defined network provides a scoped network in which only containers attached to that network are able to communicate.

In order to achieve this, open your Terminal and type :

```
docker network create mynetwork
```
## Deploying an Instance of MongoDB as a Container and setting up the database
1. Open a terminal and type this command :
```
docker run -d -e MONGO_INITDB_DATABASE=MongoDbDemo --name my_mongo_host --network mynetwork mongo
```

Note : A mongoDB instance is running with default port 27017.

The ``` -d ``` option means that a docker container will run in the background of your terminal.

The Environment Variable ``` -e MONGO_INITDB_DATABASE ``` allows us to specify the name of the database (In my case it's named MongoDbDemo).

The ``` --name ``` option allows us to assign a name to the container. So, you can use it when referencing the container within a Docker network.

2. Then, Open src/main/resources/application.properties in order to set the database name and the host name. It should have these following lines : 
```
spring.data.mongodb.host=my_mongo_host
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

``` COPY : ``` this instruction will copy-paste the generated JAR file to the container.

``` ENTRYPOINT : ``` this instruction tells docker to run and execute the application.

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
2. To run an instance of the image that we've just created using Dockerfile, type the following command:
```
docker run -d -p 9091:8080 --network mynetwork springboot-mongo
```
The ```-p``` option map the port number of the container to the port number of the Docker host in order to get access to the application externally.

The ```--network ``` option links the container to the network that we've created at the beginning of this guide.

Note : By default, Tomcat listens on port 8080

3. Check the running containers, By typing this command :
```
docker ps
```
You should see the following : 
```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
f5a13241eeb4        springboot-mongo    "java -jar /app.jar"     About an hour ago   Up About an hour    0.0.0.0:9091->8080/tcp   nervous_margulis
e0f7d3ba530b        mongo               "docker-entrypoint.s…"   About an hour ago   Up About an hour    27017/tcp                my_mongo_host
```
4. To get access to the API externally, open your Web Browser or a REST client app such as (Postman, Advanced REST Client, etc.) and type :
```
<Your docker host ip address>:9091/client
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
