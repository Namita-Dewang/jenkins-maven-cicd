pipeline {
    agent any

    environment {
        // Your Docker Hub repo (update if your repo name is different)
        DOCKER_HUB_REPO = "namit77/jenkins-maven-cicd"

        // Your EC2 username and public IP
        AWS_HOST = "ubuntu@3.110.105.247"

        // Jenkins credential IDs EXACTLY as you have them
        AWS_KEY = credentials('aws-ec2-key')
    }

    stages {

        stage('Checkout') {
            steps {
                git(
                    url: 'https://github.com/Namita-Dewang/jenkins-maven-cicd.git',
                    credentialsId: 'github-creds'
                )
            }
        }

        stage('Build Maven Project') {
            steps {
                sh 'mvn -B clean package'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_HUB_REPO}:latest ."
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push ${DOCKER_HUB_REPO}:latest"
                }
            }
        }

        stage('Deploy on AWS EC2') {
            steps {
                script {
                    sh """
                    ssh -o StrictHostKeyChecking=no -i ${AWS_KEY} ${AWS_HOST} '
                        sudo docker pull ${DOCKER_HUB_REPO}:latest &&
                        sudo docker stop myapp || true &&
                        sudo docker rm myapp || true &&
                        sudo docker run -d --name myapp -p 80:80 ${DOCKER_HUB_REPO}:latest
                    '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Deployment Successful on AWS EC2!"
        }
        failure {
            echo "❌ Pipeline Failed — check console output."
        }
    }
}
