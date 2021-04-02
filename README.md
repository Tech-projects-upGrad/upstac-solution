
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

<img width="867" alt="ecsrole1" src="https://user-images.githubusercontent.com/77958988/113393258-07d92600-93b4-11eb-9d13-f7d243ee066e.png">

<img width="803" alt="ecsrole2" src="https://user-images.githubusercontent.com/77958988/113393305-1a535f80-93b4-11eb-9d85-d20b904edb32.png">

<img width="825" alt="ecsrole3" src="https://user-images.githubusercontent.com/77958988/113393353-2ccd9900-93b4-11eb-8a08-e83f7bddf7cb.png">

<img width="800" alt="ecsrole4" src="https://user-images.githubusercontent.com/77958988/113393406-3fe06900-93b4-11eb-8b09-ffcd173199ac.png">

<img width="917" alt="ecsrole5" src="https://user-images.githubusercontent.com/77958988/113393439-4cfd5800-93b4-11eb-9594-4151cf01f3be.png">


17. Create ecsAutoscaleRole with AutoScalingFullAccess & AmazonEC2ContainerServiceAutoscaleRole
<img width="960" alt="scale1" src="https://user-images.githubusercontent.com/77958988/113405264-2f85b980-93c7-11eb-8dea-d0b1f78d09dc.png">

<img width="826" alt="scale2" src="https://user-images.githubusercontent.com/77958988/113405329-49270100-93c7-11eb-9c7d-3c7f4c0f2678.png">

<img width="869" alt="scale3" src="https://user-images.githubusercontent.com/77958988/113405364-5b08a400-93c7-11eb-831f-6bed45a19f81.png">

<img width="938" alt="scale4" src="https://user-images.githubusercontent.com/77958988/113405400-6bb91a00-93c7-11eb-86df-de6df9306c98.png">

<img width="951" alt="scale5" src="https://user-images.githubusercontent.com/77958988/113405455-825f7100-93c7-11eb-9c69-7939d724ecbb.png">

<img width="918" alt="scale6" src="https://user-images.githubusercontent.com/77958988/113405518-96a36e00-93c7-11eb-82ff-164478be5956.png">

<img width="932" alt="scale7" src="https://user-images.githubusercontent.com/77958988/113405551-a753e400-93c7-11eb-80c7-fbe1e00fe6cc.png">

18. Create Task definition for MySQL

<img width="882" alt="mysql1" src="https://user-images.githubusercontent.com/77958988/113406865-d703eb80-93c9-11eb-95a5-b93a83b06fbe.png">


<img width="832" alt="mysql2" src="https://user-images.githubusercontent.com/77958988/113406889-e4b97100-93c9-11eb-910c-3a3b349d2e66.png">


<img width="869" alt="mysql3" src="https://user-images.githubusercontent.com/77958988/113406924-f13dc980-93c9-11eb-9e70-005726683be9.png">


<img width="921" alt="mysql4" src="https://user-images.githubusercontent.com/77958988/113406944-fd298b80-93c9-11eb-827b-e1a9d3ae302d.png">


<img width="927" alt="mysql5" src="https://user-images.githubusercontent.com/77958988/113406965-061a5d00-93ca-11eb-83a3-21472b236e99.png">


<img width="689" alt="mysql6" src="https://user-images.githubusercontent.com/77958988/113406990-13374c00-93ca-11eb-8881-ab2fcd7de6f2.png">


<img width="944" alt="mysql7" src="https://user-images.githubusercontent.com/77958988/113407013-1f230e00-93ca-11eb-85f0-2b255780524c.png">


18. Create Task definition for Registration


<img width="869" alt="registration1" src="https://user-images.githubusercontent.com/77958988/113408297-bab57e00-93cc-11eb-9786-fa55ad7a079d.png">

<img width="873" alt="registration2" src="https://user-images.githubusercontent.com/77958988/113408331-cb65f400-93cc-11eb-974f-75c12fdcb32a.png">

<img width="930" alt="registration3" src="https://user-images.githubusercontent.com/77958988/113408385-e46ea500-93cc-11eb-8bb7-7fec5b62a1ff.png">

<img width="930" alt="registration4" src="https://user-images.githubusercontent.com/77958988/113408425-f3555780-93cc-11eb-9922-7629c2033e22.png">

<img width="717" alt="registration6" src="https://user-images.githubusercontent.com/77958988/113408449-023c0a00-93cd-11eb-83cf-e25863570359.png">

