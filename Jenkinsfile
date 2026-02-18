pipeline {
    agent {
        docker { image 'node:18-alpine' }
    }
    environment {
        IMAGE_NAME = 'myapp'
        REGISTRY = 'yassine251/mon-app-devops'
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-credentials')
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Unit Tests') {
            steps {
                sh 'npm install'
                sh 'npm test'
            }
        }
        stage('Docker Build') {
            steps {
                sh "/usr/bin/docker build \
                -t ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} \
                -t ${REGISTRY}/${IMAGE_NAME}:latest \
                ."
            }
        }
        stage('Docker Push') {
            steps{
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "echo $DOCKER_PASSWORD | /usr/bin/docker login -u $DOCKER_USERNAME --password-stdin"
                    sh "/usr/bin/docker push ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}"
                    sh "/usr/bin/docker push ${REGISTRY}/${IMAGE_NAME}:latest"
                }
            }
        }
    }
}