pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        IMAGE_NAME = "${DOCKERHUB_CREDENTIALS_USR}/my-node-app"
        GIT_COMMIT = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
    }
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${GIT_COMMIT} ."
                sh "docker tag ${IMAGE_NAME}:${GIT_COMMIT} ${IMAGE_NAME}:latest"
            }
        }
        stage('Push to Dockerhub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh "docker push ${IMAGE_NAME}:${GIT_COMMIT}"
                sh "docker push ${IMAGE_NAME}:latest"
            }
        }
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh '''
                # Ví dụ triển khai lên máy chủ
                docker pull ${IMAGE_NAME}:latest
                docker stop my-node-app || true
                docker rm my-node-app || true
                docker run -d --name my-node-app -p 3000:3000 ${IMAGE_NAME}:latest
                '''
            }
        }
    }
    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}