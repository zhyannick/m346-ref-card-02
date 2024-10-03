
# Architecture Ref.Card 02 - React Application (serverless)

Link zur Übersicht<br/>
https://gitlab.com/bbwrl/m346-ref-card-overview

## Installation der benötigten Werkzeuge

Für das Bauen der App wird Node bzw. npm benötigt. Die Tools sind unter 
der folgenden URL zu finden. Für die meisten Benutzer:innen empfiehlt sich 
die LTS Version.<br/>
https://nodejs.org/en/download/

Node Version Manager<br/>
Für erfahren Benutzer:innen empfiehlt sich die Installation des 
Node Version Manager nvm. Dieses Tool erlaubt das Installiert und das 
Wechseln der Node Version über die Kommandozeile.<br/>
**Achtung: Node darf noch nicht auf dem Computer installiert sein.**<br/>
https://learn2torials.com/a/how-to-install-nvm


## Inbetriebnahme auf eigenem Computer

Projekt herunterladen<br/>
```git clone git@gitlab.com:bbwrl/m346-ref-card-02.git```
<br/>
```cd architecture-refcard-02```

### Projekt bauen und starten
Die Ausführung der Befehle erfolgt im Projektordner

Builden mit Node/npm<br/>
```$ npm install```

Das Projekt wird gebaut und die entsprechenden Dateien unter dem Ordner node_modules gespeichert.

Die App kann nun mit folgendem Befehl gestartet werden<br/>
```$ npm start```

Die App kann nun im Browser unter der URL http://localhost:3000 betrachtet werden.



### Inbetriebnahme mit Docker Container
1. Create Dockerfile 

 ```
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

2. Create the Docker-Image

```
docker build -t  yannickzh/m346-ref-card-02 .
docker push  yannickzh/m346-ref-card-02
```
### Github Actions

1. Create Workflow directory

```
mkdir .github/workflows
```

2. Create workflow files

ci.yaml
```
name: CI

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - name: Build the Docker image
        run: docker build -t zhyannick/m346-ref-card-02 .
      - name: Login to Docker Hub
        run: docker login -u zhyannick -p  ${{ secrets.DOCKER_PASSWORD }}
      - name: Push the Docker image
        run: docker push zhyannick/m346-ref-card-02
```

deploy.yaml
```

name: Build and Deploy Docker to AWS

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up AWS credentials for session
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_SESSION_TOKEN: ${{ secrets.AWS_SESSION_TOKEN }}
          AWS_DEFAULT_REGION: ${{ secrets.AWS_REGION }}
        run: |
          aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
          aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
          aws configure set aws_session_token $AWS_SESSION_TOKEN
          aws configure set region $AWS_DEFAULT_REGION

      - name: Log in to Amazon ECR
        run: |
          aws ecr get-login-password --region ${{ secrets.AWS_REGION }} | docker login --username AWS --password-stdin ${{ secrets.ECR_REPOSITORY }}

      - name: Build Docker image
        run: docker build -t my-docker-app .

      - name: Tag Docker image
        run: docker tag my-docker-app:latest ${{ secrets.ECR_REPOSITORY }}:latest

      - name: Push Docker image to Amazon ECR
        run: docker push ${{ secrets.ECR_REPOSITORY }}:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest

    steps:
      - name: Set up AWS credentials for session
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_SESSION_TOKEN: ${{ secrets.AWS_SESSION_TOKEN }}
          AWS_DEFAULT_REGION: ${{ secrets.AWS_REGION }}
        run: |
          aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
          aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
          aws configure set aws_session_token $AWS_SESSION_TOKEN
          aws configure set region $AWS_DEFAULT_REGION


      - name: Update ECS service
        run: |
          aws ecs update-service --cluster ref-card-yannick --service ref-card-yannick-service --task-definition ref-card-yannick
```


