pipeline {
    agent any

    environment {
        AWS_KEY = credentials('aws-ec2-key')
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/Namita-Dewang/jenkins-maven-cicd.git'
            }
        }

        stage('Build Maven Project') {
            steps {
                bat 'mvn clean package'
            }
        }

        stage('Run Tests') {
            steps {
                bat 'mvn test'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t namit77/jenkins-maven-cicd:latest .'
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                 usernameVariable: 'USER',
                                                 passwordVariable: 'PASS')]) {
                    bat """
                    docker login -u %USER% -p %PASS%
                    docker push namit77/jenkins-maven-cicd:latest
                    """
                }
            }
        }

        stage('Deploy on AWS EC2') {
            steps {
                bat """
                "C:\\Program Files\\Git\\usr\\bin\\ssh.exe" -o StrictHostKeyChecking=no -i %AWS_KEY% ubuntu@3.110.105.247 "
                    docker pull namit77/jenkins-maven-cicd:latest &&
                    docker stop cicd_app || true &&
                    docker rm cicd_app || true &&
                    docker run -d --name cicd_app -p 8080:8080 namit77/jenkins-maven-cicd:latest
                "
                """
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Completed Successfully!"
        }
        failure {
            echo "❌ Pipeline Failed — check console output."
        }
    }
}
