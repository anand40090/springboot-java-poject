# Jenkins CI-CD pipeline using docker, maven

________________

In this project we are building CI-CD pipeline for java based project using maven and docker tool. 

_______________

## Seting up tools 

1. Install the required plugins and tools in the jenkins
   - Docker
   - Sonarqube
   - Maven
   - JDK
2. Configure the tools in Jenkins
3. Create token for Sonarqube and configure it in jenkins 
4. Configure docker credentials in jenkins
5. Create webhook in sonarqube
-----------------------

## Install sonarqube using docker compose 

Using a Docker container to run SonarQube can simplify the setup process significantly. 
Below are the steps to install and run SonarQube using Docker on Ubuntu.

### Prerequisites - 

1.  Install Docker 
```
sudo apt update
sudo apt install docker.io
sudo systemctl start docker
sudo systemctl enable docker

```
2. Install Docker Compose:

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

```

### Set Up SonarQube with Docker

1. Create a Docker Compose file:
Create a directory for SonarQube and a Docker Compose file:

```
mkdir sonarqube-docker
cd sonarqube-docker
nano docker-compose.yml

```

2.  Edit the docker-compose.yml file:
Add the following content to the docker-compose.yml file:
```
version: "3"
services:
  sonarqube:
    image: sonarqube:latest
    container_name: sonarqube
    ports:
      - "9000:9000"
    environment:
      - SONAR_JDBC_URL=jdbc:postgresql://db:5432/sonarqube
      - SONAR_JDBC_USERNAME=sonar
      - SONAR_JDBC_PASSWORD=sonar
    depends_on:
      - db
  db:
    image: postgres:latest
    container_name: sonarqube-db
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=sonar
      - POSTGRES_DB=sonarqube
    volumes:
      - "sonarqube_db_data:/var/lib/postgresql/data"
volumes:
  sonarqube_db_data:

```

3. Run Docker Compose:

```
sudo docker-compose up -d

```

### Access SonarQube

1. Open a web browser and go to http://your_server_ip:9000.

2. Log in with the default username and password (admin/admin).
_________________________

# Configure the tools, credentials in jenkins 

1. configure credentials for sonqrqube and docker 
```
Jenkins Dashboard >> Manage Jenkins >> Credentials >> System >> Global credentials (unrestricted)

```

![image](https://github.com/user-attachments/assets/c5bc6d3e-26e6-4a20-bf21-88f6f610342e)

2. configure the maven in jenkins 

![image](https://github.com/user-attachments/assets/bbc01a38-a843-4175-b040-cd848f4d449f)

3. configure sonarqube in jenkins 

![image](https://github.com/user-attachments/assets/e61a8afc-d57a-4acc-8c7a-e41b9947a7bd)

# Create webhook in sonarqube for jenkins
1. Login to sonarqube and created PAT token to be configured in the jenkins, in above screenshot this token have be saved in the jenkins credentials 'sq-1'

![image](https://github.com/user-attachments/assets/bdfaaf3f-a646-4eab-a99b-cc4372c2b171)

2. configure webhook using jenkins URL 

![image](https://github.com/user-attachments/assets/72f96fc5-774e-4c22-b60b-a8e76742f045)

### Jenkins Pipeline 

```
pipeline {
    agent any
    
    tools {
        maven 'M2'
    }
    
    //create environment to call it in the variable later globally//
    environment {
        registry_credentials = "docker-cred" //docker registry credentials configured in jenkins//
        sonarqube_credentials = "sq-1" //sonarqube credentials configured at jenkins//
        docker_registry = "anand40090/spring-boot-demo" //docker registery configured//
    }

    stages {
        stage('Git Pull') {
            steps {
                git branch: 'master', changelog: false, poll: false, url: 'https://github.com/anand40090/springboot-java-poject.git'
            }
        }
        stage('maven image build'){
            steps{
                sh 'mvn clean install'
            }
        }
        stage('Sonar Scan'){
            steps{
                withSonarQubeEnv(credentialsId: 'sq-1', installationName: 'sq-1'){
                    sh 'mvn sonar:sonar'
                }
                waitForQualityGate abortPipeline: false, credentialsId: 'sq-1'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = "${docker_registry}:${BUILD_NUMBER}" //create variable for docker image//
                    sh "docker build -t ${imageName} ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    def imageName = "${docker_registry}:${BUILD_NUMBER}"
                    withCredentials([usernamePassword(credentialsId: "${registry_credentials}", passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                        sh "echo $PASSWORD | docker login -u $USERNAME --password-stdin"
                        sh "docker push ${imageName}"
                    }
                }
            }
        }
        stage('Stop and Remove Previous Container') {
            steps {
                script {
                    def previousContainer = sh(script: "docker ps -aqf name=spring-boot-demo-", returnStdout: true).trim()
                    if (previousContainer) {
                        sh "docker stop ${previousContainer}"
                        sh "docker rm ${previousContainer}"
                    }
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    def imageName = "${docker_registry}:${BUILD_NUMBER}"
                    sh "docker run -itd -p 1244:8080 --name spring-boot-demo-${BUILD_NUMBER} ${imageName}"
                }
            }
        }
        stage('check container'){
            steps{
                sh 'docker ps '
            }
        }
    }
}

```

### Jenkins stages

![image](https://github.com/user-attachments/assets/c47490db-f204-4ddf-b97b-137e2a12043c)

### Sonarqube code scan result 

![image](https://github.com/user-attachments/assets/d603b763-f0bb-4425-9dc3-f181c7d77989)

### Docker images pushed to dockerhub as per the jenkins build number 

![image](https://github.com/user-attachments/assets/24dadd8e-65f7-4ed6-b250-430d7ab36fc8)

### Docker container started as per the jenkins stage 'Run Docker Container'

![image](https://github.com/user-attachments/assets/a9f89562-deca-4cac-ab76-e45dd07309c4)

### Docker container accessible on the defined port 1244 

![image](https://github.com/user-attachments/assets/6c455aae-e97f-4af3-bbe2-0c9611696ca3)


