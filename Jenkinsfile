pipeline {
    agent any

    environment {
        IMAGE = "python-cicd-demo"
        CONTAINER_NAME = "demo"
        DOCKER = "/usr/local/bin/docker"
    }

    stages {
        stage('Checkout') {
            steps { sh 'ls -la' }
        }

        stage('Detect Tools') {
            steps {
                sh '''
                if command -v python3 >/dev/null 2>&1; then
                  command -v python3 > .python_cmd
                else
                  echo "" > .python_cmd
                fi
                echo "/usr/local/bin/docker" > .docker_cmd
                echo "PYTHON_CMD=$(cat .python_cmd)"
                echo "DOCKER_CMD=$(cat .docker_cmd)"
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                PYTHON_CMD="$(cat .python_cmd || true)"
                if [ -n "$PYTHON_CMD" ]; then
                  "$PYTHON_CMD" -m pip install -r requirements.txt || true
                fi
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                PYTHON_CMD="$(cat .python_cmd || true)"
                DOCKER_CMD="$(cat .docker_cmd || true)"
                if [ -n "$PYTHON_CMD" ]; then
                  "$PYTHON_CMD" -m pytest -q
                else
                  $DOCKER_CMD run --rm -v "$PWD":/app -w /app python:3.10-slim \
                    bash -lc "pip install -r requirements.txt && pytest -q"
                fi
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                DOCKER_CMD="$(cat .docker_cmd || true)"
                $DOCKER_CMD build -t ${IMAGE}:latest .
                '''
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                DOCKER_CMD="$(cat .docker_cmd || true)"
                $DOCKER_CMD rm -f ${CONTAINER_NAME} >/dev/null 2>&1 || true
                $DOCKER_CMD run -d --name ${CONTAINER_NAME} -p 5000:5000 ${IMAGE}:latest
                '''
            }
        }
    }

    post {
        always {
            sh '''
            echo "Docker (if available):"
            DOCKER_CMD="$(cat .docker_cmd || true)"
            $DOCKER_CMD --version || true
            $DOCKER_CMD ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}" || true
            '''
        }
    }
}
