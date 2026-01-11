# Spider DevOps Induction Project

This repository documents my complete end-to-end implementation of the **Spider DevOps Induction Task**, covering **local setup, Dockerization, Nginx reverse proxy, and CI/CD automation using Jenkins and GitHub Webhooks**.

The project consists of:
- **Backend**: Rust (Actix-Web) + Diesel ORM + PostgreSQL  
- **Frontend**: React  
- **Infrastructure**: Docker, Docker Compose, Nginx  
- **CI/CD**: Jenkins (Dockerized) + GitHub App + Docker Hub  


## Objectives
- Containerize a full-stack application
- Use Docker Compose for multi-service orchestration
- Set up Nginx as a reverse proxy
- Implement CI/CD using Jenkins
- Secure secrets using Jenkins Credentials
- Trigger automated builds on every GitHub push


## Tech Stack

| Layer            | Technology              |
|------------------|-------------------------|
| Backend          | Rust, Actix-Web         |
| ORM              | Diesel                  |
| Database         | PostgreSQL              |
| Frontend         | React                   |
| Reverse Proxy    | Nginx                   |
| Containerization | Docker, Docker Compose  |
| CI/CD            | Jenkins (Docker)        |
| Registry         | Docker Hub              |
| Webhooks         | GitHub App + ngrok      |


## Local Setup

### Repository Setup
```
git clone https://github.com/Smya-Gangwar/Spider-Devops-Induction.git
cd Spider-Devops-Induction
```

### Install Homebrew
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
Homebrew is a package manager for macOS/Linux that simplifies installing developer tools.

### Install PostgreSQL
```
brew install postgresql
brew services start postgresql
```
Create database:
```
psql postgres
CREATE DATABASE rust_server;
\q
```

