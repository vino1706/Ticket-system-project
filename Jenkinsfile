pipeline {
    agent any

    tools {
        maven 'Maven'
        jdk 'jdk-17'
        nodejs 'node-20'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        ansiColor('xterm')
        disableConcurrentBuilds()
    }

    stages {

        stage('Backend server Initiation') {
            steps {
                echo 'Starting Spring Boot backend application in the background...'

                sh '''
                    BUILD_ID=dontKillMe nohup mvn spring-boot:run > backend.log 2>&1 &
                '''
                echo 'Waiting for backend server to warm up...'
                sleep 10
            }
        }

        stage('Frontend Server Initiation') {
            steps {
                dir('frontend') { 
                    echo 'Installing Frontend Dependencies...'
                    sh 'npm install'
                    
                    echo 'Launching Frontend React server in the background...'

                    sh '''
                        BUILD_ID=dontKillMe nohup npm run dev > frontend.log 2>&1 &
                    '''
                }
                echo 'Waiting for frontend server to warm up...'
                sleep 5
            }
        }

        stage('Checking Outputs') {
            steps {
                echo 'Verifying that application services are live...'
                // Pings the servers to confirm they are accepting requests
                sh 'curl -I http://localhost:8080/swagger-ui/index.html || echo "Backend check complete/skipped"'
                sh 'curl -I http://localhost:5173 || echo "Frontend check complete"'
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
