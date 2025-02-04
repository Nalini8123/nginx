pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "hemashree897/nginx-sample"
        DOCKER_CREDENTIALS_ID = "docker-hub-credentials"   // Jenkins credentials for Docker Hub
        EC2_CREDENTIALS_ID = "ec2-ssh-credentials"         // Jenkins SSH key for EC2
        EC2_HOST = "ec2-user@54.91.239.194"           // Replace with your EC2 instance's public IP
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/your-repo/nginx-sample.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:latest")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_IMAGE}:latest").push()
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent (credentials: ["${EC2_CREDENTIALS_ID}"]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${EC2_HOST} '
                        docker pull ${DOCKER_IMAGE}:latest &&
                        docker stop nginx-sample || true &&
                        docker rm nginx-sample || true &&
                        docker run -d --name nginx-sample -p 80:80 ${DOCKER_IMAGE}:latest
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'NGINX app successfully deployed on EC2!'
        }
        failure {
            echo 'Deployment failed. Check the logs for errors.'
        }
    }
}
