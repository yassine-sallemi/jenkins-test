pipeline {
    agent any
    environment {
        IMAGE_NAME = 'mon-app-devops'
        REGISTRY = 'yassine251'
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-credentials')
        HAS_CHANGES = 'false'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    // Check if there are changes since last successful build
                    def lastSuccessfulCommit = null
                    def currentCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()

                    def lastSuccessfulBuild = currentBuild.previousSuccessfulBuild
                    if (lastSuccessfulBuild) {
                        lastSuccessfulCommit = lastSuccessfulBuild.getEnvironment().GIT_COMMIT
                    }

                    if (lastSuccessfulCommit == null || lastSuccessfulCommit != currentCommit) {
                        env.HAS_CHANGES = 'true'
                        echo "Changes detected. Current: ${currentCommit}, Last: ${lastSuccessfulCommit ?: 'none'}"
                    } else {
                        echo "No changes detected since last successful build"
                    }
                }
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
            when {
                expression { env.HAS_CHANGES == 'true' }
            }
            steps {
                sh "docker build \
                -t ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} \
                -t ${REGISTRY}/${IMAGE_NAME}:latest \
                ."
            }
        }
        stage('Docker Push') {
            when {
                expression { env.HAS_CHANGES == 'true' }
            }
            steps{
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                    sh "docker push ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}"
                    sh "docker push ${REGISTRY}/${IMAGE_NAME}:latest"
                }
            }
            post {
                always {
                    sh 'docker logout'
                }
            }
        }
        stage('Cleanup') {
            when {
                expression { env.HAS_CHANGES == 'true' }
            }
            steps {
                script {
                    // Remove dangling images and build cache
                    sh 'docker image prune -f'
                    sh 'docker builder prune -f'
                }
            }
        }
    }
}