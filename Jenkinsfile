pipeline {
    agent any

    environment {
        IMAGE = "python-cicd-demo"
    }

    stages {
        // We don't need an extra git checkout here because
        // Jenkins already checks out the repo (Declarative: Checkout SCM)
        stage('Checkout') {
            steps {
                echo "Using source code checked out by Jenkins job SCM config"
                sh 'ls -la'
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
                // Stop old container if exists
                sh "docker rm -f demo || true"
                // Run new container
                sh "docker run -d --name demo -p 5000:5000 ${IMAGE}:latest"
            }
        }
    }
}
