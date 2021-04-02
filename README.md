
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

<img width="936" alt="iamrole" src="https://user-images.githubusercontent.com/77958988/113383306-49140a80-93a1-11eb-9a5e-5cf6b55e7157.png">


<img width="936" alt="iampolicy" src="https://user-images.githubusercontent.com/77958988/113383864-6eeddf00-93a2-11eb-92f0-1e1d50c4de13.png">


<img width="936" alt="rolecreate" src="https://user-images.githubusercontent.com/77958988/113383887-8036eb80-93a2-11eb-845f-a3dce337250b.png">


<img width="936" alt="rolescraeted" src="https://user-images.githubusercontent.com/77958988/113383436-8bd5e280-93a1-11eb-9838-c84eb7c02b8b.png">


<img width="936" alt="attachroletoinstance" src="https://user-images.githubusercontent.com/77958988/113383985-af4d5d00-93a2-11eb-8377-60c7463bd876.png">


<img width="936" alt="attachroletoinstance2" src="https://user-images.githubusercontent.com/77958988/113384020-c5f3b400-93a2-11eb-94ab-66956128f437.png">


<img width="936" alt="attachroletoinstance3" src="https://user-images.githubusercontent.com/77958988/113384062-dc017480-93a2-11eb-98a0-e05c9fcc478b.png">



11.Create repository for registration microservice of UPSTAC Application and push image to repository
````
ubuntu@ip-172-31-47-81:~$ aws ecr create-repository --repository-name registration --region us-east-1
{
    "repository": {
        "repositoryArn": "arn:aws:ecr:us-east-1:976591367883:repository/registration",
        "registryId": "976591367883",
        "repositoryName": "registration",
        "repositoryUri": "976591367883.dkr.ecr.us-east-1.amazonaws.com/registration",
        "createdAt": "2021-04-02T05:10:30+00:00",
        "imageTagMutability": "MUTABLE",
        "imageScanningConfiguration": {
            "scanOnPush": false
        },
        "encryptionConfiguration": {
            "encryptionType": "AES256"
        }
    }
}
ubuntu@ip-172-31-47-81:~$ sudo docker tag 9968ab95c134 976591367883.dkr.ecr.us-east-1.amazonaws.com/registration
ubuntu@ip-172-31-47-81:~$ aws ecr get-login-password | sudo docker login --username AWS --password-stdin 976591367883.dkr.ecr.us-east-1.amazonaws.com/registration
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
ubuntu@ip-172-31-47-81:~$ sudo docker push 976591367883.dkr.ecr.us-east-1.amazonaws.com/registration
Using default tag: latest
The push refers to repository [976591367883.dkr.ecr.us-east-1.amazonaws.com/registration]
7b91809557c5: Pushed
f199ad476e12: Pushed
531743b7098c: Pushed
latest: digest: sha256:a06813987fe12f3397df3cc24662ab7d057043ac86be83a01921dc15ec73e922 size: 953

````

12.Create repository for master microservice of UPSTAC Application and push image to repository

````
ubuntu@ip-172-31-47-81:~$ aws ecr create-repository --repository-name master --region us-east-1
{
    "repository": {
        "repositoryArn": "arn:aws:ecr:us-east-1:976591367883:repository/master",
        "registryId": "976591367883",
        "repositoryName": "master",
        "repositoryUri": "976591367883.dkr.ecr.us-east-1.amazonaws.com/master",
        "createdAt": "2021-04-02T06:31:46+00:00",
        "imageTagMutability": "MUTABLE",
        "imageScanningConfiguration": {
            "scanOnPush": false
        },
        "encryptionConfiguration": {
            "encryptionType": "AES256"
        }
    }
}
ubuntu@ip-172-31-47-81:~$ sudo docker tag 8d430c791a5c  976591367883.dkr.ecr.us-east-1.amazonaws.com/master
ubuntu@ip-172-31-47-81:~$ aws ecr get-login-password | sudo docker login --username AWS --password-stdin 976591367883.dkr.ecr.us-east-1.amazonaws.com/master
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
ubuntu@ip-172-31-47-81:~$ sudo docker push 976591367883.dkr.ecr.us-east-1.amazonaws.com/master
Using default tag: latest
The push refers to repository [976591367883.dkr.ecr.us-east-1.amazonaws.com/master]
1e21866ae6bb: Pushed
f199ad476e12: Pushed
531743b7098c: Pushed
latest: digest: sha256:f1d222b32a2ee0f966464622703d9ed99f69d1da32af44e8cc935b862e5f2be3 size: 953


