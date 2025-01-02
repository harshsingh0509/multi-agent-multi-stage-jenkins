pipeline {
    agent none
    stages {
        stage('Clone Repository') {
            agent any
            steps {
                git 'https://your-repo-url.git'
            }
        }
        stage('Build Frontend') {
            agent {
                label 'frontend-agent'
            }
            steps {
                script {
                    docker.image('node:16-alpine').inside {
                        sh 'npm install'
                        sh 'npm run build'
                    }
                }
            }
        }
        stage('Build Backend') {
            agent {
                label 'backend-agent'
            }
            steps {
                script {
                    docker.image('maven:3.8.1-adoptopenjdk-11').inside {
                        sh 'mvn clean install'
                    }
                }
            }
        }
        stage('Setup Database') {
            agent {
                label 'database-agent'
            }
            steps {
                sh """
                  docker run --name postgres -e POSTGRES_PASSWORD=mysecretpassword -d -p 5432:5432 postgres:13-alpine
                  sleep 10
                  docker exec -i postgres psql -U postgres -c "CREATE DATABASE mydb;"
                """
            }
        }
        stage('Verify Containers') {
            agent any
            steps {
                script {
                    def frontendStatus = sh(script: 'docker ps | grep node:16-alpine', returnStatus: true)
                    def backendStatus = sh(script: 'docker ps | grep maven:3.8.1-adoptopenjdk-11', returnStatus: true)
                    def dbStatus = sh(script: 'docker ps | grep postgres:13-alpine', returnStatus: true)

                    if (frontendStatus == 0 && backendStatus == 0 && dbStatus == 0) {
                        echo 'All containers are running as expected'
                    } else {
                        error 'One or more containers did not start as expected'
                    }
                }
            }
        }
    }
    post {
        always {
            script {
                sh 'docker stop $(docker ps -q --filter ancestor=node:16-alpine)'
                sh 'docker stop $(docker ps -q --filter ancestor=maven:3.8.1-adoptopenjdk-11)'
                sh 'docker stop $(docker ps -q --filter ancestor=postgres:13-alpine)'
                echo 'Stopped all running containers'
            }
        }
    }
}
