pipeline {
    agent any

    environment {
        IMAGE = "python-cicd-demo"
        CONTAINER_NAME = "demo"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Using source code checked out by Jenkins job SCM config"
                sh 'ls -la'
            }
        }

        stage('Prepare Python') {
            steps {
                // detect python3 location (works even if Jenkins PATH is limited)
                sh '''
                set -e
                echo "Detecting python3..."
                if command -v python3 >/dev/null 2>&1; then
                  export PYTHON_CMD=$(command -v python3)
                elif [ -x /opt/homebrew/bin/python3 ]; then
                  export PYTHON_CMD=/opt/homebrew/bin/python3
                elif [ -x /usr/local/bin/python3 ]; then
                  export PYTHON_CMD=/usr/local/bin/python3
                else
                  export PYTHON_CMD=""
                fi
                echo "PYTHON_CMD=$PYTHON_CMD"
                # persist for following stages by writing to a temp file
                echo "$PYTHON_CMD" > .python_cmd
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                // Use python3 -m pip if available, otherwise skip install (tests will run in Docker fallback)
                sh '''
                set -e
                PYTHON_CMD="$(cat .python_cmd || true)"
                if [ -n "$PYTHON_CMD" ]; then
                  echo "Using $PYTHON_CMD to install dependencies"
                  "$PYTHON_CMD" -m pip install --upgrade pip || true
                  "$PYTHON_CMD" -m pip install -r requirements.txt
                else
                  echo "python3 not found in PATH for Jenkins. Skipping host install; tests will run inside a python Docker container."
                fi
                '''
            }
        }

        stage('Run Tests') {
            steps {
                // If python is available on host run pytest, otherwise run tests inside a python Docker container
                sh '''
                set -e
                PYTHON_CMD="$(cat .python_cmd || true)"
                if [ -n "$PYTHON_CMD" ]; then
                  echo "Running tests with host Python: $PYTHON_CMD"
                  "$PYTHON_CMD" -m pytest -q
                else
                  echo "Running tests inside python:3.10-slim Docker container (fallback)"
                  docker run --rm -v "$PWD":/app -w /app python:3.10-slim \
                    bash -lc "pip install -r requirements.txt && pytest -q"
                fi
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                // Build the app Docker image
                sh '''
                set -e
                echo "Building Docker image ${IMAGE}:latest"
                docker build -t ${IMAGE}:latest .
                docker images ${IMAGE} --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
                '''
            }
        }

        stage('Run Container') {
            steps {
                // Stop any old container then run new one on port 5000
                sh '''
                set -e
                echo "Stopping any existing container named ${CONTAINER_NAME}..."
                docker rm -f ${CONTAINER_NAME} >/dev/null 2>&1 || true
                echo "Starting container ${CONTAINER_NAME} -> http://localhost:5000"
                docker run -d --name ${CONTAINER_NAME} -p 5000:5000 ${IMAGE}:latest
                docker ps --filter "name=${CONTAINER_NAME}" --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"
                '''
            }
        }
    }

    post {
        always {
            // show workspace and container status to help debug
            sh '''
            echo "Workspace contents:"
            ls -la
            echo "Docker containers (quick):"
            docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}" || true
            '''
        }
        success {
            echo "Pipeline finished successfully."
        }
        failure {
            echo "Pipeline failed — check the console output for errors above."
        }
    }
}
