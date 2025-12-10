pipeline {
    agent any

    environment {
        IMAGE = "python-cicd-demo"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/your-username/jenkins-python-demo.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'pytest -q'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE}:latest ."
            }
        }

        stage('Run Container') {
            steps {
                sh "docker rm -f demo || true"
                sh "docker run -d --name demo -p 5000:5000 ${IMAGE}:latest"
            }
        }
    }
}
