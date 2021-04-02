
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
4.Create jar files for the backend microservices .
````
mvn clean package install spring-boot:repackage -Pprod
````
5.Write Dockerfile for frontend and place it in UPSTAC-Microservices-Frontend.
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
6.Write Dockerfile's for backend microservice's(document,master & register )and place them in corresponding folders in UPSTAC-Microservices-Backend.
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
7.Create docker image for frontend.

````
UPSTAC-Microservices-Frontend$sudo docker build -t vaishalinankani08/upstac-frontend .
````
8.Create docker image's for backend microservices i.e. document , master & registeration.

````
UPSTAC-Microservices-Backend/upstac-api-document$sudo docker build -t vaishalinankani08/upstac-document .
UPSTAC-Microservices-Backend/upstac-api-register$sudo docker build -t vaishalinankani08/upstac-registration .
UPSTAC-Microservices-Backend/upstac-api-master$sudo docker build -t vaishalinankani08/upstac-master .
````
9.Install aws cli on ubuntu compute instance
````
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
````
10.Create IAM role with AmazonEC2ContainerRegistryFullAccess  policy
11.
<img width="862" alt="iamrole" src="https://user-images.githubusercontent.com/77958988/113383306-49140a80-93a1-11eb-9a5e-5cf6b55e7157.png">


<img width="800" alt="iampolicy" src="https://user-images.githubusercontent.com/77958988/113383864-6eeddf00-93a2-11eb-92f0-1e1d50c4de13.png">


<img width="726" alt="rolecreate" src="https://user-images.githubusercontent.com/77958988/113383887-8036eb80-93a2-11eb-845f-a3dce337250b.png">


<img width="936" alt="rolescraeted" src="https://user-images.githubusercontent.com/77958988/113383436-8bd5e280-93a1-11eb-9838-c84eb7c02b8b.png">


<img width="942" alt="attachroletoinstance" src="https://user-images.githubusercontent.com/77958988/113383985-af4d5d00-93a2-11eb-8377-60c7463bd876.png">


<img width="702" alt="attachroletoinstance2" src="https://user-images.githubusercontent.com/77958988/113384020-c5f3b400-93a2-11eb-94ab-66956128f437.png">


<img width="796" alt="attachroletoinstance3" src="https://user-images.githubusercontent.com/77958988/113384062-dc017480-93a2-11eb-98a0-e05c9fcc478b.png">






