
# Introduction
This learning path helps you deploy microservice architecture based UPSTAC application on AWS ECS.
This application uses MySQL database ,frontend is in reactJS and backend microservice's are written in springboot.

![image](https://user-images.githubusercontent.com/77958988/113536855-b9669a00-95f4-11eb-83b1-0577e803544b.png)


References:

Original github link:

https://github.com/upgrad-edu/UPSTAC-Microservices-Frontend

https://github.com/upgrad-edu/UPSTAC-Microservices-Backend/
# Task1
1.Ubuntu 20.04 compute instance is used for this exercise
<img width="932" alt="ubuntuami" src="https://user-images.githubusercontent.com/77958988/113335071-1686f500-9342-11eb-8ca3-2e36b6a3260d.png">

<img width="935" alt="ubuntuinstancetype" src="https://user-images.githubusercontent.com/77958988/113335439-8f864c80-9342-11eb-8863-9b09bb1e8f56.png">


<img width="913" alt="storage" src="https://user-images.githubusercontent.com/77958988/113336199-a37e7e00-9343-11eb-8d49-9caf11e6d46a.png">

# Task2
2.Clone Frontend and backend code of UPSTAC application on ubuntu compute instance.
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
3.Install Maven and docker (https://docs.docker.com/engine/install/ubuntu/) on ubuntu compute instance.

Maven installation:
````
sudo apt update
sudo apt install maven
````
4.Create jar files for the backend microservices using below command.
````
mvn clean package install spring-boot:repackage -Pprod
````
5.Write Dockerfile for frontend and place it in UPSTAC-Microservices-Frontend folder.
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
6.Write Dockerfile's for backend microservice's(document,master & register )and place them in corresponding folders in UPSTAC-Microservices-Backend folder.
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
7.Create docker image for frontend using below command.

````
UPSTAC-Microservices-Frontend$sudo docker build -t vaishalinankani08/upstac-frontend .
````
8.Create docker image's for backend microservices i.e. document , master & registeration using below command.

````
UPSTAC-Microservices-Backend/upstac-api-document$sudo docker build -t vaishalinankani08/upstac-document .
UPSTAC-Microservices-Backend/upstac-api-register$sudo docker build -t vaishalinankani08/upstac-registration .
UPSTAC-Microservices-Backend/upstac-api-master$sudo docker build -t vaishalinankani08/upstac-master .
````
# Task3
9.Install aws cli on ubuntu compute instance
````
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
````
10.Create IAM role with AmazonEC2ContainerRegistryFullAccess  policy and attach it to th ubuntu EC2 instance.This is required so that
 docker images created for UPSTAC application can be pushed to ECR.

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

# Task4

16.Create security group for UPSTAC Application that will allow traffic towards upstac application enabling ports frontend service  at 3000,master service at 8080,
    registration service at 8090,document service at 8082

<img width="935" alt="tg1" src="https://user-images.githubusercontent.com/77958988/113416623-5819ad80-93df-11eb-876c-3c7d920e9d2a.png">

<img width="960" alt="tg2" src="https://user-images.githubusercontent.com/77958988/113416670-7089c800-93df-11eb-8286-b0f9c0378eac.png">

<img width="883" alt="tg3" src="https://user-images.githubusercontent.com/77958988/113416700-7c758a00-93df-11eb-88e5-183eab852774.png">

<img width="933" alt="tg4" src="https://user-images.githubusercontent.com/77958988/113416722-89927900-93df-11eb-8816-9eab82049a9e.png">

17. Create Application load balancer and target group with routing rule for frontend service.

<img width="857" alt="lb" src="https://user-images.githubusercontent.com/77958988/113416769-9ca54900-93df-11eb-9f21-4e285643e5f8.png">

<img width="884" alt="lb2" src="https://user-images.githubusercontent.com/77958988/113416797-a7f87480-93df-11eb-9077-7da5d0ddf497.png">

<img width="872" alt="lb3" src="https://user-images.githubusercontent.com/77958988/113416822-b6469080-93df-11eb-9f7b-0587101d441f.png">

<img width="796" alt="lb4" src="https://user-images.githubusercontent.com/77958988/113416848-c3637f80-93df-11eb-8b60-79a2031b1ff6.png">

<img width="930" alt="lb5" src="https://user-images.githubusercontent.com/77958988/113416871-d37b5f00-93df-11eb-89ef-2ec21f104673.png">

<img width="938" alt="lb6" src="https://user-images.githubusercontent.com/77958988/113416895-e130e480-93df-11eb-9265-cd57a82c7fbb.png">

<img width="941" alt="lb7" src="https://user-images.githubusercontent.com/77958988/113418718-a761dd00-93e3-11eb-89dd-b3db6bb5f598.png">

<img width="943" alt="lb8" src="https://user-images.githubusercontent.com/77958988/113418742-b3e63580-93e3-11eb-88d2-1717f00aa612.png">

<img width="938" alt="lb9" src="https://user-images.githubusercontent.com/77958988/113418780-c1032480-93e3-11eb-8d10-f5e2f4f1feb6.png">

<img width="943" alt="lb10" src="https://user-images.githubusercontent.com/77958988/113418793-cc565000-93e3-11eb-9ea0-b54b36ecebf9.png">

# Task5

18.Create ecsTaskExecutionRole with AmazonECSTaskExecutionRolePolicy .

<img width="867" alt="ecsrole1" src="https://user-images.githubusercontent.com/77958988/113393258-07d92600-93b4-11eb-9d13-f7d243ee066e.png">

<img width="803" alt="ecsrole2" src="https://user-images.githubusercontent.com/77958988/113393305-1a535f80-93b4-11eb-9d85-d20b904edb32.png">

<img width="825" alt="ecsrole3" src="https://user-images.githubusercontent.com/77958988/113393353-2ccd9900-93b4-11eb-8a08-e83f7bddf7cb.png">

<img width="800" alt="ecsrole4" src="https://user-images.githubusercontent.com/77958988/113393406-3fe06900-93b4-11eb-8b09-ffcd173199ac.png">

<img width="917" alt="ecsrole5" src="https://user-images.githubusercontent.com/77958988/113393439-4cfd5800-93b4-11eb-9594-4151cf01f3be.png">

19. Create Task definition for MySQL.set environment variables as mentioned below
MYSQL_ROOT_PASSWORD=password 
MYSQL_DATABASE=upgradpg  
MYSQL_USER=upgradpg  
MYSQL_PASSWORD=upgradpg

<img width="882" alt="mysql1" src="https://user-images.githubusercontent.com/77958988/113406865-d703eb80-93c9-11eb-95a5-b93a83b06fbe.png">


<img width="832" alt="mysql2" src="https://user-images.githubusercontent.com/77958988/113406889-e4b97100-93c9-11eb-910c-3a3b349d2e66.png">


<img width="869" alt="mysql3" src="https://user-images.githubusercontent.com/77958988/113406924-f13dc980-93c9-11eb-9e70-005726683be9.png">


<img width="921" alt="mysql4" src="https://user-images.githubusercontent.com/77958988/113406944-fd298b80-93c9-11eb-827b-e1a9d3ae302d.png">


<img width="927" alt="mysql5" src="https://user-images.githubusercontent.com/77958988/113406965-061a5d00-93ca-11eb-83a3-21472b236e99.png">


<img width="689" alt="mysql6" src="https://user-images.githubusercontent.com/77958988/113406990-13374c00-93ca-11eb-8881-ab2fcd7de6f2.png">


<img width="944" alt="mysql7" src="https://user-images.githubusercontent.com/77958988/113407013-1f230e00-93ca-11eb-85f0-2b255780524c.png">

20. Create Task definition for Registration.set enviornment variables  MASTER_SVC_HOST =master.upgrad.com 
   where upgrad.com is the namespace used for service deployment and master is the name of master service .& MYSQL_HOST=mysql.upgrad.com
   represents mysql service in upgrad.com namespace.

<img width="869" alt="registration1" src="https://user-images.githubusercontent.com/77958988/113408297-bab57e00-93cc-11eb-9786-fa55ad7a079d.png">

<img width="873" alt="registration2" src="https://user-images.githubusercontent.com/77958988/113408331-cb65f400-93cc-11eb-974f-75c12fdcb32a.png">

<img width="930" alt="registration3" src="https://user-images.githubusercontent.com/77958988/113408385-e46ea500-93cc-11eb-8bb7-7fec5b62a1ff.png">

<img width="930" alt="registration4" src="https://user-images.githubusercontent.com/77958988/113408425-f3555780-93cc-11eb-9922-7629c2033e22.png">

<img width="717" alt="registration6" src="https://user-images.githubusercontent.com/77958988/113408449-023c0a00-93cd-11eb-83cf-e25863570359.png">

<img width="717" alt="registration6" src="https://user-images.githubusercontent.com/77958988/113408504-18e26100-93cd-11eb-9c0f-a838f554dc11.png">

<img width="938" alt="registration7" src="https://user-images.githubusercontent.com/77958988/113408536-2b5c9a80-93cd-11eb-8b06-56112e5deca4.png">


19. Create Task definition for Document.set enviornment variables MASTER_SVC_HOST =master.upgrad.com 
   where upgrad.com is the namespace used for service deployment and master is the name of master service & MYSQL_HOST=mysql.upgrad.com
   represents mysql service in upgrad.com namespace.

<img width="893" alt="document1" src="https://user-images.githubusercontent.com/77958988/113409713-842d3280-93cf-11eb-9074-5df80a0c2461.png">

<img width="843" alt="document2" src="https://user-images.githubusercontent.com/77958988/113409744-927b4e80-93cf-11eb-99bf-9f456b5e311f.png">

<img width="934" alt="document3" src="https://user-images.githubusercontent.com/77958988/113409771-a030d400-93cf-11eb-81cf-d653c3f6bebb.png">

<img width="923" alt="document4" src="https://user-images.githubusercontent.com/77958988/113409803-ae7ef000-93cf-11eb-8b7f-959366a4f08c.png">

<img width="770" alt="document5" src="https://user-images.githubusercontent.com/77958988/113409834-bdfe3900-93cf-11eb-9664-6c8e4973db4b.png">

<img width="944" alt="document6" src="https://user-images.githubusercontent.com/77958988/113409881-ceaeaf00-93cf-11eb-8665-851afd47b092.png">


20. Create Task definition for master service .set enviornment variable MYSQL_HOST=mysql.upgrad.com that
   represents mysql service in upgrad.com namespace .
<img width="815" alt="master1" src="https://user-images.githubusercontent.com/77958988/113413335-13d6df00-93d8-11eb-920f-9d73eeb52814.png">

<img width="861" alt="master2" src="https://user-images.githubusercontent.com/77958988/113413351-1fc2a100-93d8-11eb-96d1-65d74ae15644.png">

<img width="937" alt="master3" src="https://user-images.githubusercontent.com/77958988/113413366-2e10bd00-93d8-11eb-85e2-3553fb36ae0a.png">

<img width="925" alt="master4" src="https://user-images.githubusercontent.com/77958988/113413391-3a951580-93d8-11eb-8b99-7d38a2f1617b.png">

<img width="941" alt="master5" src="https://user-images.githubusercontent.com/77958988/113413414-48e33180-93d8-11eb-8a63-380344720f5b.png">


21. Create Task definition for Frontend .Click on  "Configure via JSON" and add "pseudoTerminal": true,this is required to prevent frontend 
    task from exiting .Set below enviornment variables which represent the self hostname and hostnames of backend microservices
   REACT_APP_ROOTURL= frontend.upgrad.com
   REACT_APP_DOCSVC_HOST=document.upgrad.com 
   REACT_APP_REGSVC_HOST=registration.upgrad.com 
   REACT_APP_MASTERSVC_HOST=master.upgrad.com
<img width="682" alt="frontend1" border="2" src="https://user-images.githubusercontent.com/77958988/113413455-60221f00-93d8-11eb-9b8d-7efda714c23a.png">

<img width="835" alt="frontend2" border="2" src="https://user-images.githubusercontent.com/77958988/113413481-6f08d180-93d8-11eb-82fe-2592e9e45404.png">

<img width="938" alt="frontend3" border="2" src="https://user-images.githubusercontent.com/77958988/113413500-7af49380-93d8-11eb-8617-caddecf29181.png">

<img width="854" alt="frontend4" border="2" src="https://user-images.githubusercontent.com/77958988/113413529-88aa1900-93d8-11eb-9ff1-856634fc3535.png">

<img width="936" alt="frontend5" src="https://user-images.githubusercontent.com/77958988/113503514-09d7ec00-9550-11eb-999d-1fdaab603f62.png">

<img width="792" alt="frontend6" border="2" src="https://user-images.githubusercontent.com/77958988/113413608-ac6d5f00-93d8-11eb-8568-5e9f1417c3f9.png">

<img width="944" alt="frontend7" border="2" src="https://user-images.githubusercontent.com/77958988/113413628-babb7b00-93d8-11eb-990b-e3a7a7174e13.png">




24. Create cluster for launching microservices of UPSTAC Application in farget mode.

<img width="1028" alt="cluster1" src="https://user-images.githubusercontent.com/77958988/113430592-121d1380-93f8-11eb-842a-fa459558da6b.png">
<img width="697" alt="cluster2" src="https://user-images.githubusercontent.com/77958988/113430648-23feb680-93f8-11eb-88ae-fc108973869d.png">
<img width="766" alt="cluster3" src="https://user-images.githubusercontent.com/77958988/113430678-2fea7880-93f8-11eb-859a-26c9c0121d07.png">
<img width="933" alt="cluster4" src="https://user-images.githubusercontent.com/77958988/113430708-3aa50d80-93f8-11eb-8727-4dfb876cad4f.png">

25. Create mysql service with a single task with service discovery enabled,with appropriate namespace(upgrad.com).
<img width="778" alt="mysqlsvc1" src="https://user-images.githubusercontent.com/77958988/113434594-dfc2e480-93fe-11eb-80af-1d1209a56bc6.png">

<img width="754" alt="mysqlsvc2" src="https://user-images.githubusercontent.com/77958988/113434620-ed786a00-93fe-11eb-919b-79aa16b01f9a.png">

<img width="1354" alt="mysqlsvc3" src="https://user-images.githubusercontent.com/77958988/113434666-06811b00-93ff-11eb-92bf-fe17bd1257c1.png">

<img width="1354" alt="mysqlsvc4" src="https://user-images.githubusercontent.com/77958988/113434782-36302300-93ff-11eb-88b0-a652124062e3.png">

<img width="801" alt="mysqlsvc5" src="https://user-images.githubusercontent.com/77958988/113434826-47792f80-93ff-11eb-825a-58ae3bd8b9c6.png">

<img width="733" alt="mysqlsvc6" src="https://user-images.githubusercontent.com/77958988/113434847-54961e80-93ff-11eb-9771-9e131abfd499.png">

<img width="794" alt="mysqlsvc7" src="https://user-images.githubusercontent.com/77958988/113434900-6bd50c00-93ff-11eb-94b5-4be06cc3780b.png">

<img width="806" alt="mysqlsvc8" src="https://user-images.githubusercontent.com/77958988/113434955-8313f980-93ff-11eb-92f1-305ab162e547.png">

<img width="921" alt="mysqlsvc9" src="https://user-images.githubusercontent.com/77958988/113434985-93c46f80-93ff-11eb-977c-461f46749607.png">

26. Create document service with a single task with service discovery enabled,with appropriate namespace .

<img width="749" alt="docsvc1" src="https://user-images.githubusercontent.com/77958988/113470960-d6755e80-9476-11eb-96b1-6af515655d52.png">
<img width="726" alt="docsvc2" src="https://user-images.githubusercontent.com/77958988/113470966-e42ae400-9476-11eb-82d6-0bbdbbe9e5cd.png">
<img width="1354" alt="docsvc3" src="https://user-images.githubusercontent.com/77958988/113470973-f016a600-9476-11eb-8493-1410b9bb6533.png">
<img width="852" alt="docsvc4" src="https://user-images.githubusercontent.com/77958988/113470979-fc026800-9476-11eb-9f43-2b39ae9a9de9.png">
<img width="842" alt="docsvc5" src="https://user-images.githubusercontent.com/77958988/113470985-0886c080-9477-11eb-90da-73c6122cb648.png">
<img width="810" alt="docsvc6" src="https://user-images.githubusercontent.com/77958988/113470995-15a3af80-9477-11eb-89a4-429a5a8221fd.png">
<img width="752" alt="docsvc7" src="https://user-images.githubusercontent.com/77958988/113471003-20f6db00-9477-11eb-8c5a-1047f4751ff1.png">
<img width="734" alt="docsvc8" src="https://user-images.githubusercontent.com/77958988/113471013-2e13ca00-9477-11eb-8de9-17129f6d1f75.png">
<img width="922" alt="docsvc9" src="https://user-images.githubusercontent.com/77958988/113471026-408e0380-9477-11eb-9a1f-a97b9d82335d.png">

27. Create ecsAutoscaleRole with AutoScalingFullAccess & AmazonEC2ContainerServiceAutoscaleRole
<img width="897" alt="scale1" src="https://user-images.githubusercontent.com/77958988/114198415-ea3a2c80-9970-11eb-98e5-a919626195de.png">

<img width="826" alt="scale2" src="https://user-images.githubusercontent.com/77958988/113405329-49270100-93c7-11eb-9c7d-3c7f4c0f2678.png">

<img width="869" alt="scale3" src="https://user-images.githubusercontent.com/77958988/113405364-5b08a400-93c7-11eb-831f-6bed45a19f81.png">

<img width="938" alt="scale4" src="https://user-images.githubusercontent.com/77958988/113405400-6bb91a00-93c7-11eb-86df-de6df9306c98.png">

<img width="951" alt="scale5" src="https://user-images.githubusercontent.com/77958988/113405455-825f7100-93c7-11eb-9c69-7939d724ecbb.png">

<img width="918" alt="scale6" src="https://user-images.githubusercontent.com/77958988/113405518-96a36e00-93c7-11eb-82ff-164478be5956.png">

<img width="932" alt="scale7" src="https://user-images.githubusercontent.com/77958988/113405551-a753e400-93c7-11eb-80c7-fbe1e00fe6cc.png">

28.Create autoscaleapp policy and attach it to both user and IAM role that will be used to configure autoscaling for registration service.

<img width="710" alt="auto1" src="https://user-images.githubusercontent.com/77958988/113471335-4edd1f00-9479-11eb-9f19-da3733995866.png">
<img width="684" alt="auto2" src="https://user-images.githubusercontent.com/77958988/113471343-5c92a480-9479-11eb-93fe-9f0b232b761f.png">
<img width="767" alt="auto3" src="https://user-images.githubusercontent.com/77958988/113471354-674d3980-9479-11eb-9cd3-a77548ee9feb.png">
<img width="906" alt="auto4" src="https://user-images.githubusercontent.com/77958988/113471365-72a06500-9479-11eb-918a-5f1bd5a116e4.png">
<img width="935" alt="auto5" src="https://user-images.githubusercontent.com/77958988/113471373-7d5afa00-9479-11eb-9353-f810e7854736.png">

29.Create registration service with a single task with service discovery enabled,with appropriate namespace .Also enable autoscaling 
   with desired task=1 and maximum task=2.configure autoscaling at targetcpuutilization by registration service > 5%.
<img width="764" alt="regsvc1" src="https://user-images.githubusercontent.com/77958988/113471065-7d59fa80-9477-11eb-94f0-f3a78e29a446.png">
<img width="779" alt="regsvc2" src="https://user-images.githubusercontent.com/77958988/113471072-8e0a7080-9477-11eb-97ad-ecf096d13b18.png">
<img width="827" alt="regsvc3" src="https://user-images.githubusercontent.com/77958988/113471086-a8dce500-9477-11eb-9f15-2c5d970f63f1.png">
<img width="808" alt="regsvc4" src="https://user-images.githubusercontent.com/77958988/113471092-b5f9d400-9477-11eb-8efc-480504509b2b.png">
<img width="808" alt="regsvc5" src="https://user-images.githubusercontent.com/77958988/113471101-c1e59600-9477-11eb-9169-ba09c266cb3e.png">
<img width="846" alt="regsvc6" src="https://user-images.githubusercontent.com/77958988/113471108-d0cc4880-9477-11eb-8bba-8aeb992983d4.png">
<img width="874" alt="regsvc7" src="https://user-images.githubusercontent.com/77958988/113471128-eb9ebd00-9477-11eb-9c3e-89e1da014336.png">
<img width="787" alt="regsvc8" src="https://user-images.githubusercontent.com/77958988/113471136-f9544280-9477-11eb-813f-305213102ab0.png">
<img width="876" alt="regsvc9" src="https://user-images.githubusercontent.com/77958988/113471167-328cb280-9478-11eb-812a-2ada44b14e5a.png">
<img width="787" alt="regsvc10" src="https://user-images.githubusercontent.com/77958988/113471170-3fa9a180-9478-11eb-8d2c-8f8f9c71b6b9.png">

30.Create master service with a single task with service discovery enabled,with appropriate namespace.
 
<img width="727" alt="mastersvc" src="https://user-images.githubusercontent.com/77958988/113474976-a63ab980-9490-11eb-9674-bedf01424cbf.png">
<img width="747" alt="mastersvc2" src="https://user-images.githubusercontent.com/77958988/113475138-632d1600-9491-11eb-99e8-1e83226c384f.png">
<img width="932" alt="mastersvc3" src="https://user-images.githubusercontent.com/77958988/113475170-97a0d200-9491-11eb-91f6-667017957802.png">
<img width="738" alt="mastersvc4" src="https://user-images.githubusercontent.com/77958988/113475193-b2734680-9491-11eb-9b6e-22b072684f60.png">
<img width="748" alt="mastersvc5" src="https://user-images.githubusercontent.com/77958988/113475017-de41fc80-9490-11eb-86b7-74ff2c9e4596.png">
<img width="740" alt="mastersvc6" src="https://user-images.githubusercontent.com/77958988/113475033-ea2dbe80-9490-11eb-8984-54d117f2421f.png">
<img width="749" alt="mastersvc7" src="https://user-images.githubusercontent.com/77958988/113475044-f6198080-9490-11eb-8bce-3520503b5e25.png">
<img width="936" alt="mastersvc8" src="https://user-images.githubusercontent.com/77958988/113475049-00d41580-9491-11eb-8e05-68ea84b3f7b8.png">

31.Create frontend service with a single task with service discovery enabled,with appropriate namespace.
<img width="750" alt="frontendsvc1" src="https://user-images.githubusercontent.com/77958988/113477758-c8890300-94a1-11eb-940e-ba7e4047e51d.png">
<img width="932" alt="frontendsvc2" src="https://user-images.githubusercontent.com/77958988/113477768-e22a4a80-94a1-11eb-9966-f23d06ce9feb.png">
<img width="716" alt="frontendsvc3" src="https://user-images.githubusercontent.com/77958988/113477775-ed7d7600-94a1-11eb-879b-ee024a2a6f69.png">
<img width="886" alt="frontendsvc4" src="https://user-images.githubusercontent.com/77958988/113477782-f8380b00-94a1-11eb-9fcd-cb5a052b76ec.png">
<img width="710" alt="frontendsvc5" src="https://user-images.githubusercontent.com/77958988/113477789-0c7c0800-94a2-11eb-9c99-6819cac4f84e.png">
<img width="767" alt="frontendsvc6" src="https://user-images.githubusercontent.com/77958988/113477796-1aca2400-94a2-11eb-9ce0-f69ce2cc66a8.png">
<img width="618" alt="frontendsvc7" src="https://user-images.githubusercontent.com/77958988/113477803-2584b900-94a2-11eb-9de9-796f921a7b1f.png">
<img width="856" alt="frontendsvc8" src="https://user-images.githubusercontent.com/77958988/113477804-2fa6b780-94a2-11eb-81b4-898c7ba0c417.png">
<img width="936" alt="frontendsvc9" src="https://user-images.githubusercontent.com/77958988/113477811-39c8b600-94a2-11eb-8822-60b18387fda3.png">


32.  Use dig command to verify DNS resolution.
<img width="935" alt="ecssvc" src="https://user-images.githubusercontent.com/77958988/113478150-a5ac1e00-94a4-11eb-9829-ed474484be62.png">

````
ubuntu@ip-172-31-47-81:~$ dig registration.upgrad.com

; <<>> DiG 9.16.1-Ubuntu <<>> registration.upgrad.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 9741
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;registration.upgrad.com.       IN      A

;; ANSWER SECTION:
registration.upgrad.com. 60     IN      A       172.31.90.219

;; Query time: 4 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Sat Apr 03 12:22:39 UTC 2021
;; MSG SIZE  rcvd: 68

````
# Task6
33.Create alarm for "upstac_regsvc_cpu_utilization_above_5percentage" in cloudwatch with cpu utilization threshold as 5 % and configure email address to triger email notification when CPU Utilization exceeds 5 % for clustername=upstac and servicename=registration.
<img width="936" alt="alarm2" src="https://user-images.githubusercontent.com/77958988/113479392-74375080-94ac-11eb-965d-45f395471834.png">
<img width="776" alt="alarm3" src="https://user-images.githubusercontent.com/77958988/113479404-844f3000-94ac-11eb-9560-a0d248fe8198.png">
<img width="857" alt="alarm4" src="https://user-images.githubusercontent.com/77958988/113479409-90d38880-94ac-11eb-85d1-d921e12b1380.png">
<img width="836" alt="alarm5" src="https://user-images.githubusercontent.com/77958988/113479411-9c26b400-94ac-11eb-9cca-5d4eaa1e7802.png">
<img width="853" alt="alarm6" src="https://user-images.githubusercontent.com/77958988/113479415-a5b01c00-94ac-11eb-977f-f99b22c80015.png">
<img width="774" alt="alarm7" src="https://user-images.githubusercontent.com/77958988/113479422-aea0ed80-94ac-11eb-9b07-595b3bb1bf51.png">
<img width="755" alt="alarm8" src="https://user-images.githubusercontent.com/77958988/113479437-beb8cd00-94ac-11eb-80fb-cb3828b1af99.png">
<img width="674" alt="alarm9" src="https://user-images.githubusercontent.com/77958988/113479444-c8423500-94ac-11eb-84f7-3018e5d32b3d.png">
<img width="688" alt="alarm10" src="https://user-images.githubusercontent.com/77958988/113479472-f162c580-94ac-11eb-9f34-511739c1fe03.png">
<img width="675" alt="alarm11" src="https://user-images.githubusercontent.com/77958988/113479480-ffb0e180-94ac-11eb-9ef1-961ae7183eea.png">
<img width="575" alt="alarm12" src="https://user-images.githubusercontent.com/77958988/113479723-46530b80-94ae-11eb-9152-43fe2820bc4d.png">
<img width="934" alt="alarm13" src="https://user-images.githubusercontent.com/77958988/113479560-6f26d100-94ad-11eb-9671-8c1e62c43cd3.png">

34.Install apache benchmark on ubuntu EC2 instance and pump traffic towards registration service using register  API with json payload
     [register.json](https://github.com/vaishalinankani08/upstac-solution/blob/main/register.json) so as 
     to reach the target threshold for CPU Utilization and trigger autoscaling as well as email notification.
````
ubuntu@ip-172-31-47-81:~$sudo apt install apache2-utils
ubuntu@ip-172-31-47-81:~$ ab -p register.json -T application/json -c 100 -n 50000 http://registration.upgrad.com:8090/auth/doctor/register
This is ApacheBench, Version 2.3 <$Revision: 1843412 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking registration.upgrad.com (be patient)


Completed 5000 requests
Completed 10000 requests
Completed 15000 requests
Completed 20000 requests
Completed 25000 requests
Completed 30000 requests
Completed 35000 requests
Completed 40000 requests
Completed 45000 requests
Completed 50000 requests
Finished 50000 requests


Server Software:
Server Hostname:        registration.upgrad.com
Server Port:            8090

Document Path:          /auth/doctor/register
Document Length:        224 bytes

Concurrency Level:      100
Time taken for tests:   353.687 seconds
Complete requests:      50000
Failed requests:        49999
   (Connect: 0, Receive: 0, Length: 49999, Exceptions: 0)
Non-2xx responses:      49999
Total transferred:      24900079 bytes
Total body sent:        22000000
HTML transferred:       7250079 bytes
Requests per second:    141.37 [#/sec] (mean)
Time per request:       707.375 [ms] (mean)
Time per request:       7.074 [ms] (mean, across all concurrent requests)
Transfer rate:          68.75 [Kbytes/sec] received
                        60.74 kb/s sent
                        129.50 kb/s total

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        1    1   0.7      1      22
Processing:    14  704 334.4    678    3630
Waiting:       13  702 332.6    677    3630
Total:         14  705 334.4    679    3631

Percentage of the requests served within a certain time (ms)
  50%    679
  66%    808
  75%    904
  80%    977
  90%   1144
  95%   1300
  98%   1497
  99%   1623
 100%   3631 (longest request)
ubuntu@ip-172-31-47-81:~$

````
35.Observe that alarm condition is reached for "upstac_regsvc_cpu_utilization_above_5percentage" alarm as well as autoscaling.registration microservice scales up and a email notification is received at email id configured in alarm
<img width="938" alt="alarm14" src="https://user-images.githubusercontent.com/77958988/113479578-8239a100-94ad-11eb-86be-832d9b30cbf7.png">
<img width="920" alt="alarm15" src="https://user-images.githubusercontent.com/77958988/113479587-8cf43600-94ad-11eb-8fea-bf8fc3ce966e.png">
<img width="828" alt="alarm16" src="https://user-images.githubusercontent.com/77958988/113479602-a2696000-94ad-11eb-8c9e-09a73136d3e9.png">

Email Notification:
![alarm17](https://user-images.githubusercontent.com/77958988/113501300-a5ae2b80-9541-11eb-875c-279be0244011.png)
<img width="812" alt="alarm18" src="https://user-images.githubusercontent.com/77958988/113482115-be273300-94ba-11eb-90b5-414f8793f49f.png">