### Install Rust
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source ~/.zshrc
```
Rust is a systems programming language known for performance and safety.


## Backend Setup (Rust + Diesel)

```
cd Backend
```
Install Diesel CLI (Postgres support)
```
cargo install diesel_cli --no-default-features --features postgres
```
Environment Variables
Create .env
```
DATABASE_URL=postgres://postgres:postgres@db:5432/rust_server
```
Initialize Database
```
diesel setup
```
Run Backend
```
cargo run
```

### Backend Error Resolutions
Resolved dependency conflicts using:
```
rm Cargo.lock
cargo clean
rm -rf ~/.cargo/registry
rm -rf ~/.cargo/git
cargo update -p time --precise 0.3.17
cargo build
cargo run
```
Removed the following from Cargo.toml:
```
[patch.crates-io]
time = "=0.3.17"
```


## Frontend Setup (React)

```
cd Frontend
brew install node@18
brew link node@18 --force
rm -rf node_modules package-lock.json
npm install
npm audit fix
npm install react-scripts@5.1.3
npm install http-proxy-middleware
npm start
```


## Dockerization

Files Created
```
Backend/Dockerfile
Frontend/Dockerfile
Frontend/.dockerignore
docker-compose.yml
```

Key Changes
1. setupProxy.js
```
target: "http://backend:8080"
```
2. Backend .env (Docker networking)
```
DATABASE_URL=postgres://postgres:postgres@db:5432/rust_server
```
3. Run Containers
```
docker-compose down
docker-compose build --no-cache
docker-compose up
```


## Nginx Reverse Proxy

### Backend Binding Fix
In Backend/src/main.rs:
```
.bind("0.0.0.0:8080")
```
This is mandatory for Docker container networking.

### Files Added
```
nginx/nginx.conf
nginx/Dockerfile
```

### Commands
```
docker-compose down -v
docker-compose build
docker-compose up -d
```


## Jenkins Setup (Dockerized)

Why Docker Jenkins?
1. Portable
2. Same environment everywhere
3. Production-like
4. Version controlled

### Setup
```
cd Jenkins
docker-compose up -d
```
Get admin password:
```
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```
Access Jenkins: http://localhost:8080


## CI/CD Automation (GitHub + Jenkins)

### GitHub App Setup
```
App Name: jenkins-ci-cd-spider
Homepage URL: http://localhost:8080
```

### Webhook via ngrok
Install ngrok:
```
brew install ngrok
ngrok config add-authtoken <YOUR_TOKEN>
ngrok http 8080
```
GitHub does not allow localhost webhooks, so ngrok is required.

### Jenkins Credentials Stored
1. GitHub App (Private Key + App ID)
2. GitHub Webhook Secret
3. Docker Hub Username
4. Docker Hub Access Token
All secrets are stored securely in Jenkins Credentials.


## Restarting the Full System

1. docker desktop start
2. Up Jenkins Container
```
cd Jenkins && docker-compose up -d
```
3. Start ngrok tunnel
```
ngrok http 8080 --host-header=rewrite
```
Any push to GitHub triggers the CI/CD pipeline automatically.


## CI/CD Flow

1. GitHub Push
2. GitHub Webhook
3. Jenkins Pipeline
    a. Checkout Code
    b. Build Docker Images
    c. Push Images to Docker Hub
    d. docker-compose pull
    e. docker-compose up -d
4. Nginx + Backend + Frontend + PostgreSQL
5. Application live on http://localhost/


## Cloud Hosting (AWS EC2 Deployment)

This level extends the CI/CD pipeline by deploying the containerized application to a cloud-hosted AWS EC2 instance.

### AWS EC2 Instance Setup
1. Login to AWS Console
2. Create a new EC2 instance with the following configuration:
    a. Instance Name: spider-devops-ec2
    b. AMI: Ubuntu Server 24.04 LTS
    c. Instance Type: t3.small
    d. Storage: 25 GB (gp3)
    e. Public IPv4: Enabled
    f. Key Pair:
        1. Name: spider-ec2-key-v2
        2. Type: RSA
        3. Format: .pem

#### Security Group Rules
| Port	  | Protocol 	| Purpose                               |
|---------|-------------|---------------------------------------|
| 22	  | TCP	        | SSH                                   |
| 80	  | TCP	        | HTTP                                  |
| 443	  | TCP	        | HTTPS                                 |
| 8080	  | TCP	        | Jenkins (only if Jenkins runs on EC2) |

Source: IPv4 – Anywhere

3. Launch the Instance

### Login to EC2 Instance
```
ssh -i spider-ec2-key-v2.pem ubuntu@3.109.1.112
```

### Install Required Packages on EC2
```
sudo apt update
sudo apt install -y docker.io docker-compose-plugin
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker ubuntu
sudo apt install -y docker-compose
newgrp docker
```

### Clone Project Repository on EC2
```
git clone https://github.com/Smya-Gangwar/Spider-Devops-Induction.git
cd Spider-Devops-Induction
```
- Updated *docker-compose.yml* to pull images from Docker Hub
- Updated *Jenkinsfile* to deploy via SSH to EC2

### Jenkins → EC2 SSH Authentication
#### Add Jenkins Credential
Kind: SSH Username with private key
Scope: Global
ID: ec2-ssh-key
Description: AWS EC2 SSH Key
Username: ubuntu
Private Key: contents of spider-ec2-key-v2.pem

### GitHub SSH Access from EC2
Generate SSH key on EC2:
```
ssh-keygen -t ed25519 -C "ec2-spider-devops"
```

Add key to SSH agent:
```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

Verify GitHub access:
```
ssh -T git@github.com
```

Update repository remote:
```
git remote set-url origin git@github.com:Smya-Gangwar/Spider-Devops-Induction.git
```

#### Add SSH Key to GitHub
Title: EC2 SSH Key
Key Type: Authentication Key
Key: contents of ~/.ssh/id_ed25519.pub

### Cloud Deployment Flow
1. GitHub push triggers Jenkins pipeline
2. Jenkins builds Docker images
3. Images pushed to Docker Hub
4. Jenkins SSHs into EC2
5. EC2 pulls latest images
6. Containers restarted using Docker Compose
7. Application becomes live

### Live Application URL
```
http://3.109.1.112
```
