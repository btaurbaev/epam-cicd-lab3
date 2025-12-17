pipeline {
    agent any

    tools {
        nodejs 'node-7.8.0'
    }

    environment {
        APP_PORT = ''
        IMAGE_NAME = ''
        CONTAINER_NAME = ''
        ENV_NAME = ''
    }

    stages {

        stage('Init env') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        ENV_NAME = 'prod'
                        APP_PORT = '3000'
                        IMAGE_NAME = 'nodemain:v1.0'
                        CONTAINER_NAME = 'node-main'
                    } else if (env.BRANCH_NAME == 'dev') {
                        ENV_NAME = 'dev'
                        APP_PORT = '3001'
                        IMAGE_NAME = 'nodedev:v1.0'
                        CONTAINER_NAME = 'node-dev'
                    } else {
                        error "Unsupported branch: ${env.BRANCH_NAME}"
                    }

                    echo """
                    ENV_NAME=${ENV_NAME}
                    APP_PORT=${APP_PORT}
                    IMAGE_NAME=${IMAGE_NAME}
                    CONTAINER_NAME=${CONTAINER_NAME}
                    """
                }
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Build stage for ${ENV_NAME}"
                sh 'node -v'
                sh 'npm -v'
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                echo "Test stage for ${ENV_NAME}"
                sh 'npm test || true'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: ${IMAGE_NAME}"
                sh "docker build -t ${IMAGE_NAME} ."
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying ${ENV_NAME} on port ${APP_PORT}"

                sh """
                  docker rm -f ${CONTAINER_NAME} || true

                  docker run -d \
                    --name ${CONTAINER_NAME} \
                    -p ${APP_PORT}:${APP_PORT} \
                    -e PORT=${APP_PORT} \
                    ${IMAGE_NAME}
                """
            }
        }
    }

    post {
        success {
            echo "Deployment successful for ${ENV_NAME}"
        }
        failure {
            echo "Deployment failed for ${ENV_NAME}"
        }
    }
}
