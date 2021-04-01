
# Introduction
This learning path helps you deploy microservice architecture based UPSTAC application on AWS ECS.
This application uses MySQL database ,Frontend is in reactjs and Backend microservice's are written in springboot.


References:

Original github link:

https://github.com/upgrad-edu/UPSTAC-Microservices-Frontend

https://github.com/upgrad-edu/UPSTAC-Microservices-Backend/

1.Ubuntu 20.04 compute instance is used for this exercise
<img width="932" alt="ubuntuami" src="https://user-images.githubusercontent.com/77958988/113335071-1686f500-9342-11eb-8ca3-2e36b6a3260d.png">

<img width="935" alt="ubuntuinstancetype" src="https://user-images.githubusercontent.com/77958988/113335439-8f864c80-9342-11eb-8863-9b09bb1e8f56.png">


<img width="913" alt="storage" src="https://user-images.githubusercontent.com/77958988/113336199-a37e7e00-9343-11eb-8d49-9caf11e6d46a.png">


2.Clone Frontend and backend code of UPSTAC application
````
ubuntu@ip-172-31-47-81:~$ git clone https://github.com/upgrad-edu/UPSTAC-Microservices-Frontend.git
Cloning into 'UPSTAC-Microservices-Frontend'...
remote: Enumerating objects: 170, done.
remote: Counting objects: 100% (170/170), done.
remote: Compressing objects: 100% (142/142), done.
remote: Total 170 (delta 18), reused 170 (delta 18), pack-reused 0
Receiving objects: 100% (170/170), 473.92 KiB | 18.23 MiB/s, done.
Resolving deltas: 100% (18/18), done.
ubuntu@ip-172-31-47-81:~$ ls
UPSTAC-Microservices-Frontend
ubuntu@ip-172-31-47-81:~$ git clone https://github.com/upgrad-edu/UPSTAC-Microservices-Backend.git
Cloning into 'UPSTAC-Microservices-Backend'...
Username for 'https://github.com': vaishalinankani08
Password for 'https://vaishalinankani08@github.com':
remote: Enumerating objects: 368, done.
remote: Counting objects: 100% (368/368), done.
remote: Compressing objects: 100% (217/217), done.
remote: Total 368 (delta 135), reused 337 (delta 110), pack-reused 0
Receiving objects: 100% (368/368), 132.16 KiB | 12.01 MiB/s, done.
Resolving deltas: 100% (135/135), done.
ubuntu@ip-172-31-47-81:~$ ls
UPSTAC-Microservices-Backend  UPSTAC-Microservices-Frontend


````
3.Install Maven and docker (https://docs.docker.com/engine/install/ubuntu/) on ubuntu.

Maven installation:
````
sudo apt update
sudo apt install maven
````
4.Write Dockerfile for frontend and place it in UPSTAC-Microservices-Frontend.
Dockerfile:
````
FROM node:10
WORKDIR /opt/app
COPY package*.json ./
COPY . .
RUN CI=true
# Installs all node packages
RUN npm install
# Copies everything over to Docker environment
# Uses port which is used by the actual application
EXPOSE 3000
# Finally runs the application
CMD [ "npm", "start" ]
````
5.Write Dockerfile's for backend microservice's(document,master & register )and place them in corresponding folders in UPSTAC-Microservices-Backend.
Dockerfile:
````
FROM openjdk:14-jdk-alpine
MAINTAINER upgrad
ADD ./target/upstac-api-0.0.1-SNAPSHOT.jar /opt/app/upstac-api-0.0.1-SNAPSHOT.jar
WORKDIR /opt/app
ENV PATH="${PATH}:${JAVA_HOME}/bin"
EXPOSE 8080
ENTRYPOINT [ "java", "-jar", "/opt/app/upstac-api-0.0.1-SNAPSHOT.jar"]

````
6.Create docker image for frontend.

````
UPSTAC-Microservices-Frontend$sudo docker build -t vaishalinankani08/upstac-frontend .
````
7.Create docker image's for backend microservices i.e. document , master & registeration.

````
UPSTAC-Microservices-Backend/upstac-api-document$sudo docker build -t vaishalinankani08/upstac-document .
UPSTAC-Microservices-Backend/upstac-api-register$sudo docker build -t vaishalinankani08/upstac-registration .
UPSTAC-Microservices-Backend/upstac-api-master$sudo docker build -t vaishalinankani08/upstac-master .
````
8.

