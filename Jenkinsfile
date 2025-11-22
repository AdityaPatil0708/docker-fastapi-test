pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'fastapi-app'
        DOCKER_TAG = "${BUILD_NUMBER}"
        COMPOSE_PROJECT_NAME = 'fastapi-project'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                script {
                    sh 'docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .'
                    sh 'docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest'
                }
            }
        }
        
        stage('Run Tests') {
            steps {
                echo 'Running tests...'
                script {
                    sh '''
                        docker run --rm ${DOCKER_IMAGE}:${DOCKER_TAG} \
                        python -c "from app import services; print('Import test passed')"
                    '''
                }
            }
        }
        
        stage('Stop Old Containers') {
            steps {
                echo 'Stopping old containers...'
                script {
                    sh '''
                        docker-compose down || true
                    '''
                }
            }
        }
        
        stage('Deploy with Docker Compose') {
            steps {
                echo 'Deploying application...'
                script {
                    sh '''
                        # Create required directories
                        mkdir -p prometheus grafana/provisioning/datasources
                        
                        # Deploy using docker-compose
                        docker-compose up -d
                        
                        # Wait for services to be healthy
                        sleep 10
                    '''
                }
            }
        }
        
        stage('Health Check') {
            steps {
                echo 'Performing health check...'
                script {
                    sh '''
                        # Check if FastAPI is responding
                        curl -f http://localhost:8000/ || exit 1
                        
                        # Check if Prometheus is up
                        curl -f http://localhost:9090/-/healthy || exit 1
                        
                        # Check if Grafana is up
                        curl -f http://localhost:3000/api/health || exit 1
                    '''
                }
            }
        }
        
        stage('Verify Data Persistence') {
            steps {
                echo 'Verifying data persistence...'
                script {
                    sh '''
                        # Check if data directory exists
                        if [ -d "./app/data" ]; then
                            echo "Data directory exists"
                            ls -la ./app/data
                        else
                            echo "Data directory will be created on first use"
                        fi
                    '''
                }
            }
        }
    }
    
    post {
        success {
            echo 'Deployment successful!'
            echo 'FastAPI: http://localhost:8000'
            echo 'Prometheus: http://localhost:9090'
            echo 'Grafana: http://localhost:3000 (admin/admin)'
        }
        failure {
            echo 'Deployment failed!'
            sh 'docker-compose logs'
        }
        always {
            echo 'Cleaning up unused Docker images...'
            sh 'docker image prune -f'
        }
    }
}