pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'urbanbasket-svc:3'
        CONTAINER_NAME = 'urbanbasket-svc'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Pre-check Docker') {
            steps {
                sh '''
                    echo "[INFO] Checking for Docker..."
                    if ! command -v docker >/dev/null 2>&1; then
                      echo "[ERROR] Docker not found on this Jenkins agent. Failing pipeline as required."
                      exit 1
                    fi
                    echo "[INFO] Docker is available: $(docker --version)"
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "[INFO] Building Docker image ${DOCKER_IMAGE} from ./flask-app"
                    docker build -t ${DOCKER_IMAGE} ./flask-app
                '''
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    echo "[INFO] Stopping and removing any existing container ${CONTAINER_NAME}"
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true

                    echo "[INFO] Running container ${CONTAINER_NAME} on port 12088 -> 5000"
                    docker run -d -p 12088:5000 --name ${CONTAINER_NAME} ${DOCKER_IMAGE}
                '''
            }
        }

        stage('Test Endpoint') {
            steps {
                sh '''
                    echo "[INFO] Waiting for container to start..."
                    sleep 5

                    echo "[INFO] Curling http://localhost:12088"
                    curl -v http://localhost:12088 || {
                      echo "[ERROR] Failed to reach Flask service on port 12088";
                      exit 1;
                    }
                '''
            }
        }
    }

    post {
        always {
            sh '''
                echo "[INFO] Post-build: listing Docker containers (if Docker is present)..."
                if command -v docker >/dev/null 2>&1; then
                  docker ps -a || true
                else
                  echo "[WARN] Docker not installed; skipping docker ps -a"
                fi
            '''
        }
    }
}
