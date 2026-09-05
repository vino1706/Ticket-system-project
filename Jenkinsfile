pipeline {
    agent any

    tools {
        maven 'Maven'
        jdk 'JDK-21'
        nodejs 'NodeJS-18'
    }
    environment {
        DOCKER_CREDENTIALS = credentials('docker_hub')
        SONARQUBE_CREDENTIALS = credentials('sonar_token')
    }

    stages {
        stage('Backend server Build') {
            steps {
                echo 'Starting Spring Boot backend application in the background...'
                sh ' mvn spring-boot:run'
            }
        }
        stage('Frontend server Build') {
            steps {
                echo 'Starting Frontend application in the background...'
                sh 'cd frontend && npm install '

            }
        }
        stage('Backend server Test') {
            steps { 
                sh 'mvn test'
                junit '**/target/surefire-reports/*.xml'    
            }
        }
        stage('Frontend server test') {
            steps {
                sh 'npm test'
            }
        }
        stage('SonarQube Scanning') {
            steps {
                echo 'Scanning Code Quality using SonarQube'
                 withCredentials([usernamePassword(credentialsId: 'sonar_token', usernameVariable: 'SONAR_USER', passwordVariable: 'SONAR_PAT')]) {
                sh "mvn clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=jenkins -Dsonar.projectName='jenkins' -Dsonar.host.url=http://16.60.119.201:9000 -Dsonar.token=${SONAR_PAT}"
            }
          }
        }
        stage('Docker Backend build') {
            steps {
                echo 'Verifying that application services are live...'
                sh 'docker build -t backendbookingapp:v1 . '
            }
        }
        stage('Docker Frontend build') {
            steps {
                echo 'Verifying that application services are live...'
                sh '''
                cd frontend/
                docker build -t frontendbookingapp:v1 . '
                '''
            }
        }
        stage('Docker Login & Push:Backend') {
            steps {
                // Ensure you choose "Username and password" or "Secret text" in Jenkins credentials
                withCredentials([usernamePassword(credentialsId: 'docker_hub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PAT')]) {
                    sh '''
                        echo "$DOCKER_PAT" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push "$DOCKER_USER"/backendbookingapp:v1
                    '''
                }
            }
        }
        stage('Docker Login & Push:Frontend') {
            steps {
                // Ensure you choose "Username and password" or "Secret text" in Jenkins credentials
                withCredentials([usernamePassword(credentialsId: 'docker_hub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PAT')]) {
                    sh '''
                        echo "$DOCKER_PAT" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push "$DOCKER_USER"/frontendbookingapp:v1
                    '''
                }
            }
        }
        stage('Docker run ') {
            steps {
                echo 'Spinning up containers for Monitoring Stacks'
                dir('monitoring') {
                    sh '''
                        docker compose up -d --build
                    '''
                }
            }
        }
        stage('Prometheus & Grafana Monitoring') {
            steps {
                echo 'Spinning up containers for Monitoring Stacks'
                dir('monitoring') {
                    sh '''
                        cd frontend
                        docker compose up -d --build
                    '''
                }
            }
        }
        stage('Verify Health') {
            steps {
                echo 'Waiting for monitoring web servers to initialize...'
                sleep 10
                echo 'Verifying endpoint accessibility...'
                sh 'curl -I http://localhost:9090' // Verifies Prometheus Port
                sh 'curl -I http://localhost:3000' // Verifies Grafana Port
            }
        }
    }
    post {
        success {
            echo '============================================================='
            echo ' BookMyShow project is actively running!'
            echo ' Backend API: http://localhost:8080'
            echo ' Frontend Client: http://localhost:5173'
            echo '============================================================='
        }
        failure {
            echo 'Application failed to start. Printing logs...'
            sh 'cat backend.log || true'
            sh 'cat frontend/frontend.log || true'
        }
    }
}
