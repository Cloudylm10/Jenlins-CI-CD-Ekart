# 🛒 Ekart CI/CD Deployment with Jenkins, SonarQube, Docker & OWASP

This repository documents the **CI/CD pipeline setup** for deploying an **Ekart shopping cart web application** using **Jenkins, SonarQube, Maven, Docker, and OWASP Dependency Check**.

---

## 📌 Project Description

I deployed an **Ekart shopping website** made in **Java (Spring Boot + Thymeleaf)**.  
This is a demo project for practicing **Spring + Thymeleaf**.  

<img width="1151" height="530" alt="Jenkins" src="https://github.com/user-attachments/assets/f01370d4-fa24-44e0-8512-47ae5ba8e0c0" />


👉 The idea was to build a **basic shopping cart web app** where users can register, log in, browse products, and add them to their **own shopping cart (session-based)**. Checkout is **transactional**.

### 🛠️ Tech stack of the application:
- Spring Boot
- Spring Security
- Thymeleaf
- Spring Data JPA
- Spring Data REST
- H2 Database (in-memory)
- Dockerized deployment

### Features:
- User registration & login  
- Product shopping & cart functionality  
- Transactional checkout  

> 💡 The source code of the application is **forked** from [jaiswaladi246/Ekart](https://github.com/jaiswaladi246/Ekart).  
Special thanks and credits to the original author 🙏.

---

## ⚙️ CI/CD Pipeline Overview

The pipeline is implemented in **Jenkins** and has two parts:

- **CI Pipeline (Continuous Integration):**
  - Checkout code from GitHub  
  - SonarQube code analysis  
  - OWASP Dependency check  
  - Build app with Maven  
  - Build & push Docker image to DockerHub  
  - Trigger CD pipeline  

- **CD Pipeline (Continuous Deployment):**
  - Deploy the pushed Docker image to a container  
  - Run the application on exposed port  

### 🔄 Pipeline Flow Diagram

![Pipeline Flow](./e49c9d21-3c95-43b1-aefa-f4d4da575089.png)

---

## 🏗️ CI Pipeline Script (Jenkinsfile)

```groovy
pipeline {
    agent any
    
    tools {
        jdk 'Java21'        // Jenkins JDK config
        maven 'maven3'      // Maven tool
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/jaiswaladi246/Ekart.git'
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                sh '''
                $SCANNER_HOME/bin/sonar-scanner \
                  -Dsonar.host.url=http://localhost:9000/ \
                  -Dsonar.login=squ_bb060a5459897718afa79e8df54885243d6f16e4 \
                  -Dsonar.projectName=shopping-cart \
                  -Dsonar.projectKey=shopping-cart \
                  -Dsonar.java.binaries=.
                '''
            }
        }
        
        stage('OWASP SCAN') {
            steps {
               dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP-check'
               dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Build Application') {
            steps {
              sh "mvn clean install -DskipTests=true"
            }
        }
        
        stage('Build & push Docker images') {
            steps {
              script {
                withDockerRegistry([credentialsId: 'aba5182c-c628-4fd6-bd4c-418c70e33be4', url: 'https://index.docker.io/v1/']) {
                    sh "docker build -t shopping:latest -f docker/Dockerfile ."
                    sh "docker tag shopping:latest lubhitdocker/shopping:latest"
                    sh "docker push lubhitdocker/shopping:latest"
                }
              }
            }
        }
        
        stage('Trigger CD pipeline') {
            steps {
              build(job: 'CD_pipeline', wait: true, propagate: true)
            }
        }
    }
}

🔑 Explanation:
SonarQube Analysis: Scans code for bugs, vulnerabilities, and code smells.
OWASP Dependency Check: Finds security risks in project dependencies.
Build Application: Uses Maven to build .jar file.
Docker Build & Push: Builds image and pushes it to DockerHub.
Trigger CD Pipeline: Calls another Jenkins job for deployment.


🚀 CD Pipeline Script (Deployment)
pipeline {
    agent any

    stages {
        stage('Docker Deploy to container') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'aba5182c-c628-4fd6-bd4c-418c70e33be4', url: 'https://index.docker.io/v1/']) {
                        sh '''
                        docker rm -f shopping-cart || true
                        docker run -d --name shopping-cart -p 8070:8070 lubhitdocker/shopping:latest
                        '''
                    }
                }
            }
        }
    }
}

🔑 Explanation:

Removes old container if exists.

Runs new container with exposed port 8070.

Deploys the latest DockerHub image.
