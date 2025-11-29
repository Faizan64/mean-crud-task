Here is your improved README.md including a clean architecture diagram section.
You can paste this directly into GitHub.


---

MEAN CRUD Deployment — Docker + Jenkins + Nginx + MongoDB

System Architecture

The system architecture consists of:

Angular frontend served via Nginx

Node.js + Express backend

MongoDB database running in container

Nginx reverse proxy routing all traffic through port 80

Jenkins automatically building and deploying using Docker images

Docker Hub storing production-ready container images

EC2/Ubuntu VM running application stack



---

📁 Project Structure

frontend/        → Angular UI
backend/         → Node.js API
docker-compose.yml
Jenkinsfile
README.md


---

🧩 Components Used

Component	Technology

Frontend	Angular
Backend	Node.js + Express
Database	MongoDB
Deployment	Docker + Docker Compose
CI/CD	Jenkins
Reverse Proxy	Nginx
Hosting	Ubuntu VM (AWS/Azure/GCP)
Image storage	Docker Hub
Source Code	GitHub



---

1️⃣ Clone the Repository

git clone https://github.com/<your-username>/mean-crud-deployment.git
cd mean-crud-deployment


---

2️⃣ Dockerization

Backend Dockerfile

FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm","start"]

Frontend Dockerfile

FROM node:18 as build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80


---

3️⃣ Build & Push Docker Images

docker build -t <dockerhub-user>/mean-backend ./backend
docker build -t <dockerhub-user>/mean-frontend ./frontend

docker push <dockerhub-user>/mean-backend
docker push <dockerhub-user>/mean-frontend


---

4️⃣ Cloud Server Setup (Ubuntu)

ssh ubuntu@<server-ip>

Install Docker:

sudo apt update
sudo apt install -y docker.io docker-compose
sudo systemctl enable docker


---

5️⃣ Deployment With Docker Compose

Create docker-compose.yml:

version: "3.8"
services:
  mongo:
    image: mongo
    container_name: mongo
    volumes:
      - mongo_data:/data/db
    ports:
      - "27017:27017"

  backend:
    image: <dockerhub-user>/mean-backend
    container_name: backend
    environment:
      - MONGO_URL=mongodb://mongo:27017/mean
    ports:
      - "3000:3000"
    depends_on:
      - mongo

  frontend:
    image: <dockerhub-user>/mean-frontend
    container_name: frontend
    ports:
      - "4200:80"
    depends_on:
      - backend

volumes:
  mongo_data:

Run deployment:

sudo docker-compose up -d


---

6️⃣ Nginx Reverse Proxy Configuration

Install Nginx:

sudo apt install nginx -y

Edit config:

sudo nano /etc/nginx/sites-available/default

Paste:

server {
    listen 80;

    location / {
        proxy_pass http://localhost:4200;
    }

    location /api/ {
        proxy_pass http://localhost:3000/;
    }
}

Restart:

sudo systemctl restart nginx

App now runs at:

http://<server-ip>


---

7️⃣ Jenkins CI/CD Setup

Install Jenkins:

sudo apt install openjdk-17-jdk -y
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins -y
sudo systemctl start jenkins


---

Jenkins Credentials Required

Credential	ID	Purpose

Docker Hub login	dockerhub	push images
SSH private key	server-key	deploy to server



---

Jenkinsfile (Pipeline Automation)

pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/<your>/mean-crud-deployment.git'
            }
        }

        stage('Build images') {
            steps {
                sh 'docker build -t <dockerhub-user>/mean-backend ./backend'
                sh 'docker build -t <dockerhub-user>/mean-frontend ./frontend'
            }
        }

        stage('Login DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                }
            }
        }

        stage('Push images') {
            steps {
                sh 'docker push <dockerhub-user>/mean-backend'
                sh 'docker push <dockerhub-user>/mean-frontend'
            }
        }

        stage('Deploy to Server') {
            steps {
                sshagent(['server-key']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@<SERVER_IP> "
                    cd /app &&
                    docker-compose pull &&
                    docker-compose down &&
                    docker-compose up -d"
                    '''
                }
            }
        }
    }
}


---

✔ Final Application

Accessible in browser:

http://<server-ip>


---

📸 Required Screenshots for Submission

✔ Jenkins Pipeline success
✔ Docker images in Docker Hub
✔ Nginx config
✔ Docker containers running on VM
✔ Browser UI running
✔ GitHub repo structure
✔ Architecture diagram (already included above)


---

🏁 Conclusion

This project demonstrates real DevOps lifecycle using:

Continuous Integration

Continuous Deployment

Cloud compute

Containers

Reverse proxy networking

Version control

Automated build and release