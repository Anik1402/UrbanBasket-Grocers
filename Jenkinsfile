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
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${env.DOCKER_IMAGE}", "./flask-app")
                }
            }
        }
        
        stage('Run Container') {
            steps {
                script {
                    sh "docker stop ${env.CONTAINER_NAME} || true"
                    sh "docker rm ${env.CONTAINER_NAME} || true"
                    sh "docker run -d -p 12088:5000 --name ${env.CONTAINER_NAME} ${env.DOCKER_IMAGE}"
                }
            }
        }
        
        stage('Test') {
            steps {
                script {
                    // Wait for container to start
                    sleep 5
                    // Test the endpoint
                    sh 'curl -s http://localhost:12088'
                }
            }
        }
    }
    
    post {
        always {
            // Clean up
            script {
                sh "docker ps -a"
            }
        }
    }
}