````
13.Create repository for document microservice of UPSTAC Application and push image to repository

````
ubuntu@ip-172-31-47-81:~$ aws ecr create-repository --repository-name document --region us-east-1
{
    "repository": {
        "repositoryArn": "arn:aws:ecr:us-east-1:976591367883:repository/document",
        "registryId": "976591367883",
        "repositoryName": "document",
        "repositoryUri": "976591367883.dkr.ecr.us-east-1.amazonaws.com/document",
        "createdAt": "2021-04-02T06:34:50+00:00",
        "imageTagMutability": "MUTABLE",
        "imageScanningConfiguration": {
            "scanOnPush": false
        },
        "encryptionConfiguration": {
            "encryptionType": "AES256"
        }
    }
}
ubuntu@ip-172-31-47-81:~$ sudo docker tag c8cab07c2755  976591367883.dkr.ecr.us-east-1.amazonaws.com/document
ubuntu@ip-172-31-47-81:~$ aws ecr get-login-password | sudo docker login --username AWS --password-stdin 976591367883.dkr.ecr.us-east-1.amazonaws.com/document
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
ubuntu@ip-172-31-47-81:~$ sudo docker push 976591367883.dkr.ecr.us-east-1.amazonaws.com/document
Using default tag: latest
The push refers to repository [976591367883.dkr.ecr.us-east-1.amazonaws.com/document]
0409820a92ac: Pushed
f199ad476e12: Pushed
531743b7098c: Pushed
latest: digest: sha256:08905111cc99a86c76ae02613fc9d04b51123b46104f9baf7556a7131a2c5eef size: 953
ubuntu@ip-172-31-47-81:~$

````
14.Create repository for frontend microservice of UPSTAC Application and push image to repository
````
ubuntu@ip-172-31-47-81:~$ aws ecr create-repository --repository-name frontend --region us-east-1
{
    "repository": {
        "repositoryArn": "arn:aws:ecr:us-east-1:976591367883:repository/frontend",
        "registryId": "976591367883",
        "repositoryName": "frontend",
        "repositoryUri": "976591367883.dkr.ecr.us-east-1.amazonaws.com/frontend",
        "createdAt": "2021-04-02T06:38:31+00:00",
        "imageTagMutability": "MUTABLE",
        "imageScanningConfiguration": {
            "scanOnPush": false
        },
        "encryptionConfiguration": {
            "encryptionType": "AES256"
        }
    }
}
ubuntu@ip-172-31-47-81:~$ sudo docker tag 120bc5e8ef65   976591367883.dkr.ecr.us-east-1.amazonaws.com/frontend
ubuntu@ip-172-31-47-81:~$ aws ecr get-login-password | sudo docker login --username AWS --password-stdin 976591367883.dkr.ecr.us-east-1.amazonaws.com/frontend
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
ubuntu@ip-172-31-47-81:~$ sudo docker push 976591367883.dkr.ecr.us-east-1.amazonaws.com/frontend
Using default tag: latest
The push refers to repository [976591367883.dkr.ecr.us-east-1.amazonaws.com/frontend]
63e7fa92ec01: Pushed
0768e4fa7453: Pushed
45575915d9df: Pushed
59e1fbdfa06e: Pushed
e633f0a01c11: Pushed
42993f010c69: Pushed
532218acbf39: Pushed
e34081af2c41: Pushed
25f2c43959c6: Pushed
28f17d067081: Pushed
f56d0812d4cc: Pushed
0919c6356f3e: Pushed
30840de9c53a: Pushed
latest: digest: sha256:1011d58e07470cd8cd928755d766ffb9cd8284c21e722198ed00e9fd89b4313b size: 3053
ubuntu@ip-172-31-47-81:~$

````
15. Verify that repositories are created from AWS console for ECR
![image](https://user-images.githubusercontent.com/77958988/113389125-1b34c300-93ad-11eb-9e0e-ed6298b9cc8a.png)


16.Create ecsTaskExecutionRole with AmazonECSTaskExecutionRolePolicy 

18. Create Task definition for MySQL

17. Create Task definition for Registration

18. Create Task definition for Document

19. Create Task definition for master

20. Create Task definition for Frontend
