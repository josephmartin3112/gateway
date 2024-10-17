pipeline {
    agent any

    tools {
        maven 'Maven-3.9.8'
        jdk 'JDK-21'
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        git url: 'https://github.com/josephmartin3112/gateway', branch: 'main'
                    }
                }
            }
        }

        stage('Pre-Build') {
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        bat '''
                        docker stop gateway-sr || true
                        docker rm gateway-sr || true
                        docker rmi -f gateway-sr:latest || true
                        '''
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        bat 'mvn clean install'
                    }
                }
            }
        }

        stage('Create Docker Network') {
            steps {
                script {
                    // Create a Docker network, ignore errors if it already exists
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        bat 'docker network create my-network || echo "Network already exists"'
                    }
                }
            }
        }

        stage('Post-Build') {
            steps {
                script {
                    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                        bat '''
                        docker build -t gateway-sr:latest .
                        docker run -d --network my-network -p 8222:8222 --name gateway-sr security-sr:latest
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed!'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline had errors, but execution continued.'
        }
    }
}