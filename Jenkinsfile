pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = "namit77/jenkins-maven-app"    // change to your Docker Hub repo
        AWS_HOST = "ubuntu@3.110.105.247"               // your EC2 username + public IP
        AWS_KEY = credentials('aws-ec2-key')            // Jenkins credential ID for your PEM
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

        stage('Test') {
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

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS')
                ]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push ${DOCKER_HUB_REPO}:latest"
                }
            }
        }

        stage('Deploy to AWS EC2') {
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
            echo "Successfully Deployed to EC2 🚀"
        }
        failure {
            echo "Pipeline Failed ❌"
        }
    }
}
