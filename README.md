# DOCKER PROJECT – Jenkins + Docker Swarm Deployment 🚀

This project demonstrates an **end-to-end DevOps workflow** using **Docker**, **Docker Swarm**, **Docker Compose**, and **Jenkins** to build Docker images, push them to DockerHub, and deploy multiple containers across multiple servers.

---

## 📌 Key Concepts Used

- **Docker** – Containerization platform
- **Dockerfile** – To create Docker images by automation
- **Docker Compose** – To create multiple containers on a single server
- **Docker Swarm** – To create multiple containers on multiple servers
- **Docker Stack** – Combination of Docker Swarm + Docker Compose
- **Jenkins** – CI/CD automation tool

---

## 📂 Project Structure


---

## 1️⃣ Install Docker on All Servers

Create **3 servers** and install Docker on **all of them**.

```bash
yum install docker -y
systemctl start docker
systemctl status docker
```
2️⃣ Create Docker Swarm Cluster

Run the following commands on the Manager Node:


```bash
docker swarm init 
docker node ls

```
3️⃣ Dockerfile – Create Custom Docker Image

📄 Use file name: Dockerfile

```bash
FROM ubuntu
RUN apt-get update -y
RUN apt-get install apache2 -y
COPY index.html /var/www/html/
CMD ["/usr/sbin/apachectl", "-D", "FOREGROUND"]
```
4️⃣ index.html – Application File

📄 Use file name: index.html

Static HTML file

Served by Apache inside Docker container

Represents application UI

5️⃣ Install Jenkins (Master Node)

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
amazon-linux-extras install java-openjdk11 -y
yum install git maven jenkins -y
systemctl start jenkins.service
systemctl status jenkins.service
```

6️⃣ Jenkins Pipeline – Build, Tag & Push Image

```bash
pipeline {
    agent any
    
    stages {
       stage('checkout'){
           steps{
               git branch: 'main', url: 'https://github.com/kshitij0798/DockerProject.git'
           }
       }
         stage('build'){
           steps{
               sh 'docker build -t $img .'
           }
       }
         stage('tag'){
           steps{
               sh 'docker tag $img $repo'
           }
       }
          stage('push'){
           steps{
               sh 'docker login -u kshitij0798  -p $pass'
               sh 'docker push $repo'
           }
       }
           stage('deploy'){
           steps{
               sh 'docker stack deploy -c docker-compose.yml paytm'
               
           }
       }
         
    }
}

```

Jenkins Pipeline Flow

Checkout code from GitHub

Build Docker image

Tag Docker image

Push image to DockerHub

7️⃣ Give Docker Permissions to Jenkins

```bash
chmod 777 /var/run/docker.sock
systemctl daemon-reload
systemctl restart docker.service

```

8️⃣ Docker Compose File for Swarm Deployment

```bash
version: '3.8'
services:
  movies:
    image: kshitij0798/paymovies:latest
    ports:
      - "81:80"
    deploy:
      replicas: 3
  train:
    image: kshitij0798/paytrain:latest
    ports:
      - "82:80"
    deploy:
      replicas: 3
  dth:
    image: kshitij0798/paydth:latest
    ports:
      - "83:80"
    deploy:
      replicas: 3
  recharge:
    image: kshitij0798/payrecharge:latest
    ports:
      - "84:80"
    deploy:
      replicas: 3


```
9️⃣ Deploy Docker Stack on Swarm Manager

```bash
docker stack deploy -c docker-compose.yml paytm

```

Verify deployment:

```bash
docker service ls
docker stack ps paytm

```
🌐 Application Access

```bash
http://<manager-ip>:81 → Movies
http://<manager-ip>:82 → Train
http://<manager-ip>:83 → DTH
http://<manager-ip>:84 → Recharge

```

🧹 Cleanup Commands


```bash

docker stack rm paytm
docker service rm $(docker service ls -q)
docker rm -f $(docker ps -aq)
docker rmi -f $(docker images -q)
docker system prune -a --volumes -f

```

🎯 Interview Key Takeaways

Docker Swarm provides clustering and load balancing

Jenkins automates CI/CD pipelines

Docker Compose v3 supports Swarm mode

Docker Stack combines Swarm + Compose

Apache runs in foreground to keep container alive

