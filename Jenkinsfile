pipeline {
    agent any

    environment {
        IMAGE_NAME = "niharpatel0011/jenkins_CICD"
        TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/niharp0011/portfoliyo.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $IMAGE_NAME:$TAG .
                    docker tag $IMAGE_NAME:$TAG $IMAGE_NAME:latest
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        set +x
                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" --password-stdin

                        docker push $IMAGE_NAME:$TAG
                        docker push $IMAGE_NAME:latest
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy on EC2') {
            steps {
                sh '''
                    docker stop student-app || true
                    docker rm student-app || true

                    docker pull $IMAGE_NAME:latest

                    docker run -d \
                        --name student-app \
                        -p 80:80 \
                        --restart unless-stopped \
                        $IMAGE_NAME:latest

                    docker ps
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Build, push and deployment completed!'
        }
        failure {
            echo 'FAILED: Check the Jenkins Console Output.'
        }
    }
}
