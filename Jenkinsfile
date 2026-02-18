pipeline {
    agent any
    environment {
        IMAGE_NAME = 'mon-app-devops'
        REGISTRY = 'yassine251'
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-credentials')
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Unit Tests') {
            agent {
                docker { image 'node:18-alpine' }
            }
            steps {
                sh 'npm install'
                sh 'npm test'
            }
        }
        stage('Docker Build') {
            steps {
                sh "docker build \
                -t ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} \
                -t ${REGISTRY}/${IMAGE_NAME}:latest \
                ."
            }
        }
        stage('Docker Push') {
            steps{
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                    sh "docker push ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}"
                    sh "docker push ${REGISTRY}/${IMAGE_NAME}:latest"
                }
            }
        }
    }
}