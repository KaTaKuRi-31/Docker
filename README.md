📘 Déploiement CI/CD complet avec Docker, GitHub Actions, AWS ECR & EC2
👤 Réalisé par : Dylan aka KaTaKuRi-31
📦 Technologies utilisées :
Docker & Docker Compose

Node.js (Backend) + Nginx (Frontend)

MongoDB

GitHub & GitHub Actions (CI/CD)

AWS EC2 & AWS ECR

🔁 Sommaire
Architecture du projet

Pré-requis

Mise en place du projet Docker

Déploiement sur GitHub

Création du repo AWS ECR

CI/CD avec GitHub Actions

Déploiement depuis ECR vers EC2

📁 1. Architecture du projet
pgsql
Copier
Modifier
projet-docker/
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
├── frontend/
│   └── index.html
├── nginx/
│   └── default.conf
├── docker-compose.yml
└── .github/workflows/docker.yml
🧰 2. Pré-requis
Compte GitHub

Compte AWS avec :

Un utilisateur IAM avec accès ECR

Clés AWS : Access Key ID & Secret Key

AWS CLI installé (si tu travailles avec EC2)

Git installé

Docker installé (local et/ou sur EC2)

🛠️ 3. Mise en place du projet Docker
📁 Créer les dossiers :
bash
Copier
Modifier
mkdir -p projet-docker/{frontend,backend,nginx}
cd projet-docker
🧾 docker-compose.yml
yaml
Copier
Modifier
services:
  frontend:
    image: nginx:alpine
    volumes:
      - ./frontend:/usr/share/nginx/html:ro
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
    ports:
      - "8080:80"
    depends_on:
      - backend

  backend:
    build: ./backend
    ports:
      - "3000:3000"
    depends_on:
      - mongo

  mongo:
    image: mongo
    ports:
      - "27017:27017"
🌐 frontend/index.html
html
Copier
Modifier
<!DOCTYPE html>
<html>
<head><title>Frontend Docker</title></head>
<body>
  <h1>Hello from Frontend</h1>
  <script>
    fetch('/api')
      .then(res => res.json())
      .then(data => {
        const el = document.createElement('pre');
        el.textContent = JSON.stringify(data, null, 2);
        document.body.appendChild(el);
      });
  </script>
</body>
</html>
🧠 backend/server.js
js
Copier
Modifier
const express = require('express');
const mongoose = require('mongoose');
const app = express();

mongoose.connect('mongodb://mongo:27017/testdb')
  .then(() => console.log('MongoDB connected'))
  .catch(err => console.error('MongoDB connection error:', err));

app.get('/api', (req, res) => {
  res.json({ message: 'Hello from the API!' });
});

app.listen(3000, () => console.log('Server running on port 3000'));
📦 backend/package.json
json
Copier
Modifier
{
  "name": "backend",
  "version": "1.0.0",
  "main": "server.js",
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.0.4"
  }
}
🐳 backend/Dockerfile
Dockerfile
Copier
Modifier
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
⚙️ nginx/default.conf
nginx
Copier
Modifier
server {
  listen 80;

  location / {
    root /usr/share/nginx/html;
    index index.html;
  }

  location /api {
    proxy_pass http://backend:3000;
  }
}
🔗 4. Déploiement sur GitHub
bash
Copier
Modifier
cd projet-docker
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/<username>/<repo>.git
git push -u origin main
☁️ 5. Création du repo AWS ECR
Aller dans la console AWS > ECR

Créer un repository privé

Nom : projet-docker

Copier l'URI du repo (ex: 123456789012.dkr.ecr.eu-west-3.amazonaws.com/projet-docker)

🔐 6. CI/CD avec GitHub Actions
🔑 Ajouter les secrets GitHub :
Secret Name	Valeur
AWS_ACCESS_KEY_ID	ta clé IAM
AWS_SECRET_ACCESS_KEY	ta clé secrète IAM
AWS_REGION	eu-west-3 (si Paris)
ECR_REPOSITORY	l'URI complet du repo ECR
🧾 .github/workflows/docker.yml
yaml
Copier
Modifier
name: Build and Push to AWS ECR

on:
  push:
    branches: [ "main" ]

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ secrets.AWS_REGION }}

    - name: Log in to Amazon ECR
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build, tag, and push image to ECR
      env:
        ECR_REPOSITORY: ${{ secrets.ECR_REPOSITORY }}
      run: |
        IMAGE_TAG=latest
        docker build -t $ECR_REPOSITORY:$IMAGE_TAG ./backend
        docker push $ECR_REPOSITORY:$IMAGE_TAG
🚀 Déclencher le pipeline :
bash
Copier
Modifier
git add .
git commit -m "Setup GitHub Actions CI/CD"
git push
🐳 7. Déploiement depuis ECR vers EC2
✅ Sur ton instance EC2 :
Se connecter :

bash
Copier
Modifier
ssh -i docker.pem ubuntu@<ip-publique>
Installer AWS CLI :

bash
Copier
Modifier
sudo apt update
sudo apt install awscli -y
Configurer les credentials :

bash
Copier
Modifier
aws configure
# Tu entres :
# AWS Access Key ID
# AWS Secret Access Key
# Region : eu-west-3
# Output format : json
Se connecter à ECR :

bash
Copier
Modifier
aws ecr get-login-password --region eu-west-3 | docker login --username AWS --password-stdin <URI-ECR>
Récupérer l'image :

bash
Copier
Modifier
docker pull <URI-ECR>:latest
Lancer le conteneur :

bash
Copier
Modifier
docker run -d -p 3000:3000 <URI-ECR>:latest
