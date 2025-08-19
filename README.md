🚀 Continuous Deployment Pipeline Documentation Overview:-

This document outlines the Jenkins Continuous Deployment pipeline configuration for deploying a shopping cart application using Docker containers.

Pipeline Configuration groovy pipeline { agent any

stages {
    stage('Docker Deploy to Container') {
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

🔑 Pipeline Explanation

The deployment script performs the following operations:

Container Cleanup: Removes any existing container named shopping-cart to prevent conflicts

New Deployment: Runs a new container instance with:

Container name: shopping-cart

Port mapping: 8070 (host) → 8070 (container)

Image source: Latest version from lubhitdocker/shopping on DockerHub

📊 Quality Assurance SonarQube Analysis:

Accessed via containerized deployment

Login credentials: admin/admin

Project quality metrics and code analysis verified

🔌 Jenkins Plugin Configuration The following plugins were installed and configured in Jenkins:

SonarQube Scanner Plugin - For code quality analysis

OWASP Dependency-Check Plugin - For vulnerability scanning

Docker Pipeline Plugin - For Docker integration

Docker Plugin - For container management

🛠️ Technology Stack Runtime: Java 21 (OpenJDK)

Framework: Spring Boot with Thymeleaf and Spring Security

Build Tool: Maven 3

CI/CD: Jenkins

Code Quality: SonarQube (deployed via Docker)

Security Scan: OWASP Dependency Check

Containerization: Docker & DockerHub

Development Environment: Eclipse IDE

🙌 Acknowledgements

Original Project Author: Adi Jaiswal

Community Support: Jenkins & DevOps community for plugin support and documentation

Open Source Contributors: Various open source projects that made this implementation possible

📜 License Information

This project configuration is intended for educational and demonstration purposes only. The original application implementation belongs to jaiswaladi246/Ekart.