<img width="717" alt="registration6" src="https://user-images.githubusercontent.com/77958988/113408504-18e26100-93cd-11eb-9c0f-a838f554dc11.png">

<img width="938" alt="registration7" src="https://user-images.githubusercontent.com/77958988/113408536-2b5c9a80-93cd-11eb-8b06-56112e5deca4.png">


19. Create Task definition for Document

<img width="893" alt="document1" src="https://user-images.githubusercontent.com/77958988/113409713-842d3280-93cf-11eb-9074-5df80a0c2461.png">

<img width="843" alt="document2" src="https://user-images.githubusercontent.com/77958988/113409744-927b4e80-93cf-11eb-99bf-9f456b5e311f.png">

<img width="934" alt="document3" src="https://user-images.githubusercontent.com/77958988/113409771-a030d400-93cf-11eb-81cf-d653c3f6bebb.png">

<img width="923" alt="document4" src="https://user-images.githubusercontent.com/77958988/113409803-ae7ef000-93cf-11eb-8b7f-959366a4f08c.png">

<img width="770" alt="document5" src="https://user-images.githubusercontent.com/77958988/113409834-bdfe3900-93cf-11eb-9664-6c8e4973db4b.png">

<img width="944" alt="document6" src="https://user-images.githubusercontent.com/77958988/113409881-ceaeaf00-93cf-11eb-8665-851afd47b092.png">


20. Create Task definition for master
<img width="815" alt="master1" src="https://user-images.githubusercontent.com/77958988/113413335-13d6df00-93d8-11eb-920f-9d73eeb52814.png">

<img width="861" alt="master2" src="https://user-images.githubusercontent.com/77958988/113413351-1fc2a100-93d8-11eb-96d1-65d74ae15644.png">

<img width="937" alt="master3" src="https://user-images.githubusercontent.com/77958988/113413366-2e10bd00-93d8-11eb-85e2-3553fb36ae0a.png">

<img width="925" alt="master4" src="https://user-images.githubusercontent.com/77958988/113413391-3a951580-93d8-11eb-8b99-7d38a2f1617b.png">

<img width="941" alt="master5" src="https://user-images.githubusercontent.com/77958988/113413414-48e33180-93d8-11eb-8a63-380344720f5b.png">


20. Create Task definition for Frontend

<img width="682" alt="frontend1" border="2" src="https://user-images.githubusercontent.com/77958988/113413455-60221f00-93d8-11eb-9b8d-7efda714c23a.png">

<img width="835" alt="frontend2" border="2" src="https://user-images.githubusercontent.com/77958988/113413481-6f08d180-93d8-11eb-82fe-2592e9e45404.png">

<img width="938" alt="frontend3" border="2" src="https://user-images.githubusercontent.com/77958988/113413500-7af49380-93d8-11eb-8617-caddecf29181.png">

<img width="854" alt="frontend4" border="2" src="https://user-images.githubusercontent.com/77958988/113413529-88aa1900-93d8-11eb-9ff1-856634fc3535.png">

<img width="792" alt="frontend6" border="2" src="https://user-images.githubusercontent.com/77958988/113413567-99f32580-93d8-11eb-9543-da052888a2e0.png">

<img width="792" alt="frontend6" border="2" src="https://user-images.githubusercontent.com/77958988/113413608-ac6d5f00-93d8-11eb-8568-5e9f1417c3f9.png">

<img width="944" alt="frontend7" border="2" src="https://user-images.githubusercontent.com/77958988/113413628-babb7b00-93d8-11eb-990b-e3a7a7174e13.png">

21.Create security group for UPSTAC Application

<img width="935" alt="tg1" src="https://user-images.githubusercontent.com/77958988/113416623-5819ad80-93df-11eb-876c-3c7d920e9d2a.png">

<img width="960" alt="tg2" src="https://user-images.githubusercontent.com/77958988/113416670-7089c800-93df-11eb-8286-b0f9c0378eac.png">

<img width="883" alt="tg3" src="https://user-images.githubusercontent.com/77958988/113416700-7c758a00-93df-11eb-88e5-183eab852774.png">

<img width="933" alt="tg4" src="https://user-images.githubusercontent.com/77958988/113416722-89927900-93df-11eb-8816-9eab82049a9e.png">




22. Create Application load balancer and target group with routing rule for frontend service
