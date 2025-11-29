---

# MEAN Stack Application Deployment Using Jenkins and Docker Compose

## System Architecture

   ![Image](https://github.com/user-attachments/assets/2bac77db-43e2-4814-93a1-52ed507e139e) 

## The system architecture consists of:

Angular frontend served via Nginx

Node.js + Express backend

MongoDB database running in container

Nginx reverse proxy routing all traffic through port 80

Jenkins automatically building and deploying using Docker images

Docker Hub storing production-ready container images

Ubuntu instance running application stack



---

## 📁 Project Structure

    frontend/        → Angular UI    → Dockerfile 
    backend/         → Node.js API   → Dockerfile 
    docker-compose.yml 
    Jenkinsfile 


---

## 🧩 Components Used

| Component     | Technology                |
| ------------- | ------------------------- |
| Frontend      | Angular                   |
| Backend       | Node.js + Express         |
| Database      | MongoDB                   |
| Deployment    | Docker + Docker Compose   |
| CI/CD         | Jenkins                   |
| Reverse Proxy | Nginx                     |
| Hosting       | Ubuntu instance (AWS)     |
| Image storage | Docker Hub                |
| Source Code   | GitHub                    |


---


## Project steps:

## 1️⃣ EC2 instace  Setup

### Create an Ubuntu VM and SSH into it: <br>
    ssh ubuntu@<server-ip>
    
<img width="1920" height="1020" alt="Image" src="https://github.com/user-attachments/assets/c309311e-58e8-4edf-9255-0b213db35c5e" />

### Install Docker and Docker compose:

    sudo apt update 
    sudo apt install -y docker.io docker-compose 
    sudo systemctl enable docker 


---


## 2️⃣ Clone the Repository

    git clone https://github.com/Faizan64/mean-crud-task.git <br>
    cd mean-crud-task


---


## 3️⃣ Dockerization

### Backend Dockerfile:

    FROM node:18
    WORKDIR /app
    COPY package*.json ./
    RUN npm install
    COPY . .
    EXPOSE 8080
    CMD ["node", "server.js"]

### Frontend Dockerfile: 

    FROM node:18 AS build
    WORKDIR /app
    COPY package*.json ./
    RUN npm install
    COPY . .
    RUN npm run build
    
    FROM nginx:alpine
    COPY --from=build /app/dist/angular-15-crud /usr/share/nginx/html
    EXPOSE 80
    CMD ["nginx", "-g", "daemon off;"]


---


## 4️⃣ Build & Push Docker Images

    docker build -t faizann16/mean-backend ./backend 
    docker build -t faizann16/mean-frontend ./frontend 
    
    docker push faizann16/mean-backend 
    docker push faizann16/mean-frontend 

### Docker images:

<img width="1920" height="1020" alt="Image" src="https://github.com/user-attachments/assets/22cb9d87-36ad-4482-a4f3-0fbdd514dfd6" />


---


## 5️⃣ Deployment With Docker Compose

### Create docker-compose.yml:

    version: '3'
    services:
      mongodb:
        image: mongo
        container_name: mongo
        restart: always
        ports:
          - "27017:27017"
        networks:
          - mean-network

      backend:
        image: faizann16/mean-backend
        container_name: backend
        restart: always
        depends_on:
          - mongodb
        environment:
          - DB_URL=mongodb://mongo:27017/tutorialsDB
        ports:
          - "8080:8080"
        networks:
          - mean-network

      frontend:
        image: faizann16/mean-frontend
        container_name: frontend
        restart: always
        depends_on:
          - backend
        ports:
          - "4200:80"
        networks:
          - mean-network
    
    networks:
      mean-network:
        driver: bridge

### Run deployment:

    sudo docker-compose up -d
    docker ps

### Build containers:

<img width="1920" height="1020" alt="Image" src="https://github.com/user-attachments/assets/c95eaae6-e5cc-4336-a115-8a78e0eb6843" />
    

---


## 6️⃣ Nginx Reverse Proxy Configuration

### Install Nginx:

    sudo apt install nginx -y

### Edit config:

    sudo nano /etc/nginx/sites-available/default

Paste:

    server {
        listen 80;

        location / {
        proxy_pass http://localhost:4200;
        }

        location /api/tutorials/ {
        proxy_pass http://localhost:8080/;
        }
    }


Restart:

    sudo systemctl restart nginx

### App now runs at:

    http://<server-ip>


---


## 7️⃣ Jenkins CI/CD Setup

### Install Jenkins:

    sudo apt install openjdk-17-jdk -y
    wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
    sudo sh -c 'echo deb https://pkg.jenkins.io/debian binary/ >       /etc/apt/sources.list.d/jenkins.list'
    sudo apt update
    sudo apt install jenkins -y
    sudo systemctl start jenkins

Since I am running my backend 8080 so I have changed the jenkins default port i.e 8080 to 9090


---


### Jenkins Credentials Required

| Credential       | ID         | Purpose          |
| ---------------- | ---------- | ---------------- |
| Docker Hub login | dockerhub  | push images      |
| SSH private key  | server-key | deploy to server |


---


### Jenkinsfile (Pipeline Automation)

    pipeline {
        agent any
    
        environment {
            DOCKERHUB_USER = 'faizann16'
            BACKEND_IMAGE = "${DOCKERHUB_USER}/mean-backend"
            FRONTEND_IMAGE = "${DOCKERHUB_USER}/mean-frontend"
        }
    
        stages {
            stage('Checkout') {
                steps {
                    checkout scm
                }
            }
    
            stage('Build Docker Images') {
                steps {
                    script {
                        sh """
                          docker build -t ${BACKEND_IMAGE}:${BUILD_NUMBER} -t ${BACKEND_IMAGE}:latest backend
                          docker build -t ${FRONTEND_IMAGE}:${BUILD_NUMBER} -t ${FRONTEND_IMAGE}:latest frontend
                        """
                    }
                }
            }
    
            stage('Push Docker Images') {
                steps {
                    script {
                        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                         usernameVariable: 'DOCKERHUB_USERNAME',
                                                         passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                            sh """
                              echo "$DOCKERHUB_PASSWORD" | docker login -u "$DOCKERHUB_USERNAME" --password-stdin
                              docker push ${BACKEND_IMAGE}:${BUILD_NUMBER}
                              docker push ${BACKEND_IMAGE}:latest
                              docker push ${FRONTEND_IMAGE}:${BUILD_NUMBER}
                              docker push ${FRONTEND_IMAGE}:latest
                              docker logout
                            """
                        }
                    }
                }
            }
    
    	stage('Clean Docker Containers') {
    	    steps {
            	sh """
              	  docker rm -f \$(docker ps -aq) || true
              	  docker network prune -f || true
              	  docker volume prune -f || true
            	"""
        	    }
    	}
    
    
            stage('Deploy to Docker') {
                steps {
                    script {
                        sh """
                          docker-compose pull
                          docker-compose up -d
                        """
                    }
                }
            }
        }
    
        post {
            success {
                echo '🚀 Deployment successful!'
            }
            failure {
                echo '❗ Build or deployment failed!'
            }
        }
    }


---


## ✔ Final Application

### Accessible in browser:

    http://ec2-pulic-ip

### Frontend:

<img width="1920" height="1020" alt="Image" src="https://github.com/user-attachments/assets/f5720803-26fd-4ba7-950d-d0a742b52095" /> <br>

<img width="1920" height="1020" alt="Image" src="https://github.com/user-attachments/assets/80d56521-f0b7-49ca-b7ff-e560be5a9cf9" />

### Backend: 

<img width="1920" height="1020" alt="Image" src="https://github.com/user-attachments/assets/3aae4a84-d0a5-48ba-adb6-c91fe86012e8" />

### Pipeline Workflow:

<img width="1920" height="1020" alt="Image" src="https://github.com/user-attachments/assets/e75cfb95-5708-48f5-beed-7beff46d2fb4" />


