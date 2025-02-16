pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "nalini8123/nginx-sample"
        DOCKER_CREDENTIALS_ID = "Nali-ID1"   
        UBUNTU_CREDENTIALS_ID = "SSH-ID1"        
        UBUNTU_HOST = "ubuntu@13.202.44.124"           
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/Nalini8123/nginx.git'
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

        stage('Deploy to ubuntu') {
            steps {
                sshagent (credentials: ["${UBUNTU_CREDENTIALS_ID}"]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${UBUNTU_HOST} '
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
            echo 'NGINX app successfully deployed on Ubuntu!'
        }
        failure {
            echo 'Deployment failed. Check the logs for errors.'
        }
    }
}
